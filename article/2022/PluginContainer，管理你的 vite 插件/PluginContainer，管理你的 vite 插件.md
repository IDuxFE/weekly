上一小节我们了解了从敲入 vite 命令到最后服务运行起来的详细过程。本节开始我们从流程中选一些核心流程细细品味，首先看入口配置（即 resolveConfig 函数）的逻辑。

按照惯例，我们先上一个 [DEMO](https://github.com/Jouryjc/vite "DEMO")，用 vanilla 模板初始化一个 Vite 项目，然后在根目录创建 vite.config.ts 配置文件，内容如下：

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import vitePluginB from './plugins/vite-plugin-B'

export default defineConfig({
  server: {
    port: 8888
  },

  plugins: [
    {
      name: 'testA-plugin',

      enforce: 'post',

      config(config, configEnv) {
        console.log('插件A  --->  config', configEnv)
      },

      configResolved(resolvedConfigA) {
        console.log('插件A  --->  configResolved')
      }
    },
    vitePluginB()
  ]
})

// ./plugins/vite-plugin-B
import type { Plugin } from 'vite'

export default function PluginB(): Plugin {
  return {
    name: 'testB-plugin',

    enforce: 'pre',

    config(config, configEnv) {
      console.log('插件B  --->  config', configEnv)
    },

    configResolved(resolvedConfigB) {
      console.log('插件B  --->  configResolved')
    }
  }
}
```

上述代码通过 defineConfig 指定了 http 服务器的端口为 8888，并定义了两个插件 A 和 B，插件都只使用了 config 和 configResolved 钩子，插件 A 的 enforce 设置为 post，插件 B 的 enforce 设置为 pre。

然后在源码入口（packages/vite/src/node/cli.ts）打上断点：

![](https://files.mdnice.com/user/15734/973798f3-088a-4cbe-9f59-6c0408dd9cff.png)

最后我们执行以下命令：

```sh
pnpx vite dev --host 0.0.0.0 --port 3333 --config vite.config.ts
```

忘记调试过程的童鞋可以按照 [授人予渔，如何调试 CLI 代码？](./entry "授人予渔，如何调试 CLI 代码？") 的过程重新配置 debugger 环境。

## 流程分析

根据上面的执行命令，获取到的 options 的结果如下：

```typescript
options = {
  --: [],
  c: 'vite.config.ts',
  config: 'vite.config.ts',
  host:'0.0.0.0',
  port: 3333,
}
```

传给 server 的字段用 cleanOptions 处理之后返回 `{ host: '0.0.0.0', port: 3333 }`：

```typescript
// dev
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
    const { createServer } = await import('./server')
    try {
      // dev、server 的 action
      const server = await createServer({
        root,								// undefined
        base: options.base,	// undefined
        mode: options.mode,	// undefined
        configFile: options.config,	// vite.config.ts
        logLevel: options.logLevel,	// undefined
        clearScreen: options.clearScreen,	// undefined
        server: cleanOptions(options)	// { host: '0.0.0.0', port: 3333 }
      })
      
		 // ...
  })
```

从  [敲下命令后，Vite 做了哪些事？](./create-server "敲下命令后，Vite 做了哪些事？") 中我们已经知道，终端中输入子命令后，会通过 cleanOptions 对全局参数做过滤，随后通过 createServer 创建 http 服务器。

```typescript
export async function createServer(
  inlineConfig: InlineConfig = {}
): Promise<ViteDevServer> {
  // 从 CLI 和部分全局参数中获取 inlineConfig
  // 在 dev 模式下，command 是 server，mode 是 development
  const config = await resolveConfig(inlineConfig, 'serve', 'development')
	// ...
}
```

config 是整个 createServer 过程中依赖的核心信息。resolveConfig 函数位于 packages/vite/src/node/config.ts，该函数体有接近 400 行代码，先了解整体流程，然后再去阅读源码。整体流程如下图所示：

![](https://files.mdnice.com/user/15734/c488f09b-13c9-4e05-a52c-ccdfd11f11c1.jpeg)

调用 resolveConfig 时，inlineConfig 参数如下所示：

```typescript
{
  root: undefined,
  base: undefined,
  mode: undefined,
  configFile: 'vite.config.ts',
  logLevel: undefined,
  clearScreen: undefined,
  server: { host: '0.0.0.0', port: 3333 }
}
```

对于整个 Vite 应用而言，参数不仅只从命令中获取，也会从上述 configFile 指向的配置文件中加载。配置合并之后，就会去调用插件的 config 钩子，钩子参数就是完整的 config 信息。随后便会对部分配置（上图中橙色部分）做 normalize （规范化），最后执行插件的 resolvedConfig 钩子，整个配置解析过程就结束了。

从全局了解完整个 resolveConfig 流程，接下来就逐个学习绿色方块的功能逻辑。其他颜色的逻辑比较清晰，当遇到问题时再回头来阅读内部细节即可。

### 加载配置

```typescript
// 解析配置流程
export async function resolveConfig(
  inlineConfig: InlineConfig,
  command: 'build' | 'serve',
  defaultMode = 'development'
): Promise<ResolvedConfig> {
  let config = inlineConfig
  let configFileDependencies: string[] = []
  // ...

  // 这是我们能在 config 钩子中拿到的参数
  const configEnv = {
    mode,
    command
  }
  
  // demo 中 configFile 是 vite.config.ts
  let { configFile } = config
  // 例子中 configFile 是 vite.config.ts，会进入 loadConfigFromFile 逻辑
  if (configFile !== false) {
    const loadResult = await loadConfigFromFile(
      configEnv,
      configFile,
      config.root,
      config.logLevel
    )
    // 拿到 vite.confit.ts 结果，进行配置合并，可以看出 CLI 的参数优先级大于配置文件
    if (loadResult) {
      config = mergeConfig(loadResult.config, config)
      configFile = loadResult.path
      configFileDependencies = loadResult.dependencies
    }
  }
  // ...
}
```

首先定义了 configEnv 变量，在 dev 模式下，mode 等于 development，command 等于 server。对于本文例子，获取到的 configFile 是 vite.config.ts，接下来进入 loadConfigFromFile 做配置的读取：

```typescript
export async function loadConfigFromFile(
  configEnv: ConfigEnv,
  configFile?: string,
  configRoot: string = process.cwd(),	// 默认值是执行命令的根路径
  logLevel?: LogLevel
): Promise<{
  path: string
  config: UserConfig
  dependencies: string[]
} | null> {
  // node 上常用的逻辑执行时间
  const start = performance.now()
  const getTime = () => `${(performance.now() - start).toFixed(2)}ms`

  let resolvedPath: string | undefined
  let isTS = false
  let isESM = false
  let dependencies: string[] = []

  try {
    // 查找 package.json 文件，如果有 type: module 信息就将 isESM 变量变成 true
    const pkg = lookupFile(configRoot, ['package.json'])
    if (pkg && JSON.parse(pkg).type === 'module') {
      isESM = true
    }
  } catch (e) {}

  if (configFile) {
    resolvedPath = path.resolve(configFile)
    // 判断配置文件是不是 ts 文件
    isTS = configFile.endsWith('.ts')

    if (configFile.endsWith('.mjs')) {
      isESM = true
    }
  } else {
    // 尝试依次去获取 vite.config.js、vite.config.mjs、vite.config.ts、vite.config.cjs 等可能的配置文件，所以你不指定 config 参数也可以获取到文件中的配置
    const jsconfigFile = path.resolve(configRoot, 'vite.config.js')
    if (fs.existsSync(jsconfigFile)) {
      resolvedPath = jsconfigFile
    }

    if (!resolvedPath) {
      const mjsconfigFile = path.resolve(configRoot, 'vite.config.mjs')
      if (fs.existsSync(mjsconfigFile)) {
        resolvedPath = mjsconfigFile
        isESM = true
      }
    }

    if (!resolvedPath) {
      const tsconfigFile = path.resolve(configRoot, 'vite.config.ts')
      if (fs.existsSync(tsconfigFile)) {
        resolvedPath = tsconfigFile
        isTS = true
      }
    }

    if (!resolvedPath) {
      const cjsConfigFile = path.resolve(configRoot, 'vite.config.cjs')
      if (fs.existsSync(cjsConfigFile)) {
        resolvedPath = cjsConfigFile
        isESM = false
      }
    }
  }

  if (!resolvedPath) {
    debug('no config file found.')
    return null
  }

  try {
    let userConfig: UserConfigExport | undefined

    if (isESM) {
      // ...例子不会进入该逻辑，所以省略，感兴趣的童鞋可以把 vite.config.ts 改成 vite.config.mjs
    }

    if (!userConfig) {
      // 将配置文件用 esbuild 进行构建并输出 cjs 包
      const bundled = await bundleConfigFile(resolvedPath)
      // 获取构建后的依赖
      dependencies = bundled.dependencies
      // 使用 require.extensions 去扩展支持 ts 文件，获得编译之后的结果
      userConfig = await loadConfigFromBundledFile(resolvedPath, bundled.code)
      debug(`bundled config file loaded in ${getTime()}`)
    }

    // 解析后是一个函数，就执行，可以看到执行结果返回一个 promise。传入的参数 configEnv，也是为什么配置文件能够支持情景配置和异步配置的原因。
    const config = await (typeof userConfig === 'function'
      ? userConfig(configEnv)
      : userConfig)
    
    return {
      path: normalizePath(resolvedPath),
      config,
      dependencies
    }
  } catch (e) {
   	// ...
  }
}
```

loadConfigFromFile 执行 `lookupFile(configRoot, ['package.json'])` 从当前目录开始寻找 package.json 文件，如果当前目录没找到，就递归往父级目录寻找，找到后读取文件内容并返回。通过判断 package.json 中 type 等于 module 识别是否使用 esm 模块机制（isESM）。然后根据配置文件后缀定义 isTS 变量。如果没有指定 configFile 信息，Vite 会尝试依次寻找 vite.config.js、vite.config.mjs、vite.config.ts、vite.config.cjs 等可能的配置文件。

例子中配置文件是 vite.config.ts，所以通过 bundleConfigFile 去构建 ts：

```typescript
import { build } from 'esbuild'

async function bundleConfigFile(
  fileName: string,
  isESM = false
): Promise<{ code: string; dependencies: string[] }> {
  const result = await build({
    // 工作绝对路径
    absWorkingDir: process.cwd(),
    // 入口
    entryPoints: [fileName],
    // 输出文件
    outfile: 'out.js',
    // 不写入文件系统
    write: false,
    // 构建结果在 node 中运行
    platform: 'node',
    // 构建依赖
    bundle: true,
    // 输出格式
    format: isESM ? 'esm' : 'cjs',
    // 内联 sourcemap
    sourcemap: 'inline',
    // 输出构建信息
    metafile: true,
    plugins: [
      // ..
    ]
  })
  
  // 构建结果
  const { text } = result.outputFiles[0]
  return {
    code: text,
    dependencies: result.metafile ? Object.keys(result.metafile.inputs) : []
  }
}
```

bundleConfigFile 使用 esbuild 去构建 vite.config.ts 文件，返回产物和依赖列表如下图所示：

![](https://files.mdnice.com/user/15734/2e1a6324-0723-41c9-8893-d7db895b0106.png)


然后通过 loadConfigFromBundledFile 去加载配置：

```typescript
// 例子中，传入的 fileName 是绝对路径的配置文件 /../vite.config.ts
// bundledCode 是上述用 esbuild 构建后的产物
async function loadConfigFromBundledFile(
  fileName: string,
  bundledCode: string
): Promise<UserConfig> {
  // 获取文件的扩展，例子是 .ts
  const extension = path.extname(fileName)
  // 默认的 .ts 解析器，node 默认不支持 ts，所以是 undefined
  const defaultLoader = require.extensions[extension]!
  // 扩展 cjs 的 require 支持的类型，使 node 支持 ts 的解析
  require.extensions[extension] = (module: NodeModule, filename: string) => {
    if (filename === fileName) {
      // 调用 module._compile 方法对代码进行编译操作
      ;(module as NodeModuleWithCompile)._compile(bundledCode, filename)
    } else {
      defaultLoader(module, filename)
    }
  }
  
  // 如果是重启服务的情况，这里有缓存结果，所以需要清楚掉文件的缓存
  delete require.cache[require.resolve(fileName)]
  const raw = require(fileName)
  const config = raw.__esModule ? raw.default : raw
  // 重置 .ts 扩展定义，对于例子就是清除 ts 的解析器
  require.extensions[extension] = defaultLoader
  return config
}
```

可以看到，通过 require.extensions[extension] 扩展 node 支持 ts 类型，使用 (module as NodeModuleWithCompile)._compile(bundledCode, filename) 编译加载的 vite.config.ts 文件。最终返回 config 如下图所示 :

![](https://files.mdnice.com/user/15734/c49dc443-443e-4c26-88e2-cffa4f7b3f0f.png)

在 vite.config.ts 中定义的配置到这里就全部被获取到了。对于例子而言，最终从配置文件获取的是 object，若结果是函数，则会传入 configEnv 对象并执行函数获取结果，这常用于需要基于（`dev`/`serve` 或 `build`）命令或者不同的 [模式](https://cn.vitejs.dev/guide/env-and-mode.html "模式") 来决定选项的情况。

拿到配置文件的结果，最后还会跟命令行传入的参数做一个合并，对于不同的配置有不同的合并策略，我们进入 mergeConfig：

```typescript
/**
 * 合并两个的配置
 *
 * @export
 * @param {Record<string, any>} defaults
 * @param {Record<string, any>} overrides
 * @param {boolean} [isRoot=true]
 * @return {*}  {Record<string, any>}
 */
export function mergeConfig(
  defaults: Record<string, any>,
  overrides: Record<string, any>,
  isRoot = true
): Record<string, any> {
  // 此时合并 isRoot 是默认参数 true
  return mergeConfigRecursively(defaults, overrides, isRoot ? '' : '.')
}

function mergeConfigRecursively(
  defaults: Record<string, any>,
  overrides: Record<string, any>,
  rootPath: string
) {
  // 浅拷贝一份从文件 vite.config.ts 获取的配置
  const merged: Record<string, any> = { ...defaults }
  
  // 遍历 CLI 中指定的配置
  for (const key in overrides) {
    // CLI 没有定义的项，等于不会做覆盖，直接进入下一轮
    const value = overrides[key]
    if (value == null) {
      continue
    }

    // 获取 vite.config.ts 对应的配置项
    const existing = merged[key]
    
    // 如果在配置文件中未定义，直接使用 CLI 的配置
    if (existing == null) {
      merged[key] = value
      continue
    }

    // fields that require special handling
    // 此时 rootPath 是 ''，不满足
    if (key === 'alias' && (rootPath === 'resolve' || rootPath === '')) {
      merged[key] = mergeAlias(existing, value)
      continue
    } else if (key === 'assetsInclude' && rootPath === '') {
      merged[key] = [].concat(existing, value)
      continue
    } else if (key === 'noExternal' && existing === true) {
      continue
    }

    // 有一个数组的情况
    if (Array.isArray(existing) || Array.isArray(value)) {
      merged[key] = [...arraify(existing ?? []), ...arraify(value ?? [])]
      continue
    }
    // 都是对象，递归处理
    if (isObject(existing) && isObject(value)) {
      merged[key] = mergeConfigRecursively(
        existing,
        value,
        rootPath ? `${rootPath}.${key}` : key
      )
      continue
    }

    merged[key] = value
  }
  return merged
}
```

至此，整个配置获取和合并的流程就分析完了。

#### 小结

用一张图描述加载配置的过程：

![](https://files.mdnice.com/user/15734/a99bf194-1c93-436f-9b57-96e525a6355b.jpeg)

1. 先在当前和父级目录寻找 package.json 文件，找到后返回文件内容；
2. 例子中配置文件是 vite.config.ts，所以会使用 esbuild 构建输出 CJS 结果；
3. 扩展 require.extensions[.ts]，通过 module._compile 编译加载的 ts 文件；
4. 判断获取到的 config 是不是函数，是的话传入 configEnv 执行函数并获取结果；
5. 最后将第四步结果跟 CLI 的参数进行 merge，得到 config。

了解完整个流程，如果让你去实现一个支持复杂配置的命令行程序，为了提高配置的易用性，就可以模仿 Vite 通过 defineConfig 提供完备的 TypeScript 类型提示，然后使用 esbuild 进行构建，最后扩展 require.extensions 去获取配置。

### 插件及钩子

我们知道，在 resolveConfig 阶段会去调用插件的 config 和 configResolved 钩子。钩子的执行顺序依赖插件中声明的 enforce 属性。接下来我们就看一下源码：

```typescript
export async function resolveConfig(
  inlineConfig: InlineConfig,
  command: 'build' | 'serve',
  defaultMode = 'development'
): Promise<ResolvedConfig> {
  let config = inlineConfig
  let configFileDependencies: string[] = []
  let mode = inlineConfig.mode || defaultMode
  
  // 这是我们能在 config 钩子中拿到的参数
  const configEnv = {
    mode,
    command
  }
  
	// ...
  mode = inlineConfig.mode || config.mode || mode
  configEnv.mode = mode

  // 获取插件列表，拍平，并过滤真值的插件
  const rawUserPlugins = (config.plugins || []).flat().filter((p) => {
    // 假值的插件会被忽略
    if (!p) {
      return false
    // 没有 apply 属性，说明是对象，直接返回 true
    } else if (!p.apply) {
      return true
    // apply 是一个函数，则传入configEnv执行插件函数，最后根据返回值结果去决定是否使用
    } else if (typeof p.apply === 'function') {
      return p.apply({ ...config, mode }, configEnv)
    // 最后一种情况是 apply 可以定义一个属性值，比如 { apply: 'build' }，则只在构建下使用该插件
    } else {
      return p.apply === command
    }
  }) as Plugin[]

  // 根据 enforce 给插件排序
  const [prePlugins, normalPlugins, postPlugins] =
    sortUserPlugins(rawUserPlugins)

	// ...

  // 依次执行插件的 config 钩子
  const userPlugins = [...prePlugins, ...normalPlugins, ...postPlugins]
  for (const p of userPlugins) {
    if (p.config) {
      // 在 config 有返回值，就合并到 config 中
      const res = await p.config(config, configEnv)
      if (res) {
        config = mergeConfig(config, res)
      }
    }
  }

	// ...
  const resolved: ResolvedConfig = {
    // ...
  }

  // 合并内部插件和用户定义插件
  ;(resolved.plugins as Plugin[]) = await resolvePlugins(
    resolved,
    prePlugins,
    normalPlugins,
    postPlugins
  )

  // call configResolved hooks
  await Promise.all(userPlugins.map((p) => p.configResolved?.(resolved)))

  return resolved
}
```

代码只留下了插件的处理，流程如下：

1. 将用户配置的 plugins 全部拍平之后通过 apply 钩子做了插件的过滤，如果给例子中 testB-plugin 定义 { apply: 'build' }，说明它只在 build 阶段被调用。

2. 将过滤后的插件通过 sortUserPlugins 进行排序：

   ```typescript
   export function sortUserPlugins(
     plugins: (Plugin | Plugin[])[] | undefined
   ): [Plugin[], Plugin[], Plugin[]] {
     // 定义前置插件数组
     const prePlugins: Plugin[] = []
    	// 定义后置插件数组
     const postPlugins: Plugin[] = []
     // 没有定义 enforce 属性的插件数组
     const normalPlugins: Plugin[] = []
   
     if (plugins) {
       plugins.flat().forEach((p) => {
         if (p.enforce === 'pre') prePlugins.push(p)
         else if (p.enforce === 'post') postPlugins.push(p)
         else normalPlugins.push(p)
       })
     }
   
     return [prePlugins, normalPlugins, postPlugins]
   }
   ```

   函数逻辑很简单，定义 prePlugins、postPlugins、normalPlugins 三个数组，分别存储 enforce: pre、enforce: post、以及没有定义 enforce 属性的插件。最后按照 [prePlugins, normalPlugins, postPlugins] 的顺序返回插件列表。

3. 然后依次执行插件 config 钩子：

   ```typescript
   // 依次执行插件的 config 钩子
   const userPlugins = [...prePlugins, ...normalPlugins, ...postPlugins]
   for (const p of userPlugins) {
     if (p.config) {
       // 在 config 有返回值，就合并到 config 中
       const res = await p.config(config, configEnv)
       if (res) {
         config = mergeConfig(config, res)
       }
     }
   }
   ```

   注意两个细节，config 钩子函数可以是一个 Promise，config 有返回值即 res 存在，就会执行 mergeConfig（mergeConfig 就是上述 vite.config.ts 与命令行参数合并的流程） ，这种机制可以让你把 `enforece: pre` 插件的配置传给 `enforce: post` 的插件；

4. 执行完 config 钩子，就会规范一系列配置，这部分放到下一小节展开分析。用 resolved 变量存储全部已经规范后的配置；调用 resolvePlugins 合并内置插件和用户定义的外部插件：

   ```typescript
   ;(resolved.plugins as Plugin[]) = await resolvePlugins(
     resolved,
     prePlugins,
     normalPlugins,
     postPlugins
   )
   
   export async function resolvePlugins(
     config: ResolvedConfig,
     prePlugins: Plugin[],
     normalPlugins: Plugin[],
     postPlugins: Plugin[]
   ): Promise<Plugin[]> {
     // 打包构建的场景
     const isBuild = config.command === 'build'
   	// 构建模式下添加 build 配置
     const buildPlugins = isBuild
       ? (await import('../build')).resolveBuildPlugins(config)
       : { pre: [], post: [] }
   
     return [
       isBuild ? null : preAliasPlugin(),
       aliasPlugin({ entries: config.resolve.alias }),
       ...prePlugins,
       config.build.polyfillModulePreload
         ? modulePreloadPolyfillPlugin(config)
         : null,
       resolvePlugin({
         ...config.resolve,
         root: config.root,
         isProduction: config.isProduction,
         isBuild,
         packageCache: config.packageCache,
         ssrConfig: config.ssr,
         asSrc: true
       }),
       htmlInlineProxyPlugin(config),
       cssPlugin(config),
       config.esbuild !== false ? esbuildPlugin(config.esbuild) : null,
       jsonPlugin(
         {
           namedExports: true,
           ...config.json
         },
         isBuild
       ),
       wasmPlugin(config),
       webWorkerPlugin(config),
       workerImportMetaUrlPlugin(config),
       assetPlugin(config),
       ...normalPlugins,
       definePlugin(config),
       cssPostPlugin(config),
       config.build.ssr ? ssrRequireHookPlugin(config) : null,
       ...buildPlugins.pre,
       ...postPlugins,
       ...buildPlugins.post,
       // internal server-only plugins are always applied after everything else
       ...(isBuild
         ? []
         : [clientInjectionsPlugin(config), importAnalysisPlugin(config)])
     ].filter(Boolean) as Plugin[]
   }
   ```

   将用户定义的插件安插到整个插件数组中，也就控制了插件的执行顺序：

   - Alias
   - 带有 `enforce: 'pre'` 的用户插件
   - Vite 核心插件
   - 没有 enforce 值的用户插件
   - Vite 构建用的插件
   - 带有 `enforce: 'post'` 的用户插件
   - Vite 后置构建插件（最小化，manifest，报告）

   从 resolvePlugins 中可以看到完整的插件列表，每个插件功能这里不会展开说明，遇到功能疑惑时可以直接查看对应插件源码。

5. 规范配置和整合插件都处理完了，最后就会调用插件的 configResolved 钩子，使用这个钩子可以读取和存储最终解析的配置。

   ```typescript
   const resolved: ResolvedConfig = {
     ...config,
     // 配置文件
     configFile: configFile ? normalizePath(configFile) : undefined,
     // 配置文件中的依赖
     configFileDependencies,
     // CLI传入的配置
     inlineConfig,
     // 启动根目录
     root: resolvedRoot,
     // 公共基础路径
     base: BASE_URL,
     // 文件路径解析相关
     resolve: resolveOptions,
     // 静态资源文件夹
     publicDir: resolvedPublicDir,
     // 缓存目录
     cacheDir,
     // 当前启动的命令
     command,
     // 模式
     mode,
     // 是否生产环境
     isProduction,
     // 用户插件
     plugins: userPlugins,
     // 服务器
     server,
     // 构建配置
     build: resolvedBuildOptions,
     // 预览选项
     preview: resolvePreviewOptions(config.preview, server),
     // 环境变量
     env: {
       ...userEnv,
       BASE_URL,
       MODE: mode,
       DEV: !isProduction,
       PROD: isProduction
     },
     // 额外的静态资源
     assetsInclude(file: string) {
       return DEFAULT_ASSETS_RE.test(file) || assetsFilter(file)
     },
     // 日志处理器你
     logger,
     packageCache: new Map(),
     // Vite提供的解析器
     createResolver,
     // 预编译配置
     optimizeDeps: {
       ...config.optimizeDeps,
       esbuildOptions: {
         keepNames: config.optimizeDeps?.keepNames,
         preserveSymlinks: config.resolve?.preserveSymlinks,
         ...config.optimizeDeps?.esbuildOptions
       }
     },
     // worker 相关的配置
     worker: resolvedWorkerOptions
   }
   
   await Promise.all(userPlugins.map((p) => p.configResolved?.(resolved)))
   ```

6. 执行完 configResolved 钩子之后返回 resolved，通过图片来看看最终生成的配置：

  ![](https://files.mdnice.com/user/15734/6a5155d4-91ec-4c2a-860a-0be343756efd.png)


   上图展示了自定义插件和默认插件组成的列表；

  ![](https://files.mdnice.com/user/15734/60638393-541c-457c-a968-628a035b7c74.png)

   最后整个完整的配置详情；

#### 小结

![](https://files.mdnice.com/user/15734/eb8dbfaa-32d8-46c1-aae3-1b0a141999f9.png)

这一小节我们分析了在解析配置时，插件会先按照 enforce 属性进行排序，输出 pre、normal、post 三类。然后插件执行了 config 和 configResolved 钩子，前者在刚解析并合并完配置后就会触发，config 钩子的返回值能够依次传到下一个组件，后者会在全部配置规范和内外插件合并完之后触发。

这小节了解完了插件的处理情况，接下来我们就去看看其他配置的处理。

### 规范配置

这一小节我们了解常用配置`别名(alias)` 和`环境变量(env)` 处理过程。

#### alias

```typescript
// eslint-disable-next-line node/no-missing-require
export const CLIENT_ENTRY = require.resolve('vite/dist/client/client.mjs')
// eslint-disable-next-line node/no-missing-require
export const ENV_ENTRY = require.resolve('vite/dist/client/env.mjs')

// 客户端代码的别名
const clientAlias = [
  { find: /^[\/]?@vite\/env/, replacement: () => ENV_ENTRY },
  { find: /^[\/]?@vite\/client/, replacement: () => CLIENT_ENTRY }
]

// 合并内部 alias 和用户定义的 alias 配置
const resolvedAlias = normalizeAlias(
  mergeAlias(
    // @ts-ignore because @rollup/plugin-alias' type doesn't allow function
    // replacement, but its implementation does work with function values.
    clientAlias,
    config.resolve?.alias || config.alias || []
  )
)
```

有个细节可以看到，vite 的 alias 配置可以放在 config 下，但这是一个即将废弃的用法。

```typescript
export interface UserConfig {
  /**
   * Import aliases
   * @deprecated use `resolve.alias` instead
   */
  alias?: AliasOptions;
}
```

当我们自己设计工具时，因为功能迭代的关系也会出现调整配置位置、参数等情况，这时可以友好地通过 warning 信息、API 注释 @*deprecated* 等做法，让用户逐步过渡到新用法上。

通过 mergeAlias 合并内置和用户自定义的 alias：

```typescript
function mergeAlias(
  a?: AliasOptions,
  b?: AliasOptions
): AliasOptions | undefined {
  // a不存在，直接返回b
  if (!a) return b
  // b不存在，直接返回a
  if (!b) return a
  // a、b都是对象，按顺序铺开
  if (isObject(a) && isObject(b)) {
    return { ...a, ...b }
  }
  // the order is flipped because the alias is resolved from top-down,
  // where the later should have higher priority
  return [...normalizeAlias(b), ...normalizeAlias(a)]
}
```

合并策略很简单，如果 a 不存在，直接返回 b；如果 b 不存在，直接返回 a；如果 a、b 都是对象，解构放进一个对象中，a 在前，b 在后；否则就调用 normalizeAlias 处理，这里的顺序是 b 在前， a 在后，我们进入 normalizeAlias 看看：

```typescript
function normalizeAlias(o: AliasOptions = []): Alias[] {
  // 数组的情况，返回每个alias配置调用normalizeSingleAlias后的结果
  // 是对象的情况，返回所有key对应的value调用normalizeSingleAlias后的结果
  return Array.isArray(o)
    ? o.map(normalizeSingleAlias)
    : Object.keys(o).map((find) =>
        normalizeSingleAlias({
          find,
          replacement: (o as any)[find]
        })
      )
}

function normalizeSingleAlias({
  find,
  replacement,
  customResolver
}: Alias): Alias {
  // 如果find是个字符串，并且find和replacement都以 / 结尾，处理掉最后的 /
  if (
    typeof find === 'string' &&
    find.endsWith('/') &&
    replacement.endsWith('/')
  ) {
    find = find.slice(0, find.length - 1)
    replacement = replacement.slice(0, replacement.length - 1)
  }

  const alias: Alias = {
    find,
    replacement
  }
  if (customResolver) {
    alias.customResolver = customResolver
  }
  return alias
}
```

从上述代码可以看到，alias 属性可以定义成对象或数组。是数组的情况，就遍历每一个别名做 normalizeSingleAlias，对象的情况，就将 key、value 进行 normalizeSingleAlias。

normalizeSingleAlias 判断 find、replacement 是否以 / 结尾，是的话就删除掉 /。这个规定来自  [@rollup/plugin-alias](https://www.npmjs.com/package/@rollup/plugin-alias "@rollup/plugin-alias")。然后返回 alias。

#### env

```typescript
 // 如果指定了 envDir，从 envDir 获取环境变量
  const envDir = config.envDir
    ? normalizePath(path.resolve(resolvedRoot, config.envDir))
    : resolvedRoot
  const userEnv =
    inlineConfig.envFile !== false &&
    loadEnv(mode, envDir, resolveEnvPrefix(config))
```

环境变量先从定义的 envDir 配置中获取文件路径，并做 normalizePath 处理。然后通过 resolveEnvPrefix 去解析环境变量的前缀：

```typescript
export function resolveEnvPrefix({
  envPrefix = 'VITE_'
}: UserConfig): string[] {
  envPrefix = arraify(envPrefix)
  // 如果定义了空的环境变量前缀，那么就能直接获取整个 process 的环境变量，这是危险的操作，所以给了一个敏感信息的提示
  if (envPrefix.some((prefix) => prefix === '')) {
    throw new Error(
      `envPrefix option contains value '', which could lead unexpected exposure of sensitive information.`
    )
  }
  return envPrefix
}
```

当我们在配置中定义 envPrefix 是空时，我们就能获取到整个 process 上的环境变量，也就获取到了一些敏感信息。所以 Vite 此时会直接抛出一个 Error 提醒你危险的配置。处理完前缀，接着就从文件中获取定义的环境变量：

```typescript
export function loadEnv(
  mode: string,
  envDir: string,
  prefixes: string | string[] = 'VITE_'
): Record<string, string> {
  // local 是 vite 内置的后缀名，比如 .env.development.local、.env.local，所以不给用
  if (mode === 'local') {
    throw new Error(
      `"local" cannot be used as a mode name because it conflicts with ` +
        `the .local postfix for .env files.`
    )
  }
  // 获取变量的前缀
  prefixes = arraify(prefixes)
  const env: Record<string, string> = {}
  
  // 默认会去读取的环境变量文件
  const envFiles = [
    /** mode local file */ `.env.${mode}.local`,
    /** mode file */ `.env.${mode}`,
    /** local file */ `.env.local`,
    /** default file */ `.env`
  ]

  // 通过CLI参数定义的一些环境变量中，如果带有 VITE 前缀，我们也能够从 import.meta.env 上获取到
  for (const key in process.env) {
    if (
      prefixes.some((prefix) => key.startsWith(prefix)) &&
      env[key] === undefined
    ) {
      env[key] = process.env[key] as string
    }
  }

  // 依次遍历环境变量文件
  for (const file of envFiles) {
    const path = lookupFile(envDir, [file], true)
    if (path) {
      // 获取文件内容并且通过 dotenv 解析
      const parsed = dotenv.parse(fs.readFileSync(path), {
        debug: process.env.DEBUG?.includes('vite:dotenv') || undefined
      })

      // let environment variables use each other
      dotenvExpand({
        parsed,
        // prevent process.env mutation
        ignoreProcessEnv: true
      } as any)

      // only keys that start with prefix are exposed to client
      for (const [key, value] of Object.entries(parsed)) {
        if (
          prefixes.some((prefix) => key.startsWith(prefix)) &&
          env[key] === undefined
        ) {
          env[key] = value
        } else if (
          key === 'NODE_ENV' &&
          process.env.VITE_USER_NODE_ENV === undefined
        ) {
          // NODE_ENV override in .env file
          process.env.VITE_USER_NODE_ENV = value
        }
      }
    }
  }
  return env
}
```

loadEnv 做了 3 件事：

- 获取环境变量前缀和定义 envFiles，当你在 development 下，会从 .env.development.local、.env.development、.env.local、.env 四个文件去获取环境变量；
- 读取进程的环境变量，如果有符合的前缀，就会被添加到 env 中，这个一般可以在启动 vite 时去设置环境变量；
- 然后依次读取环境变量文件，使用 [dotenv](https://www.npmjs.com/package/dotenv "dotenv") 去解析，使用 [dotenv-expand](https://www.npmjs.com/package/dotenv-expand "dotenv-expand") 去扩散。最后将 VITE 前缀的环境变量缓存到 env 中。

整个环境变量读取的过程就结束了。

## 总结

本节分析了从命令执行 vite 之后，通过从参数和配置文件 vite.config.ts 中获取配置。对于 ts 的配置文件，会先使用 esbuild 做 ts 编译和构建出 CJS 格式的产物，然后通过 require.extensions 扩展对 ts 文件的支持，最终拿到 vite.config.ts 的配置信息，与 CLI 的配置参数做合并，得到用户自定义的最终配置；

接着根据插件的 enforce 属性对用户定义的插件做排序，依次调用 config 钩子。全部配置都规范之后，会触发 configResolved 钩子。

最后分析了常用配置 alias 和 env 的处理过程，知道了 alias 以 @rollup/plugins-alias 为基础，env 借用 dotenv、dotenv-expand 包的力量，完成了环境变量的设置。
