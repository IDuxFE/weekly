# 从零发布自己的Vite插件

## 背景

最近有幸参与到了一个 Webpack 升级 Vite 的项目，升级过程中踩了不少坑，也花了不少时间，最后终于升级完成。期间的一个迁移点，涉及到 Vite 插件的编写，看了官方文档几遍，终于将插件搞定。

接下来，楼主就和大家分享：如何编写你的第一个 Vite 插件。

## 知识准备

Vite 官方文档详细地介绍了插件的工作机制、插件的各种钩子、插件的执行顺序。

建议大家先把[官网文档的插件知识](https://cn.vitejs.dev/guide/api-plugin.html)过一遍，做一个基础的知识准备，我们再开始编写自己的第一个插件。

## 插件的实现

我们在做移动端网页的时候，经常有需要在我们的手机上预览页面。平常我们的做法就是将 dev 启动的服务的地址复制到我们的手机上，再进行打开，步骤比较繁琐。

我们来试着将这步骤优化一下，在控制台直接输出地址的二维码，手机直接扫码就能打开网页，这是不是更加高效率了呢 😎

那么，我们就来实现这样的一个 Vite 插件

### 代码实现

#### 1. 新建项目

我们按照 vite 的官方预定来命名我们的插件，使用 typescript 来编写我们的的插件

```bash
$ mkdir vite-plugin-qrcode-terminal
$ cd vite-plugin-qrcode-terminal
$ npm init -y
$ yarn add vite typescript
$ tsc --init
```

如何在控制台输出二维码？我们依赖`qrcode-terminal`依赖包的能力

```bash
$ yarn add qrcode-terminal
```

略过一些额外的步骤，我们直接开始编写插件的核心代码

#### 2. 代码实现

首先我们先把我们的插件的主要内容编写完毕

```ts
// src/main.ts

export function vitePluginQrcodeTerminal(params: PluginOption = {}): Plugin {
  let config: ResolvedConfig;
  return {
    // 插件名称
    name: 'vite-plugin-qrcode-terminal',
    // 本地服务时运行
    apply: 'serve',
    // 拿到最后解析完成的配置项
    configResolved(resolvedConfig) {
      config = resolvedConfig;
    },
    // 在configureServer钩子执行我们的方法
    configureServer(server) {
      return () => {
        server.middlewares.use((req, res, next) => {
          createQrcode(params, config.server);
          next();
        });
      };
    },
  };
}
```

其实再完善我们的核心函数`createQrcode`

```ts
function createQrcode(
  params: PluginOption,
  server: ServerOptions & ResolvedServerOptions
) {
  const { small } = params;
  const content = getInputContent(params, server);
  // 根据用户配置生成完整的url数组
  const urls = renderUrl(content, server);
  urls.forEach((each) => {
  // 生成二维码
    qrcode.generate(each, { small: small ?? true }, (qrcode) => {
      console.log('------');
      console.log(each);
      console.log(qrcode);
      console.log('------');
    });
  });
}
```

篇幅有限，完整的源码，可以看这里 [vite-plugin-qrcode-terminal](https://github.com/AFine970/)

#### 3. 代码打包

我们的代码编写完成之后，我们要将它进行打包，这里我推荐大家使用`tsup`作为打包工具。`tsup`是一个基于 esbuild 的 cli 工具，能力非常强大且便捷，这里不做过多介绍。

有兴趣的朋友，可以去[tsup 官方文档](https://tsup.egoist.dev/)看看

```json
// package.json

{
   "scripts": {
        "build": "rm -rf dist/ && tsup src/main.ts --dts --no-splitting"
    }
}
```

![Snipaste_2022-09-04_20-22-49.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/605602c1ba2c405ba4503829c9ca3de6~tplv-k3u1fbpfcp-watermark.image?)

#### 4. 测试验证

我们将插件进行打包构建，将将插件链接到全局

```bash
$ yarn link # 将插件链接到全局
```

我们新建一个 Vite 项目进行验证，引用我们的插件进行验证

```bash
$ yarn link "vite-plugin-qrcode-terminal"
```

```js
// vite.confg.js

import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { vitePluginQrcodeTerminal } from 'vite-plugin-qrcode-terminal';

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue(), vitePluginQrcodeTerminal()],
  server: {
    host: true,
  }
})
```

配置好了之后，启动 dev

```bash
$ yarn vite
```

我们可以在控制台看到输入的二维码

![Snipaste_2022-09-04_23-02-27.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8d736d2f6b0420bb65be27359e073dd~tplv-k3u1fbpfcp-watermark.image?)

## 发布依赖包

完成代码实现和测试工作后，接下来就是将我们的插件发布到`npm.js`上

1. 设置要进行发布的包的内容

```json
// package.json
{
    "name": "vite-plugin-qrcode-terminal",
    "version": "0.0.1",
    "description": "create QR code in terminal",
    "types": "dist/main.d.ts", // ts类型声明
    "main": "dist/main.js", // 依赖包主入口
    "files": ["dist"] // 依赖包发布内容白名单
    ...
}
```

2. 注册并登陆`npm.js`，并把 npm 源改为 npm 官方源（taobao 源的话，无法发包到官网）

```bash
$ npm login # 登录
$ npm who am i # 查看是否登录成功，当前登录的是谁
```

![login.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1644ddf383074d34a1c34f892eab933b~tplv-k3u1fbpfcp-watermark.image?)

3. 发布

```bash
$ npm publish
```

![publish.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18e01ee899fe40d59e3f48d6c47f52ad~tplv-k3u1fbpfcp-watermark.image?)

命令执行完毕，我们就可以在`npm.js`上看到我们发布的依赖包[vite-plugin-qrcode-terminal](https://www.npmjs.com/package/vite-plugin-qrcode-terminal)

![插件截图.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/52a54bb5a70d4382bea53f4850544c53~tplv-k3u1fbpfcp-watermark.image?)

4. 额外的补充

- 关于版本号管理，推荐使用`standard version`插件
- 完成插件的编码之后，还有一个最重要的事情就是编写`README.md`文档，好的文档是插件饱受欢迎的第一步
- 觉得自己的插件编写得不错，可以通过 PR 的方式将你的插件添加到 Vite 官方维护的[awesome-vite](https://github.com/vitejs/awesome-vite#plugins)列表中

## 最后

Vite 的发展趋势不可阻挡，项目从 Webpack 升级 Vite 之后，大家用了都说香。开发环境冷启动、热更新速度对比 Webpack 提升 90%左右，打包构建提升 60%左右，大大地提升了开发幸福感

希望大家看完这篇文章，都能学会编写自己的 Vite 插件，积极拥抱 Vite

好好学习不会差，我是 970，祝大家越来越强，咱们下回再见
