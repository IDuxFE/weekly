## 背景

我们知道`WindiCSS`的配置文件既支持`js`后缀也支持`ts`后缀，即：`windi.config.js`和`windi.config.ts`
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ef268b52ce6a47f0b12434d89ebf7e00~tplv-k3u1fbpfcp-watermark.image?)

我们在 vscode 安装的`WindiCSS IntelliSense`也支持读取多种后缀格式的配置文件。vscode 基于 electron 实现，electron 底层是一个`node.js + v8`的集成终端，支持运行 js 格式的代码，`WindiCSS IntelliSense`最终打包出来的运行的代码也是 js 格式的代码

> **那`WindiCSS IntelliSense`是怎么实现 js 文件加载 ts 文件的呢？**

这个问题困惑了我大半天，正好最近在写脚手架，需要支持加载 ts 后缀的配置文件

于是下定决心，把这个问题搞清楚，最后终于在源码里找到了答案

## 解惑

### `WindiCSS IntelliSense`源码

1. 我们将`WindiCSS IntelliSense`源码直接`clone`下来

```bash
$ git clone https://github.com/windicss/windicss-intellisense.git
```

2. 找到读取配置文件的核心代码

```ts
<!--src\lib\index.ts-->

// ...
async init() {
    try {
      const config = await this.loadConfig();
      this.processor = new Processor(
        config
      ) as Processor;

      this.attrPrefix = this.processor.config('attributify.prefix') as
        | string
        | undefined;
      this.variants = this.processor.resolveVariants();
      this.colors = flatColors(
        this.processor.theme('colors', {}) as colorObject
      );
      this.register();
    } catch (error) {
      Log.error(error);
    }
  }
  // ...
```

3. 关键实现

```ts
<!--src\lib\index.ts-->

// ...
import { loadConfig } from 'unconfig';

// ...
async loadConfig(file?: string) {
    if(!workspace.workspaceFolders) return;
    const { config, sources } = await loadConfig<Config>({
      sources: [
        {
          files: 'windi.config',
          extensions: ['ts', 'mts', 'cts', 'js', 'mjs', 'cjs'],
        },
        {
          files: 'tailwind.config',
          extensions: ['ts', 'mts', 'cts', 'js', 'mjs', 'cjs'],
        },
      ],
      merge: false,
      cwd: workspace.workspaceFolders[0].uri.fsPath,
    });
    Log.info(`Loading Config File: ${sources}`);
    return config;
  }
// ...
```

从关键实现的代码上看，我们就找到了答案：**使用[unconfig](https://www.npmjs.com/package/unconfig)来实现加载不同后缀的文件**
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ce2ede5406040708ff38f7d7c9891b9~tplv-k3u1fbpfcp-watermark.image?)

看到这里，我们还是没有搞懂：**为什么`js`能加载`ts`**，我们继续深入了解一下`unconfig`的内部实现

### `unconfig`源码

1. 我们将`unconfig`源码直接`clone`下来

```bash
git clone https://github.com/antfu/unconfig.git
```

2. 核心代码

```ts
<!--src\index.ts-->

// ...
import jiti from 'jiti'
// ...

async function loadConfigFile<T>(filepath: string, source: LoadConfigSource<T>): Promise<LoadConfigResult<T> | undefined> {
  let config: T | undefined

  let parser = source.parser || 'auto'

  let bundleFilepath = filepath
  let code: string | undefined

  async function read() {
    if (code == null)
      code = await fs.readFile(filepath, 'utf-8')
    return code
  }

  if (source.transform) {
    const transformed = await source.transform(await read(), filepath)
    if (transformed) {
      bundleFilepath = join(dirname(filepath), `__unconfig_${basename(filepath)}`)
      await fs.writeFile(bundleFilepath, transformed, 'utf-8')
      code = transformed
    }
  }

  if (parser === 'auto') {
    try {
      config = JSON.parse(await read())
      parser = 'json'
    }
    catch {
      parser = 'require'
    }
  }

  try {
    if (!config) {
      if (typeof parser === 'function') {
        config = await parser(filepath)
      }
      else if (parser === 'require') {
        config = await jiti(filepath, {
          interopDefault: true,
          cache: false,
          requireCache: false,
          v8cache: false,
          esmResolve: true,
        })(bundleFilepath)
      }
      else if (parser === 'json') {
        config = JSON.parse(await read())
      }
    }

    if (!config)
      return

    const rewritten = source.rewrite
      ? await source.rewrite(config, filepath)
      : config

    if (!rewritten)
      return undefined

    return {
      config: rewritten,
      sources: [filepath],
    }
  }
  catch (e) {
    if (source.skipOnError)
      return
    throw e
  }
  finally {
    if (bundleFilepath !== filepath)
      await fs.unlink(bundleFilepath).catch()
  }
}

// ...
```

3. 把核心代码进行精简，找到关键实现

```ts
<!--src\index.ts-->

// ...
import jiti from 'jiti'
// ...

async function loadConfigFile<T>(filepath: string, source: LoadConfigSource<T>): Promise<LoadConfigResult<T> | undefined> {
// ...
    try {
    if (!config) {
      if (typeof parser === 'function') {
        config = await parser(filepath)
      }
      else if (parser === 'require') {
        config = await jiti(filepath, {
          interopDefault: true,
          cache: false,
          requireCache: false,
          v8cache: false,
          esmResolve: true,
        })(bundleFilepath)
      }
      else if (parser === 'json') {
        config = JSON.parse(await read())
      }
    }
    // ...
}

// ...
```

从关键实现的代码上看，我们就找到了答案：**使用[jiti](https://www.npmjs.com/package/jiti)来实现 js 文件加载 ts 文件时，动态编译 ts 文件并返回结果。**

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b9619d30b644587bb6209a6bdf3a6e6~tplv-k3u1fbpfcp-watermark.image?)

`jiti`文档中这么描述：**`Runtime typescript and ESM support for Node.js (CommonJS)`**，我们可以更加粗暴地理解为`require-ts`。

为了让大家更好地理解`unconfig`的工作流程，楼主根据上述的`unconfig`核心代码，整理出一个`unconfig核心工作原理`流程图

![20220929202142.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e59c4b1c9aac47a6933d5f3cb4ba2b8a~tplv-k3u1fbpfcp-watermark.image?)

关于`js`文件如何加载`ts`文件的疑惑得以解开

### 代码实践

> 看过没练过，等于没看过 - B 站梦觉教游泳

我们在写脚手架的时候可以直接使用`unconfig`读取配置文件即可，例如：读取`vite.config.ts`可以这么实现

```ts
import { loadConfig } from 'unconfig'

const { config } = await loadConfig({
  sources: [
    {
      files: 'vite.config',
      async rewrite(config) {
        return await (typeof config === 'function' ? config() : config)
      },
    },
  ]
})
```

## 最后

如果你觉得这篇文章对你有帮助，记得动动鼠标点个赞，你的点赞是我更新文章的最大动力

> 好好学习不会差，我是 970，咋们下期再见（此处应有 bgm）
