# vue-cli 老项目 vite 迁移记录

公司的项目大部分都是 2B 类型的中后台 PC 端项目，且业务都非常庞大。常年累月的维护下，使用`webpack dev`速度变得非常慢。

因为最近的工作主要是负责一些前端的基础设施建设，各个业务线的产品都有一定的接触。有一个项目在我的电脑上面`npm run dev`居然要 6 分钟之久，实在是难以感到愉悦。加上公司内部已经新项目使用 vite 作为构建工具，所以挑选了一个自己比较熟悉的老项目(vue-cli)进行 vite 迁移。并将过程进行了记录。

## 项目背景

先简单说明一下项目的情况：

* 内容：2B 的 PC 端中后台管理系统，是一个双入口的应用（登录页一个入口，管理系统一个入口）。

* 构建工具：vue-cli-service
* 技术栈：vue 2.6 + Axios + Vuex + Vue-Router + sf-vue-componet(公司内部组件库) + ElementUI 
* 维护时长：3 年+，（实际上应该远远超过五年，这个项目是以一个已经维护了很久的项目为基础进行开发的，而当前最早的一条 commit 记录是 3 年前，但是这个基础项目已经开发很久时间了）
* 模块数量：6000（npm run serve 时显示的模块数量）
* 业务代码量：19.3W 行
* 兼容性要求：IE11

项目的体量在公司内部只能算是中等，之所以挑选这个项目进行处理是因为对这个业务比较熟悉。话不多说，开搞。

## 思路分析

本次的目的不是直接用`vite` 替换 `vue-cli`，而是想同时保留`vite`和`vue-cli`。除去影响开发体验也主要是集中在 dev 这一块的原因外，还有以下两点：

* 产品需要兼容`IE11`，尚不清楚`vite`在此方面有多少坑。步子迈太大了，可能扯着胯。
* `rollup` 打包机制毕竟和 `webpack` 还是不同，且业务也比较复杂。修改完成之后也很难保证所有业务都能测试到位。

`webpack` 和 `vite` 本质上都是入口文件 + 依赖分析 + 模块转换。vite 是 使用的时候进行处理（vite 针对第三方库有同样会进行预处理），`webpack`则是先全部处理完毕。当浏览器天然支持`ES module`之后，实时处理变得可能。这也是`vite` 启动速度快的原因。

所以，从理论上来说，从`Vue-cli`迁移到 `vite` 是可行的。

## 开始迁移

### step1 分析 vue.config.js

首先，我们先进行 vue.config.js 进行分析。主要是把一些插件，和比较特殊的配置整理出来。然后再去 vite 官网和社区查看，看是匹配的配置或者插件。把其中不匹配的部分进行解决，估计迁移也就差不多了。

分析完成之后主要两点无法匹配的情况

```js
// vue.config.js
module.exports = {
    chainWebpack: config => {
        config.plugins.delete('preload-loginPlatform');
        config.plugins.delete('prefetch-loginPlatform');
        config.plugins.delete('preload-platform');
        config.plugins.delete('prefetch-platform');
    },
    transpileDependencies: ['@sxf/cloudsec-components', '@sxf/validations', '@sxf/sf-vue-component']
}
```

chainWebpack 中的配置主要是为了解决两个入口资源互相加载的问题，即访问登录页时，会预加载管理系统资源。由于这个是打包后在生产环境中才会用到的内容，且这次迁移无需关系打包的内容，所以暂时不用处理。

transpileDependencies 主要是让 node_modules 中的文件也进行 babel 的处理，而我们在开发时可以使用最新的浏览器，完全可以不用 babel，所以这个问题也不用处理。 

> https://cli.vuejs.org/zh/config/#transpiledependencies

### step2 安装依赖，添加 scripts

一般情况下，会需要用到以下的 npm 包。

* vite
* vite-plugin-vue2：vite 支持 vue2 的插件（默认情况下，vite 并不支持 vue）
* vite-plugin-eslint（可选）：vite eslint 插件，用于在 run dev 时，实时校验 eslint
* vite-plugin-legacy（可选）：处理浏览器兼容性，本次这里不需要。
* vite-plugin-mock（可选）：mock 数据

```shell
yarn add vite vite-plugin-vue2 -D # 这里只安装了必要的vite 与 vite vue2插件
```

然后添加 package.json 中 添加 scripts 命令。

```json
// package.json
{
    ...,
    "scirpts": {
        ...,
        "dev": "vite"
	}
}
```

### step3 编写 vite.config.js 文件

因为原本的 vue.config.js 不是特别复杂，网上也有其他迁移的文章讲解如何迁移`webpack`配置，所以我这里直接粘贴结果。

```js
const { createVuePlugin } = require('vite-plugin-vue2')
const { resolve } = require('path');
const config = require('./config');
module.exports = {
    server: {
        host: '0.0.0.0',
        https: true,
        port: 4433,
        proxy: {
            '/platform': {
                target: `https://${config.host}`,
                secure: false,
                changeOrigin: true,
                rewrite: (path) => path.replace(/^\/platform/, '')
            },
            ... // 
        }
    },
    resolve: {
        alias: {
            '@': resolve(__dirname, './src'),
            'src': resolve(__dirname, './src'),
            'home': resolve(__dirname, './src/home'),
            'components': resolve(__dirname, './src/components'),
            'assets': resolve(__dirname, './src/assets'),
            'utils': resolve(__dirname, './src/utils'),
        }
    },
    build: { // 这里是build时才会用到，所以这里可以省略
      rollupOptions: {
        input: {
          login: resolve(__dirname, './public/login_platform.html'),
          platform: resolve(__dirname, './public/platform.html')
        }
      }
    },
    plugins: [
        createVuePlugin()
    ]
}
```

从上面的结果来看其实只用写三个部分的内容：

1. serve: 开发时的运行端口，以及数据请求的反向代理的规则
2. alias: 别名
3. plugins: 引入`vite-plugin-vue2`用于加载 vue 文件。

#### 关于多入口

在项目介绍的时候说到，项目是一个多入口的文件。实际上在 vite 的开发环境中，多入口无需配置。启动文件之后直接访问对应的 html 资源即可。

#### 关于 wp2vite

> https://github.com/tnfe/wp2vite (一个前端项目转换工具，可以让 webpack 项目支持 vite)

wp2vite 能够将根据你当前的 webpack 配置生成 vite 配置（支持 vue-cli），如果不想手写 vite.config.js 可以使用该工具进行自动转换。我是在撰写本文整理资料时，才发现有此工具。

wp2vite 转换的结果并非完全正确，存在以下问题。

* alias 添加了一些莫名其妙的别名 （也有可能是 Feature :smile:）
* proxy 配置 rewrite 不正确

以上两点，各位同学在参考的时候注意一下就行。

### step 4 调整入口文件

与 `vue-cli` 有区别的是，`vite` 默认只能将 html 作为入口文件，所以需要对 HTML 内容进行调整。

**别忘记给 script 标签添加 type="module"**

```html
<!-- index.html 主应用的入口文件 -->
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    ...
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title>...</title>
    <!-- 添加这一行直接引入之前的 入口文件 -->
    + <script type="module" src="/src/main.js"></script>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but project doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>

<!-- login.html 登录页的入口文件 -->
<!DOCTYPE html>
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    ...
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title>...</title>
    <!-- 添加这一行直接引入之前的 入口文件 -->
    + <script type="module" src="/src/login.js"></script>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but project doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>

```

### step 5 npm run dev

直接运行 npm run dev，可以发现 vite 马上就开启了服务。相对于原来 70s 左右的时间，这快了可不止一星半点，简直就是火箭起飞。

![image-20210906144343218](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffo0slyj60dy03u3ym02.jpg)

当然，服务的启动并不代表迁移已经完成。vite 更多的是在访问对应的资源时才会去处理内容。所以需要访问对应的页面来进行验证。**而真正的迁移，才刚刚开始 :sob:**

## 无尽采坑：阶段一

### 问题 1：URIError: URI malformed

vite 开启之后，访问登录页 https://localhost:4433/static/login_platform.html，报错 URIError: URI malformed

![image-20210906145521690](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffoqhfoj60n6082wgp02.jpg)

这是`decodeURIComponent`函数在执行时报的错。

通过在错误堆栈中，加上 console 语句，得知出错的 URI 是`/static/%3C%=%20BASE_URL%20%%3Efavicon.ico`

对应的代码则是 login.html 中的代码，这里的 `BASE_URL`是 vue-cli 中模板变量的写法，这里可以修改为`/public/favicon.ico`（根据自己的目录结构调整，切勿盲目套用）

```html
<link rel="icon" href="<%= BASE_URL %>favicon.ico">
=>
<link rel="icon" href="/public/favicon.ico">
```

修改完成之后，vite 则提示去掉 public 前缀，那我们直接去掉就行。

![image-20210906150224403](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffovdwej60bu011aa202.jpg)

### 问题 2：Failed to resolve import "xxx"

![image-20210906154851425](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffpvpd8j60rc04vdgq02.jpg)

提示找不到对应的 vue 文件（实际肯定是有）。经过查阅资料这一块主要是为了类型提示的问题（ts 的项目）,vite 不在自动查找.vue 扩展名的文件。

这里只需要补全`PageLogin.vue`即可。 在这里补全`.vue`扩展名不太现实，因为项目里面大部分都是省略了扩展名的。好在官方给出了解决的方案，直接配置即可。

> https://cn.vitejs.dev/config/#resolve-extensions
>
> **resolve.extensions**
>
> - **类型：** `string[]`
> - **默认：** `['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json']`
>
> 导入时想要省略的扩展名列表。注意，**不** 建议忽略自定义导入类型的扩展名（例如：`.vue`），因为它会影响 IDE 和类型支持。

#### 关于问题 2

配置**resolve.extensions**的方式，并不能够完全的解决问题。这里卖个关子，后面会继续遇到因为缺少`.vue`扩展名而导致的问题。那里会给出完美的解决方案。

### 问题 3: less import

修复完问题 2 之后，继续访问页面。这个时候又出了新的问题。

![image-20210906151020412](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffqfudvj60m7055wf802.jpg)

经过查阅资料，vite 对 less 和 sass 文件的 import 进行了优化的处理，不在需要在别名前添加 `~`符号来进行识别。

> https://vitejs.bootcss.com/guide/features.html#css-pre-processors
>
> vite 为 Sass 和 Less 改进了 `@import` 解析，以保证 vite 别名也能被使用。另外，`url()` 中的相对路径引用的，与根文件不同目录中的 Sass/Less 文件会自动变基以保证正确性。
>
> 由于 Stylus API 限制，`@import` 别名和 URL 变基不支持 Stylus。

这里就只需要全局搜索然后全局替换就行，如果有发现全局没有替换到的（多了空格什么的），手动替换一下就行。

![image-20210906151548730](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffr9vhmj60am083dgo02.jpg)

### 问题 4： net::ERR_ABORTED 408 (Request Timeout)

修复好问题 3，再次访问时出现很多资源 408 的错误。

![image-20210906152826856](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffrrmzwj60lm03eq3p02.jpg)

出现这个问题不要慌，重新刷新即可，这是 vite 在对一些依赖进行预构建。因为是请求到对应的资源才会进行预构建，在碰到需要预构建的包后，vite 直接返回了`408`。

**有时候需要重新刷新多次，每次构建完成之后重新刷新，如果发现新的需要预构建的依赖会再次返回`408`**

![image-20210906153255734](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffsbdw7j60qy046myh02.jpg)

至于这里为什么返回`408`，而不是直接挂起请求预构建完成之后再返回，个人不是很清楚。有了解 vite 原理的同学可以解答下。

> https://vitejs.bootcss.com/guide/dep-pre-bundling.html#the-why
>
> vite 将有许多内部模块的 ESM 依赖关系转换为单个模块，以提高后续页面加载性能。
>
> 一些包将它们的 ES 模块构建作为许多单独的文件相互导入。例如，[`lodash-es` 有超过 600 个内置模块](https://unpkg.com/browse/lodash-es/)！当我们执行 `import { debounce } from 'lodash-es'` 时，浏览器同时发出 600 多个 HTTP 请求！尽管服务器在处理这些请求时没有问题，但大量的请求会在浏览器端造成网络拥塞，导致页面的加载速度相当慢。
>
> 通过预构建 `lodash-es` 成为一个模块，我们就只需要一个 HTTP 请求了！

### 问题 5：Vue warn：runtime-only

等待所有内容都预构建完成之后，又出现了新的问题。

![image-20210906153921859](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffsp5knj60z305hab702.jpg)

这里问题比较好理解，vue 的默认引用的内容是不包含模板编译功能的。因为之前使用的是 vue-cli，毕竟是一家的东西 vue-cli 针对这个情况做了特殊的处理。

这种情况在 vite.config.js 中，添加一条别名即可解决。

> https://github.com/underfin/vite-plugin-vue2/issues/100

```js
// vite.config.js
{
    resolve: {
        alias: {
            '@': resolve(__dirname, './src'),
            ......,
            'vue': require.resolve('vue/dist/vue.esm.js')
        }
    }
 }
```

### 问题 6：require is not defined

![image-20210906160418206](https://tva1.sinaimg.cn/large/008i3skNgy1gu8fftpuutj612y0abjvm02.jpg)

这里继续爆出来 import 的问题，但是这里肯定不是因为确实 .vue 扩展名引起的。经过分析，问题的原因是业务代码中使用了 require 语句，而 vite 主要是依赖于 ES module 机制。这也就导致无法识别该业务代码，进而导致该模块加载失败。

> 这里吐槽下，这里 vite 控制台 和 浏览器控制台爆出了不同的错误，而 vite 控制台的错误信息完全没法排查思路。
>
> 而在迁移前期，一直盯着 vite 控制台，这就导致一个很简单的问题实际上花费了很多的时间。

这种情况下，需要手动对业务代码进行处理。

```js
{
    image: require('@uedc/login/dist/static/copyright/copyright-zh_CN@2016-2021.png')
}
// => 
import copyright from '@uedc/login/dist/static/copyright/copyright-zh_CN@2016-2021.png';
{
    image
}
```

## 采坑阶段一总结

修改完上述问题之后，本项目的多入口应用中的一个登录页面入口，终于可以正常展示了。

![image-20210906162449835](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffumz6oj60yb0px43502.jpg)

* `vite` 控制台，chrome 控制台均无报错
* 之前修改的 require 改为 import 的 logo，正常显示
* 通过网络请加载的验证码，正常显示（说明反向代理这些都是正常的）

虽然遇到问题还是比较多，但是解决起来都不是很费时间，排查起来也比较顺利。至此迁移工作可以说已经进行了一半。除去 html 模板文件中修改`favicon.ico` 和导入图片的`requrie`之外，没有改动其他的业务代码，对业务的入侵很小。

话不多说，输入账号密码，开启第二个阶段的采坑。

## 无尽采坑：阶段二

### 问题 7： uses lang html for template

![image-20210906165431300](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffv5mwcj60ys02wmy402.jpg)

这是一个相当无语的错误，应该是`vite-plugin-vue2`插件的 bug。问题的原因是在 template 中，写了`lang="html"`

```html
<template lang="html">
	<div>
        .....
    </div>
</template>
```

虽然模板默认就是 html，这个声明时多此一举，但是报错感觉不应该。

好在这个问题比较好解决，通过查找字符串全局替换即可。你问我为什么要多此一举要声明`lang="html"`?我也不知道，这不就是老项目的魅力嘛:slightly_smiling_face:。

![image-20210906165833301](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffvykbpj607m06ht9002.jpg)

### 问题 8： npm 包出错

![image-20210906170953982](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffwyadej60eq04o0tc02.jpg)

这也是本次迁移过程中，最为棘手的问题。不止一个 npm 包出错，且出错的内容也不一样。有些出错可以通过 vite 的配置解决，有些则无法解决。

一般来说有以下几种方法来解决 npm 包报错的问题。

#### 方法一：修改别名，指向源码

即在 vite.config.js 中添加 alias

```js
'@sxf/sp-qrcode': resolve(__dirname, './node_modules/@sxf/sp-qrcode/src/index.js'),
```

这种方法适用于 npm 包的源码是用 ES module 规范编写的，且上传的 npm 包中包含了源码。

#### 方法二：修改别名，指向 esm 格式的打包文件

修改方式同方法一，只不过是执行打包后的 esm 文件。

这种方式适用于 npm 包已经 build 了 esm 模块的内容，但是未在 packages.json 中提供 module 字段指向对应的文件。

一般开源的，使用人数较多的 npm 包不会出现这种情况，只是因为这次出问题有一些是内部的包，确实存在 build 了 esm 的文件，但是没有在 package.json 中指明的情况。

#### 方案三：optimiziDep.include

> https://cn.vitejs.dev/config/#optimizedeps-include
>
> 默认情况下，不在 `node_modules` 中的，链接的包不会被预构建。使用此选项可强制预构建链接的包。

从官网的文档的描述信息来看`optimiziDep.include`是无法解决 npm 包出错的问题。因为不管怎样，node_modules 中的内容都会被预构建。但是从实际的使用中来看

* 在`vite 2.4.4` 中，`optimiziDep.include`能够解决部分 npm 加载的问题。
* 在 `vite 2.5.3` 中，`optimiziDep.include`不再起作用。

因为 vite 一直在高速迭代，保不齐部分情况下此方法依旧有用。

#### 方案四：参考问题 10

问题 10 也是 npm 出问题，但是这个问题比较具有代表性，所以单独拿出来。

#### 方案无：换个 npm 包，或者修改 npm 包

如果上面三种方案都无法解决，那就只能从 npm 包本身下手了。

对于自己内部维护 npm 包，可以重新打包一个 ES module 的文件并发布。这里推荐用 rollup 打包，能够很轻松的构建`CommonJS`、`ES module`、`UMD`、`IIFE`格式的文件

> https://rollupjs.org/guide/en/#outputformat

对于第三方的 npm 包，出现这种情况应该是很久没有维护了，所以也应该换一个了。



### 问题 9：Failed to resolve entry (ops，same proplem)

出现问题原因与`问题2`相同，也是因为没有`.vue`扩展名的问题。

![image-20210906185147837](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffxhfzmj60we06e40s02.jpg)

`@sxf/cloudsec-component`是一个自己维护的业务组件库，基于基础的 UI 组件库，将一些常用的业务进行封装，单独发布一个 npm 包。主要是为了方便同一个开发团队维护多个产品（这点与公司组织架构有一定的关系，不在深究）。因为业务组件需要调试的原因，npm 发布的是`.vue`的源码。

同样因为保持着相同的编码规范，业务中对组件库的引用，以及组件库本身开发时的互相引用都是省略了`.vue`扩展的。

而以上两种方案都无法通过`resolve.extensions`解决。

> 在 vite 2.4.4 的版本中，配置了 resolve.extensions 之后，无法识别 vue-router 中的 () => import('foo/bar/baz')
>
> 这个问题在 vite 2.5.3 中进行了修复，如果出现了此类问题，可以升级一下 vite 版本。

#### 解决方案一：人肉修改

人肉一个个修改，这种情况适用于刚开始不久的项目。但是对于一个维护数年的老项目来说，这基本不现实。20W 行左右的代码，几千个 vue 文件。作为程序员明显不能这么干活。

#### 解决方案二：vite 插件

vite 插件的第一个例子就是讲的引入一个虚拟文件，这里的问题其实类似。就是没有补充扩展名，导致找不到，那么可以通过插件查找。

> https://vitejs.bootcss.com/guide/api-plugin.html#importing-a-virtual-file

![image-20210906191108879](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffxsxjdj60kf0efwfe02.jpg)

这里提供一下伪代码，**未进行实际的验证**。

因为这只是解决了 vite 加载的问题，但是没有解决类型提示的问题，这对于后续进行 ts 相关改造不友好，所以没有去验证此方案。

感兴趣的可以验证一下，个人在实际的业务中是采用的方案三。

```js
module.exports = function () {
    return {
        name: 'vite-plugin-fix-vue-ext',
        resolveId(id) {
            // 判断是否需要补充后缀，如果需要则返回id，表示命中。
            if (isNeedFixExt(id)) return id;
        },
        load(id) {
            if (isNeedFixExt(id)) return fs.readFileSync(fixExt(id));
        }
    }
}
```

#### 解决方案三：codemod

codemod 指的是 `code modify`，实际上指的就是修改代码本身，并且是通过代码修改代码，而不是人肉修改代码。

实际上 `ESLint` 的 auto fix 也是 codemod 的一种。

这里的思路是：

1. 找到所有的 import 语句  和 动态 import 语句
2. 提取 import 的值，过滤掉已经有扩展名的值。
3. 对没有扩展名的值，处理成绝对路径（考虑相对路径，vite 别名），暂时称这个绝对路径为`ap`。
4. 然后对`ap`补齐`['.js','.ts','.vue','/index.js','/index.ts','/index.vue']`依次判断是否存在。如果`.vue`类型的存在，而其余类型的不存在则表示需要补齐后缀。
5. 判断是否存在的方案可以是依次尝试读取对应文件，或者是全部**预先读取建立 Map 以供查找。**

原理大致就是这样，关键在于怎么获取所有 import，又如何修改对应的文件。

* 方案一：利用 `es-module-lexer` 和 `magic-string`。因为 vite 本身就是用的这两个包进行的裸模块重写，关于什么是裸模块重洗，可以看下掘金这边文章的，这里不再赘述。这个方案的好处是处理的效率非常的高，坏处就是只能针对 import 进行操作，场景比较单一。

  > https://juejin.cn/post/6992200385561624607

* 方案二：利用 `jscodeshift`，相对而言，`jscodeshift`是操作 AST，比方案一要慢，且要复杂很多。因为`jscodeshift`的通用性比较高，担心后续迁移过程中有其他的 codemod 的需求，所以还是选择了方案二。

  > `jscodeshift`的 API 很复杂，官方的 github 上面也有很多 codemod 的示例。我们如果要开发则可以先在https://astexplorer.net/ 进行尝试。

首先，点开 Transform，并点击 jscodeshift。

![image-20210906194050473](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffyrg2vj61ku0l9gqt02.jpg)

然后就可以边写代码边看结果。

![image-20210906194623073](https://tva1.sinaimg.cn/large/008i3skNgy1gu8ffzxxsxj61e50l9agu02.jpg)

提取动态 import 的代码和正常 import 的代码如下

```js
// 提取动态import
j(file.source)
    .find(j.ImportDeclaration)
    .forEach(path => {
        const importNode = j(path);
    	const importString = importNode.get('source');
    	importString.value.value = 'zzzzzzzz';
    })
    .toSource();

// 提取普通import
j(file.source)
    .find(j.ImportExpression) // 只修改了这里 ImportDeclaration => ImportExpression
    .forEach(path => {
        const importNode = j(path);
    	const importString = importNode.get('source');
    	importString.value.value = 'zzzzzzzz';
    })
    .toSource();
```

尝试判断文件是否存在的代码如下

```js
// str 为import的字符串
// filename 为出现这个import 语句的绝对路径
const fixExtname = (str, fileName) => {
    let importAbsolutePath;
    // 处理 相对路径，别名等情况
    if (str.startsWith('./')) {
        importAbsolutePath = path.resolve(path.dirname(fileName), str);
    } else if (str.startsWith('../')) {
        importAbsolutePath = path.resolve(path.dirname(fileName), str);
    } else  {
        importAbsolutePath = fixAlias(str);
    }
    const tries = {
        '.js': '',
        '.ts': '',
        '.vue': '.vue',
        '/index.js': '',
        '/index.ts': '',
        '/index.vue': '/index.vue',
    }
    let ext = '';
    Object.keys(tries).forEach(key => {
        const p = importAbsolutePath + key;
        if (fs.existsSync(p)) {
            ext = tries[key];
        } else {
        }
    })
    return ext;
}
```

#### 关于方案三

这里还有个问题就是 jscodeshit 无法支持.vue 文件，个人开发是基于公司的一个 codemod 工具，那里封装了对 vue 文件的识别。

识别 vue 文件也是通过 vue 官方的包解析就行了。如果大家有这方面强烈的需求，可以考虑把这些代码封装成一个单独的 jscodeshit 的 codemod。

### 问题 10： does not provide an export named 'default'

![image-20210906203935807](https://tva1.sinaimg.cn/large/008i3skNgy1gu8fg0ocahj60rw04m0th02.jpg)

这个问题是因为第三方的 npm 包是提供的是 CommonJS 规范的文件。

> https://vitejs.bootcss.com/guide/dep-pre-bundling.html#the-why
>
> **CommonJS 和 UMD 兼容性:** 开发阶段中，vite 的开发服务器将所有代码视为原生 ES 模块。因此，vite 必须先将作为 CommonJS 或 UMD 发布的依赖项转换为 ESM。

`vite` 官网说，会兼容`CommonJS` 格式的文件，但是这里为什么还是因为`CommonJS`格式的问题呢？

其实这个文件是可以 import 导入的只是并没有提供 default 这个属性，这么说 vite 也确实支持了`CommonJS`。

遇到这种情况我们需要引入 `@originjs/vite-plugin-CommonJS`这个包

```js
const { viteCommonJS } = require('@originjs/vite-plugin-CommonJS');
module.exports = {
    plugins: [
        viteCommonJS({
            include: ['procurios.resizesensor']
        }),
        createVuePlugin()
    ]
}
```



## 采坑阶段二总结

在 主入口页面的迁移过程中，改动了非常多的代码。

* 业务代码`293`个文件
* 业务组件的代码`139`个文件

但是修改这些文件大多数没有修改业务，只是补全了链接。但是 less 的文件 import 却是和 `webpack` 不兼容。

此外，还有有一些有问题的 npm 包，这次迁移的过程并没有完全解决,只是将对应的代码给注释。

最终通过以上措施，终于能够访问了，也看到了主入口的页面。

![image-20210906205346596](https://tva1.sinaimg.cn/large/008i3skNgy1gu8fg1374qj60s30mt76t02.jpg)

## 最终总结

`vite` 优点大家都知道，我主要来说一下现阶段的缺点。

总的来说现阶段的`vite`还存在以下问题：

* 手动重新`npm run dev`后，预构建之后缓存丢失，而修改`vite.config.js`导致的自动重启不会。
* 预构建需要访问到对应的文件才能触发，会导致频繁`408`，且速度非常慢。整个下来时间比较久，而且需要你手动刷新浏览器。比较奇怪的是`vite 2.4.4` 在这方面 比`vite 2.5.3` 快非常多，基本上只会出现一次`408`。
* 无法很好的处理非 `ES module`的 npm 包，与`webpack`还存在一定的差距。
* 请求数量巨大，从 network 的面板来看，请求的数量高达`468`，除去`XHR`请求之外也有将近`400`个请求。所以首次加载的时间比较慢大概花了`5秒`，这一块应该是可以通过 vite 的配置来优化。等内部的一些 npm 包调整完毕能够进行正常的开发之后，会总结一些开发的实践并发出来。
* 功能不稳定。在早期使用`vite 2.4.4`进行迁移时，问题稍微还多一些。之后更新到`vite 2.5.3`之后，部分问题被修复了，但是也增加了一些新的问题。

针对老项目需要迁移到 vite，需要考虑以下情况：

* 不在支持非`ES module`的业务代码（如 require，module.exports），如果有这些代码需要进行调整，可以尝试与`问题9`相同的方式。
* less/sass 的`@import`:  vite 不在需要 ~ 来使用别名，这一点与 webpack 不兼容。
* 因为 IE11 天然不支持`ES module`，所以 IE 调试比较麻烦，不建议需要支持 IE11 的项目将`vite`作为唯一的构建工具。
* 现阶段的`vite` 对 npm 包有着较高的要求，迁移之前可以先看下项目的 npm 包，该替换的替换，该升级的升级。

整体来说，`vite`的开发体验比较好，轻量且快。即便是一个臃肿的老项目也能够做到反应迅速且不占用过多资源。项目较大时`webpack`启动时会占用大量的电脑资源，导致这一段时间 VSCode 非常卡，这一点体验非常不好。

总的来说`vite` 未来可期。

最后祝大家搬砖开心:smile: