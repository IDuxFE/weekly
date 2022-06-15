当我们在终端上敲下 vite（vite dev、vite server）到返回下图结果

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ce3b3f4a11604ed4be982c3c29151e3b~tplv-k3u1fbpfcp-zoom-1.image)

中间发生了什么事呢？我们直接看到源码入口：

```typescript
import { cac } from 'cac'

const cli = cac('vite')

// ...

/**
 * 对于特定子命令时删除全局的参数
 */
function cleanOptions<Options extends GlobalCLIOptions>(
  options: Options
): Omit<Options, keyof GlobalCLIOptions> {
  const ret = { ...options }
  delete ret['--']
  delete ret.c
  delete ret.config
  delete ret.base
  delete ret.l
  delete ret.logLevel
  delete ret.clearScreen
  delete ret.d
  delete ret.debug
  delete ret.f
  delete ret.filter
  delete ret.m
  delete ret.mode
  return ret
}

// 全局公共参数
cli
  .option('-c, --config <file>', `[string] use specified config file`)
  .option('--base <path>', `[string] public base path (default: /)`)
  .option('-l, --logLevel <level>', `[string] info | warn | error | silent`)
  .option('--clearScreen', `[boolean] allow/disable clear screen when logging`)
  .option('-d, --debug [feat]', `[string | boolean] show debug logs`)
  .option('-f, --filter <filter>', `[string] filter debug logs`)
  .option('-m, --mode <mode>', `[string] set env mode`)

cli
  .command('[root]', 'start dev server') // default command
  .alias('serve') // the command is called 'serve' in Vite's API
  .alias('dev') // alias to align with the script name
  .option('--host [host]', `[string] specify hostname`)
  .option('--port <port>', `[number] specify port`)
  .option('--https', `[boolean] use TLS + HTTP/2`)
  .option('--open [path]', `[boolean | string] open browser on startup`)
  .option('--cors', `[boolean] enable CORS`)
  .option('--strictPort', `[boolean] exit if specified port is already in use`)
  .option(
    '--force',
    `[boolean] force the optimizer to ignore the cache and re-bundle`
  )
  .action(async (root: string, options: ServerOptions & GlobalCLIOptions) => {
    // output structure is preserved even after bundling so require()
    // is ok here
  	// 
    const { createServer } = await import('./server')
    try {
      // options 获取命令行参数
      const server = await createServer({
        root,
        base: options.base,
        mode: options.mode,
        configFile: options.config,
        logLevel: options.logLevel,
        clearScreen: options.clearScreen,
        server: cleanOptions(options)
      })

      if (!server.httpServer) {
        throw new Error('HTTP server not available')
      }

      await server.listen()

      // ...
  })

// ...
```

上述代码中，通过 [cac](https://www.npmjs.com/package/cac) 创建 CLI 工具，包特点是：

- 超级轻量，没有任何依赖，就一个文件；
- 简单上手，只需要 4 个 APIs 就能开发一个 CLI 应用：`cli.option` `cli.version` `cli.help` `cli.parse`；
- 麻雀虽小，五脏俱全。启用默认命令、类 git 子命令、验证所需参数和选项、可变参数、嵌套选项、自动帮助消息生成等功能。
- 工具使用 TypeScript 开发的。

定义了如下全局的配置：

```typescript
// 从 TS 的接口类型了解参数再友好不过啦
interface GlobalCLIOptions {
  '--'?: string[]
  c?: boolean | string
  config?: string
  base?: string
  l?: LogLevel
  logLevel?: LogLevel
  clearScreen?: boolean
  d?: boolean | string
  debug?: boolean | string
  f?: string
  filter?: string
  m?: string
  mode?: string
}
```

对于特定的子命令，不需要全局参数时通过 cleanOptions 删除。函数的作用也可以通过 TS 的声明来快速理解哦！

```typescript
function cleanOptions<Options extends GlobalCLIOptions>(
  options: Options
): Omit<Options, keyof GlobalCLIOptions> {
  // ...
}
```

[Omit<Type, Keys>](https://www.typescriptlang.org/docs/handbook/utility-types.html#omittype-keys) 工具类型函数用于从 Type 中剔除 Keys ，就能知道 cleanOptions 作用是从所有参数中剔除 CLI 的全局选项。

最后我们聚焦在 vite（等价于 vite dev、vite server ）这个 action。

从 CLI 中接收 host、port、https、open 等开发服务器（server）的配置，然后调用 createServer 创建服务，接着进入函数内部看具体做了哪些事情。函数位于 `packages/vite/src/node/server/index.ts`：

```typescript
// 接收传入配置创建服务
export async function createServer(
  inlineConfig: InlineConfig = {}
): Promise<ViteDevServer> {
  // 从 CLI + 默认参数中获取 development 或 server 的 config，具体解析留在下一节，这里只关注整个 createServer 流程
  const config = await resolveConfig(inlineConfig, 'serve', 'development')

  // 根目录
  const root = config.root  
  // 开发服务器的配置
  const serverConfig = config.server
  // https 配置
  const httpsOptions = await resolveHttpsConfig(
    config.server.https,
    config.cacheDir
  )
  // 以中间件模式创建 Vite 服务器
  let { middlewareMode } = serverConfig
  
  // middlewareMode 等于 ssr，将禁用 Vite 自身的 HTML 服务逻辑
  if (middlewareMode === true) {
    middlewareMode = 'ssr'
  }
  // 通过 connect 初始化中间接，这也是 express 的中间件依赖包
  const middlewares = connect() as Connect.Server
  // 使用 node 的 http || https || http2 创建服务
  const httpServer = middlewareMode
    ? null
    : await resolveHttpServer(serverConfig, middlewares, httpsOptions)
    
  // 创建 websocket 服务
  const ws = createWebSocketServer(httpServer, config, httpsOptions)
  
  // 获取 watch 参数
  const { ignored = [], ...watchOptions } = serverConfig.watch || {}
  
  // 通过 chokidar 监控文件变化
  const watcher = chokidar.watch(path.resolve(root), {
    // ...
  }) as FSWatcher
  
  // ⚠️ 初始化模块图
  const moduleGraph: ModuleGraph = new ModuleGraph((url, ssr) =>
    container.resolveId(url, undefined, { ssr })
  )
  
  // ⚠️ 创建插件容器
  const container = await createPluginContainer(config, moduleGraph, watcher)
  
  // 关闭服务后的回调函数
  const closeHttpServer = createServerCloseFn(httpServer)

  // eslint-disable-next-line prefer-const
  let exitProcess: () => void

  // ⚠️ 用字面量的形式定义 server 配置，能够通过插件的 configureServer 钩子中获取
  const server: ViteDevServer = {
    // 省略全部 server 的配置项...
  }
  
  // ⚠️ 插件的 transformIndexHtml 钩子在这里就执行啦，用于转换 index.html
  server.transformIndexHtml = createDevHtmlTransformFn(server)

	// ...
  
  // ⚠️ 文件改变时触发事件
  watcher.on('change', async (file) => {
   // ...
  })

  // 执行插件中的 configureServer 钩子
  const postHooks: ((() => void) | void)[] = []
  for (const plugin of config.plugins) {
    if (plugin.configureServer) {
      postHooks.push(await plugin.configureServer(server))
    }
  }

  // 省略一系列内部的中间件...
  // 主要的转换文件中间件
  middlewares.use(transformMiddleware(server))

  // 预构建
  const runOptimize = async () => {
    server._isRunningOptimizer = true
    try {
      server._optimizeDepsMetadata = await optimizeDeps(
        config,
        config.server.force || server._forceOptimizeOnRestart
      )
    } finally {
      server._isRunningOptimizer = false
    }
    server._registerMissingImport = createMissingImporterRegisterFn(server)
  }

  // 不是以中间件模式创建的 Vite 服务，是 http 服务
  if (!middlewareMode && httpServer) {
    let isOptimized = false
    
    // 重写 listen 函数为了在服务启动之前执行预构建
    const listen = httpServer.listen.bind(httpServer)
    httpServer.listen = (async (port: number, ...args: any[]) => {
      if (!isOptimized) {
        try {
          // ⚠️ 执行插件容器的 buildStart 钩子
          await container.buildStart({})
					// ⚠️ 执行预编译
          await runOptimize()
          isOptimized = true
        }
      }
      return listen(port, ...args)
    }) as any
  }

  // 最终返回 server
  return server
}
```

createServer 代码精简之后还是比较长，但如果再结合下图去看整个过程就非常清晰了

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c079a6e827814116a3a988de2c56232f~tplv-k3u1fbpfcp-zoom-1.image)

我们再来详述整个流程：

1. 当我们在终端上敲入 vite 时，vite node 端会读取、过滤命令行参数，然后调用 createServer 创建服务器；

2. resolveConfig 通过拿到外部传入的**配置信息 inlineConfig** 、**子命令**（serve 或者 build）、**mode** （ CLI 参数没有传入 mode 参数时，默认是 development ）去生成贯穿整个 vite 的 config；这个过程留在下一小节详述，过程还比较有意思。

3. 第三步创建服务器，调用 resolveHttpsConfig 获取 https 配置：

 ```typescript
 /**
  * 解析 https 配置
  *
  * @export
  * @param {(boolean | HttpsServerOptions | undefined)} https
  * @param {string} cacheDir
  * @return {*}  {(Promise<HttpsServerOptions | undefined>)}
  */
 export async function resolveHttpsConfig(
   https: boolean | HttpsServerOptions | undefined,
   cacheDir: string
 ): Promise<HttpsServerOptions | undefined> {
   // server.https 是 false 或者 undefined，不开启 https
   if (!https) return undefined

   const httpsOption = isObject(https) ? { ...https } : {}

   // 获取 https 基本配置
   const { ca, cert, key, pfx } = httpsOption
   Object.assign(httpsOption, {
     ca: readFileIfExists(ca),
     cert: readFileIfExists(cert),
     key: readFileIfExists(key),
     pfx: readFileIfExists(pfx)
   })
   if (!httpsOption.key || !httpsOption.cert) {
     httpsOption.cert = httpsOption.key = await getCertificate(cacheDir)
   }
   return httpsOption
 }
 ```

然后使用 [connect](https://www.npmjs.com/package/connect) 初始化中间件，vite 内部使用了大量中间件，timeMiddleware、corsMiddleware 等等；最后调用 resolveHttpServer 和 createWebSocketServer 创建 http 和 ws 服务器：

 ```typescript
 // 使用 node 的 http || https || http2 创建服务
 const httpServer = middlewareMode
 ? null
 : await resolveHttpServer(serverConfig, middlewares, httpsOptions)
 // 创建websocket服务
 const ws = createWebSocketServer(httpServer, config, httpsOptions)

 export async function resolveHttpServer(
   { proxy }: CommonServerOptions,
   app: Connect.Server,
   httpsOptions?: HttpsServerOptions
 ): Promise<HttpServer> {
   // ...

   // 没有定义 https 配置的情况
   if (!httpsOptions) {
     return require('http').createServer(app)
   }

   // 配置自定义代理规则
   if (proxy) {
     // #484 fallback to http1 when proxy is needed.
     return require('https').createServer(httpsOptions, app)
   } else {
     return require('http2').createSecureServer(
       {
         ...httpsOptions,
         allowHTTP1: true
       },
       app
     )
   }
 }

 export function createWebSocketServer(
   server: Server | null,
   config: ResolvedConfig,
   httpsOptions?: HttpsServerOptions
 ): WebSocketServer {
   let wss: WebSocket
   let httpsServer: Server | undefined = undefined

   const hmr = isObject(config.server.hmr) && config.server.hmr
    // 没有指定hmr的服务，默认成 httpServer
   const wsServer = (hmr && hmr.server) || server

   // 指定了 ws 服务地址
   if (wsServer) {
     wss = new WebSocket({ noServer: true })
     // ...
   } else {
     const websocketServerOptions: ServerOptions = {}
     const port = (hmr && hmr.port) || 24678
     // 存在 https 的配置
     if (httpsOptions) {
       httpsServer = createHttpsServer(httpsOptions, (req, res) => {
         // ...
         res.end(body)
       })

       httpsServer.listen(port)
       websocketServerOptions.server = httpsServer
     } else {
       // we don't need to serve over https, just let ws handle its own server
       websocketServerOptions.port = port
     }

     // vite dev server in middleware mode
     wss = new WebSocket(websocketServerOptions)
   }

   wss.on('connection', (socket) => {
     // ...
   })

   wss.on('error', (e: Error & { code: string }) => {
     // ...
   })


   return {
     on: wss.on.bind(wss),
     off: wss.off.bind(wss),
     send(payload: HMRPayload) {
       // ...
     },
     close() {
      // ...
     }
   }
 }

 ```

4. 使用 chokidar 创建文件监控器，当前目录下任何文件有风吹草动，都会触发 watcher 上的监听函数：

 ```typescript
 // 通过 chokidar 监控文件变化
 const watcher = chokidar.watch(path.resolve(root), {
   ignored: [
     '**/node_modules/**',
     '**/.git/**',
     ...(Array.isArray(ignored) ? ignored : [ignored])
   ],
   ignoreInitial: true,
   ignorePermissionErrors: true,
   disableGlobbing: true,
   ...watchOptions
 }) as FSWatcher

 // ⚠️ 文件改变时触发 change 事件，热更也是在这里开始的哦
 watcher.on('change', async (file) => {
   // ...
 })

 // 添加文件时触发 add 事件
 watcher.on('add', (file) => {
   // ...
 })

 // 删除文件时触发 unlink 事件
 watcher.on('unlink', (file) => {
   // ...
 })
 ```

5. 初始化 ModuleGraph 实例，生成模块依赖图谱，图谱中每一个节点都是 ModuleNode 的实例：

 ```typescript
 /**
  * 每一个模块节点的信息
  */
 export class ModuleNode {
   // 省略模块节点属性代码...

   constructor(url: string) {
     this.url = url
     this.type = isDirectCSSRequest(url) ? 'css' : 'js'
   }
 }

 export class ModuleGraph {
   // url 和模块的映射
   urlToModuleMap = new Map<string, ModuleNode>()
   // id 和模块的映射
   idToModuleMap = new Map<string, ModuleNode>()
   // 文件和模块的映射，一个文件对应多个模块，比如 SFC 就对应多个模块
   fileToModulesMap = new Map<string, Set<ModuleNode>>()
   // /@fs 的模块
   safeModulesPath = new Set<string>()

   constructor(
     // 内部的 resolvceId 是通过构造函数传进来的
     private resolveId: (
       url: string,
       ssr: boolean
     ) => Promise<PartialResolvedId | null>
   ) {}

   // 省略全部API方法
 }
 ```

6. 调用 createPluginContainer 生成插件容器，这里很明确地指明了 vite 插件跟 rollup 插件的联系；

 ```typescript
 // PluginContext 来自 rollup，vite 的钩子大部分取自 rollup
 class Context implements PluginContext {}

 // 转换上下文有专属 vite 的 sourcemap 处理
 class TransformContext extends Context {}

 // 定义插件容器
 const container: PluginContainer = {
   // 服务启动时读取配置的钩子
  options: await (async () => {
     // ...
   })(),
   // 开始构建时的钩子
   async buildStart() {},
   // 自定义解析器
   async resolveId(rawId, importer = join(root, 'index.html'), options) {
     // ...
   }
   // 自定义加载器钩子
   async load(id, options) {},
   // 转换器钩子
   async transform(code, id, options) {},
   // 服务关闭钩子
   async close() {}
 }

 return container
 ```

7. 用字面量定义 server 实例，上面囊括了 vite 开发服务器的全部信息，在我们开发插件时通过 `configureServer` 钩子能获取到 server 信息并可以在 middlewares 上扩展中间件。

 ```typescript
 // 用字面量的形式初始化 server 配置
 const server: ViteDevServer = {
   // 配置
   config,
   // 中间件信息
   middlewares,
   get app() {
     config.logger.warn(
       `ViteDevServer.app is deprecated. Use ViteDevServer.middlewares instead.`
     )
     return middlewares
   },
   // http server 信息
   httpServer,
   // 文件监控实例
   watcher,
   // 插件容器
   pluginContainer: container,
   // websocket server
   ws,
   // 模块图谱
   moduleGraph,
   // ssr 转换器
   ssrTransform,
   // esbuild 转换器
   transformWithEsbuild,
   // 加载并转换具体的 url 指向的文件
   transformRequest(url, options) {
     return transformRequest(url, server, options)
   },
   // transformIndexHtml 钩子
   transformIndexHtml: null!, // to be immediately set
   // ssr加载模块函数
   ssrLoadModule(url, opts?: { fixStacktrace?: boolean }) {
     server._ssrExternals ||= resolveSSRExternal(
       config,
       server._optimizeDepsMetadata
       ? Object.keys(server._optimizeDepsMetadata.optimized)
       : []
     )
     return ssrLoadModule(
       url,
       server,
       undefined,
       undefined,
       opts?.fixStacktrace
     )
   },
   // ssr 堆栈信息
   ssrFixStacktrace(e) {
     if (e.stack) {
       const stacktrace = ssrRewriteStacktrace(e.stack, moduleGraph)
       rebindErrorStacktrace(e, stacktrace)
     }
   },
   // 启动服务
   listen(port?: number, isRestart?: boolean) {
     return startServer(server, port, isRestart)
   },
   // 关闭服务
   async close() {
     process.off('SIGTERM', exitProcess)

     if (!middlewareMode && process.env.CI !== 'true') {
       process.stdin.off('end', exitProcess)
     }

     await Promise.all([
       watcher.close(),
       ws.close(),
       container.close(),
       closeHttpServer()
     ])
   },
   // 打印url辅助函数
   printUrls() {
     if (httpServer) {
       printCommonServerUrls(httpServer, config.server, config)
     } else {
       throw new Error('cannot print server URLs in middleware mode.')
     }
   },
   // 重启服务
   async restart(forceOptimize: boolean) {
     if (!server._restartPromise) {
       server._forceOptimizeOnRestart = !!forceOptimize
       server._restartPromise = restartServer(server).finally(() => {
         server._restartPromise = null
         server._forceOptimizeOnRestart = false
       })
     }
     return server._restartPromise
   },

   // ...
 }
 ```

8. 最后重写了 server 的 listen 方法，在服务器启动之前执行 containerPlugin 的 buildStart 钩子和预构建。并返回 server，外部通过调用 listen 启动服务器，并打印访问链接和启动时间等信息。也就回到了文章开头那张服务成功运行后的图：

  ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7bfaf63b3e894d52bccc6c53c2db1af0~tplv-k3u1fbpfcp-zoom-1.image)

## 总结

再回头看整个流程概览图：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff169d46d9e84cb9a5af844ce62bcc87~tplv-k3u1fbpfcp-zoom-1.image)


当我们敲下 vite 命令后，vite 在短短时间内做了解析配置、创建 http server、创建 ws、创建文件监听器、初始化模块依赖图、创建插件、预构建、启动服务等任务。

除此之外，我们还接触到工作中也许会用上的工具包：

- cac 用于创建命令行工具
- chokidar 用于监听文件变化

一起用起来吧~