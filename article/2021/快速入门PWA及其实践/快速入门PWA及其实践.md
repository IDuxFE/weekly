## 前言

本篇文章主要介绍 PWA 应用，看完这篇文章，你可以了解

- PWA 是什么？
- PWA 的优点和缺点
- PWA 应用的配置项是如何工作的
- 快速配置一个入门的 PWA 应用，并在本地进行调试

## PWA 的一些概念

PWA 英文全称为：`Progressive Web Apps`，中文翻译为：渐进式网页应用

可以类比为：原生的 web 小程序

**主要的优点**

- 可被浏览器主动发现
- 易安装，手机上可以把应用图标发往手机桌面，电脑上可以安装在浏览器中
- 独立于网络，可以离线访问，主动推送消息，资源预缓存以及离线优先
- 渐进式，向下兼容
- 响应式，多端适配

**主要的缺点**

- 浏览器兼容性差，目前该特性只有新版本谷歌浏览器、火狐浏览器、Edge 浏览器支持

**解决的问题**

如何正确缓存网站资源并使其在离线时可用的问题

主要利用了浏览器提供的 `Service Worker`的能力，`Service Worker` 运行在一个与页面 JavaScript 主线程独立的线程上，可以控制网络请求、消息推送和执行指定的任务。利用控制网络请求的能力，可以实现页面离线访问。[ Service Worker 解释 ](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps/Offline_Service_workers " Service Worker 解释 ")

**配置三要素**

- 可靠的`Https`网络协议
- `service-worker.js`
- `manifest.json`

![PWA配置三要素.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a05af63d9184c9d8c1a1abd644d107b~tplv-k3u1fbpfcp-watermark.image?)

更加完整的资料：[渐进式 Web 应用（PWA）](https://developer.mozilla.org/zh-CN/docs/Web/Progressive_web_apps "渐进式 Web 应用（PWA）")

## 快速配置

下面给大家介绍 PWA 应用在使用`webpack`上如何配置

快速创建一个 webpack 配置的项目

目录结构如下

```text
|-- my-pwa-app
    |-- .gitignore
    |-- index.html
    |-- package-lock.json
    |-- package.json
    |-- webpack.config.js
    |-- src
        |-- main.js
        |-- assets
            |-- logo.png
```

创建完毕之后，我们安装关键依赖包

- `workbox-webpack-plugin`用于生成`service-worker.js`文件
- `webpack-pwa-manifest`用于生成`manifest.json`文件
- `register-service-worker`用于帮助我们快速注册`service-worker.js`文件

```bash
npm install workbox-webpack-plugin webpack-pwa-manifest -D

npm install register-service-worker
```

首先配置`workbox-webpack-plugin`和`register-service-worker`

```js
// webpack.config.js
/*
 * @Date: 2021-11-03 21:43:51
 * @LastEditors: cunhang_wwei
 * @LastEditTime: 2021-11-03 22:19:43
 * @Description: webpack打包文件
 */
const path = require('path');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const WorkboxWebpakcPlugin = require('workbox-webpack-plugin');

module.exports = {
    mode: 'production',
    entry: './src/main.js',
    plugins: [
        new HtmlWebpackPlugin({
            template: './index.html'
        }),
        new WorkboxWebpakcPlugin.GenerateSW({
            skipWaiting: true,
            clientsClaim: true
        })
    ],
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js'
    }
};
```

```js
// register-service-worker.js
/*
 * @Date: 2021-11-03 22:20:06
 * @LastEditors: cunhang_wwei
 * @LastEditTime: 2021-11-03 22:21:24
 * @Description: 注册service-worker
 */

import { register } from "register-service-worker";

register('/service-worker.js', {
    registrationOptions: { scope: './' },
    ready(registration) {
        console.log('Service worker is active.')
    },
    registered(registration) {
        console.log('Service worker has been registered.')
    },
    cached(registration) {
        console.log('Content has been cached for offline use.')
    },
    updatefound(registration) {
        console.log('New content is downloading.')
    },
    updated(registration) {
        console.log('New content is available; please refresh.')
    },
    offline() {
        console.log('No internet connection found. App is running in offline mode.')
    },
    error(error) {
        console.error('Error during service worker registration:', error)
    }
})
```

```js
// main.js
/*
 * @Date: 2021-11-03 21:53:07
 * @LastEditors: cunhang_wwei
 * @LastEditTime: 2021-11-03 22:23:31
 * @Description: 主入口
 */

import './register-service-worker'

const name = 'pwa app'

console.log(name);
```

我们将项目打包`npm run build`，并在浏览器上运行（如何快速启动本地打包后的文件，可以参考我之前写的的一篇文章：[本地如何快速启动 Vue 或 React 打包后的文件？](https://juejin.cn/post/7026007570947309605 "本地如何快速启动 Vue 或 React 打包后的文件？")）

我们看到`service-work`已经开始工作

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6eccdc5da9584775b4e5be3612abfd52~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e9e1f09acc374a6dab1d678bdbdfb40c~tplv-k3u1fbpfcp-watermark.image?)

我们的 pwa 应用配置已经完成了一半！

但是浏览器无法识别出这是一个 PWA 应用，因为我们没有配置`manifest.json`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ae3fa88ac7624774a0c3d4d6fa239f75~tplv-k3u1fbpfcp-watermark.image?)

配置`webpack-pwa-manifest`

```js
// webpack.config.js
plugins: [
    new HtmlWebpackPlugin({
        template: './index.html'
    }),
    new WorkboxWebpakcPlugin.GenerateSW({
        skipWaiting: true,
        clientsClaim: true
    }),
    // 需要注意 WebpackPwaManifest 需要在 HtmlWebpackPlugin 之后执行
    new WebpackPwaManifest({
        name: 'My Progressive Web App',
        short_name: 'MyPWA',
        description: 'My awesome Progressive Web App!',
        theme_color: '#123124',
        background_color: '#111',
        crossorigin: 'use-credentials', //can be null, use-credentials or anonymous
        icons: [
            {
                src: path.resolve('src/assets/logo.png'),
                sizes: [192, 512] // multiple sizes
            }
        ]
    })
],
```

我们再次打包`yarn build`，发现多生成了`manifest.json`文件

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44237608a63d4ef8a26b67f6de3884e8~tplv-k3u1fbpfcp-watermark.image?)

`index.html`里面也多了`<meta>`和`<link>`标签

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b24768681df14fb4b5477ee8d6b928b8~tplv-k3u1fbpfcp-watermark.image?)

我们在浏览器里运行

看到浏览器已经识别出了`manifest.json`，地址栏也多了一个收藏为应用的快捷键

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c23fc26ee84c4c0bae0b78af70a0e033~tplv-k3u1fbpfcp-watermark.image?)

我们的 PWA 应用配置就大功告成了 ✨

## 总结和回顾

**配置三要素**

- 可靠的`Https`网络协议
- `service-worker.js`
- `manifest.json`

**配置三剑客**

- `service-work.js`文件由`workbox-webpack-plugin`生成，定义`Service Worker`如何工作
- `webpack-pwa-manifest`插件需要配置在`html-webpack-plugin`插件之后。`webpack-pwa-manifest`插件在·`html-webpack-plugin`插件生成`html`文件之前，根据配置的`PWA`的信息，增加`<meta>`和`<link>`标签
- `register-service-worker`将`service-worker.js`在浏览器中进行注册

**QA**

- 如何知道这个是完整配置 PWA 应用？
  - `F12`调试控制台里面的`Lighthouse`可以探测是不是一个合格配置的 PWA 应用
- pwa 应用缓存和浏览器缓存的有什么区别？
  - `service-worker`可以监听`fetch`事件并劫持浏览器请求，浏览器缓存主要依赖请求头字段。`service-worker`可以根据配置拉取资源供后续使用，浏览器缓存仅对请求的资源文件，无法对所有资源进行掌控

## 最后

本文章从 PWA 的理念入手，帮助大家快速的配置入门的 PWA 应用，该应用具备了资源预缓存、离线加载和浏览器主动识别的能力。

配置一个好的 PWA 应用，还涉及到消息推送、消息订阅和离线数据复用等难点，革命道路还很长。希望该文章能起到抛转引玉的作用。

如果觉得文章对你有帮助，不要忘记点个赞！

代码仓库地址：<https://github.com/AFine970/my-pwa-app>

本文作者：我是970

原文地址：<https://juejin.cn/post/7029542245082595359>