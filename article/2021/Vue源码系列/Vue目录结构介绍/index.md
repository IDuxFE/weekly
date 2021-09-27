### Vue2.x 源码学习系列

#### 前言

当我们入手准备去查看一个项目的源代码时，可以从以下几个方面进行入手：

- 查看 `package.json`，根据 `main` 或者`module` 字段能够看出源代码的入口文件
- 查看 `package.json` 的 `scripts` 字段查看项目的打包脚本，通过打包脚本能够根据构建配置找到构建的源文件主入口

介绍完后，现在开始对 `Vue2.x` 版本的源码进行分析。首先对整体的代码目录结构进行了解。

#### 目录结构介绍

```markdown
├─benchmarks          // 性能测试相关
├─dist                // 打包后的文件集合
├─flow                // flow类型声明
├─packages            // 与`vue`相关的一些其他的npm包, `vue-server-render`, `vue-template-compiler`等
├─scripts             // 构建相关的脚本
├─src                 // 源代码入口
|  ├─shared           // 项目中用到的一些公共变量,方法等
|  ├─sfc              // 用于处理单文件组件(.vue)解析的逻辑
|  ├─server           // 服务端渲染相关的代码
|  ├─platforms        // 不同平台之间的代码
|  ├─core             // Vue的核心**运行时**代码
|  |  ├─vdom          // 虚拟dom相关的代码
|  |  ├─util          // Vue里用到的一些工具方法抽取
|  |  ├─observer      // 实现响应式原理的代码
|  |  ├─instance      // vue实例相关的核心逻辑
|  |  ├─global-api    // 全局api Vue.extend, Vue.component等
|  |  ├─components    // 内置的全局组件
|  ├─compiler         // 与模板编译相关的代码
├─types               // Typescript类型声明
├─test                // 测试相关的代码
```

### Vue 的不同版本的介绍

我们按照上述查看 `scripts` 字段后，执行 Vue 仓库里的 `build` 命令，能够发现 `dist`打出了各种各样后缀的包，那么这些包之间有什么区别呢？以下表格摘自 `Vue` 官网对不同构建版本的介绍

|                               | UMD                | CommonJS              | ES Module          | **ES Module (直接用于浏览器)** |
| ----------------------------- | ------------------ | --------------------- | ------------------ | ------------------------------ |
| **完整版**                    | vue.js             | vue.common.js         | vue.esm.js         | vue.esm.browser.js             |
| **只包含运行时版**            | vue.runtime.js     | vue.runtime.common.js | vue.runtime.esm.js | -                              |
| **完整版 (生产环境)**         | vue.min.js         | -                     | -                  | vue.esm.browser.min.js         |
| **只包含运行时版 (生产环境)** | vue.runtime.min.js | -                     | -                  | -                              |

#### 术语

- **完整版**: 同时包含编译器和运行时的版本。
- **编译器**：主要用于将模板字符串编译成 Javascript 渲染函数。
- **运行时**： 负责创建 vue 实例、渲染并处理虚拟 DOM 等的代码。基本上就是除去编译器的其它一切。
- **[UMD](https://github.com/umdjs/umd)**：UMD 版本可以通过标签直接用在浏览器中。jsDelivr CDN 的 https://cdn.jsdelivr.net/npm/vue 默认文件就是运行时 + 编译器的 UMD 版本 (`vue.js`)。
- **[CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1)**：CommonJS 版本用来配合老的打包工具比如 [Browserify](http://browserify.org/) 或 [webpack 1](https://webpack.github.io/)。这些打包工具的默认文件 (`pkg.main`) 是只包含运行时的 CommonJS 版本 (`vue.runtime.common.js`)。
- **[ES Module](http://exploringjs.com/es6/ch_modules.html)**：从 2.6 开始 Vue 会提供两个 ES Modules (ESM) 构建文件：

#### 运行时 + 编译器 vs 只包含运行时

如果你需要在客户端编译模板 (比如传入一个字符串给 `template` 选项，或挂载到一个元素上并以其 DOM 内部的 HTML 作为模板)，就将需要加上编译器，即完整版：

```javascript
// 需要编译器
new Vue({
  template: '<div>{{ hi }}</div>'
})

// 不需要编译器
new Vue({
  render (h) {
    return h('div', this.hi)
  }
})
```

当使用 `vue-loader` 或 `vueify` 的时候， `*.vue`文件内部的模板会在构建时预编译成 Javascript。你在最终打好的包里实际上是不需要编译器的，所以只用运行时版本即可。也就是，当我们使用 `*.vue` 文件开发时，`*.vue` 打包出来的结果实际上是不存在 `template` 配置的， 而是已经转成了 `render` 函数。

因为运行时版本相比完整版体积要小大约 30%，所以应该尽可能使用这个版本。如果你仍然希望使用完整版，则需要在打包工具里配置一个别名：

#### webpack

```javascript
module.exports = {
  // ...
  resolve: {
    alias: {
      'vue$': 'vue/dist/vue.esm.js' // 用 webpack 1 时需用 'vue/dist/vue.common.js'
    }
  }
}
```

#### Rollup

```javascript
const alias = require('rollup-plugin-alias')

rollup({
  // ...
  plugins: [
    alias({
      'vue': require.resolve('vue/dist/vue.esm.js')
    })
  ]
})
```

#### Browserify

添加到你项目的 `package.json`：

```javascript
{
  // ...
  "browser": {
    "vue": "vue/dist/vue.common.js"
  }
}
```

#### 开发环境 vs 生成环境

对于 UMD 版本来说，开发环境/生产环境模式是硬编码好的：开发环境下用未压缩的代码，生产环境下使用压缩后的代码。

CommonJS 和 ES Module 版本是用于打包工具的，因此我们不提供压缩后的版本。你需要自行将最终的包进行压缩。

CommonJS 和 ES Module 版本同时保留原始的 `process.env.NODE_ENV` 检测，以决定它们应该运行在什么模式下。你应该使用适当的打包工具配置来替换这些环境变量以便控制 Vue 所运行的模式。把 `process.env.NODE_ENV` 替换为字符串字面量同时可以让 UglifyJS 之类的压缩工具完全丢掉仅供开发环境的代码块，以减少最终的文件尺寸。

#### webpack

在 webpack 4+ 中，你可以使用 `mode` 选项：

```javascript
module.exports = {
  mode: 'production'
}
```

但是在 webpack 3 及其更低版本中，你需要使用 [DefinePlugin](https://webpack.js.org/plugins/define-plugin/)：

```javascript
var webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: JSON.stringify('production')
      }
    })
  ]
}
```

#### Rollup

使用 [rollup-plugin-replace](https://github.com/rollup/rollup-plugin-replace)：

```javascript
const replace = require('rollup-plugin-replace')

rollup({
  // ...
  plugins: [
    replace({
      'process.env.NODE_ENV': JSON.stringify('production')
    })
  ]
}).then(...)
```

以上内容摘自 `vue官网` 对不同构建版本的解释。

### Vue 的打包脚本逻辑

上个章节我们介绍了 `Vue` 的目录结构， 现在我们顺着 `Vue` 源码继续进行分析。

在抛出了各种构建版本的概念后， 我们看看不同版本，环境的文件是怎么构建出来的。

在项目里的 `package.json` 中，关于打包的脚本主入口命令为：

```javascript
“scripts”： {
    // ...
	"build": "node scripts/build.js",
}
```

也就是说， 当执行 `npm run build` 命令时， 主要是执行了 `node scripts/build.js`， 那么看看 `scripts/build.js` 主要做了什么操作。

```javascript
// 导入各种需要用到的模块
const fs = require('fs')
const path = require('path')
const zlib = require('zlib')
const rollup = require('rollup')
const terser = require('terser')

// 没有 `dist` 目录，则创建
if (!fs.existsSync('dist')) {
  fs.mkdirSync('dist')
}

// `getAllBuilds` 获取所有的打包文件配置， 在 `scripts/config` 文件下， 主要定义了不同版本的构建配置，
// 其中几个主要的字段：定义打包入口 `entry`， 文件输出地址 `dest`， 生成包的格式 `format`等。
let builds = require('./config').getAllBuilds()

// filter builds via command line arg
// 查看是否有传入额外的参数， 如 `node scripts/build.js -- web-runtime-cjs，web-server-renderer`， 
// 则代表仅打包web-runtime-cjs, web-server-renderer对应的构建配置
if (process.argv[2]) {
  const filters = process.argv[2].split(',')
  builds = builds.filter(b => {
    return filters.some(f => b.output.file.indexOf(f) > -1 || b._name.indexOf(f) > -1)
  })
} else {
  // filter out weex builds by default
  builds = builds.filter(b => {
    return b.output.file.indexOf('weex') === -1
  })
}

// 打包主入口
build(builds)

function build (builds) {
  let built = 0 // 利用一个索引指向当前需要构建哪个包
  const total = builds.length
  
  // 定义一个迭代函数, 每个next函数代表一个构建包的执行
  // 将所有构建包组成构建队列, 逐个依次进行构建
  const next = () => {
    buildEntry(builds[built]).then(() => {
      built++
      if (built < total) {
        next()
      }
    }).catch(logError) // promise的错误捕获
  }
  next()
}

function buildEntry (config) { // 打包单个构建版本
  const output = config.output
  const { file, banner } = output
  const isProd = /(min|prod)\.js$/.test(file)
  
  // 这里会返回pomise, 所以利用这个进行链式迭代生成不同的构建版本
  return rollup.rollup(config)
    .then(bundle => bundle.generate(output))
    .then(({ output: [{ code }] }) => {
      if (isProd) {
        const minified = (banner ? banner + '\n' : '') + terser.minify(code, {
          toplevel: true,
          output: {
            ascii_only: true
          },
          compress: {
            pure_funcs: ['makeMap']
          }
        }).code
        return write(file, minified, true)
      } else {
        return write(file, code)
      }
    })
}
```

```javascript
// scripts/config.js
const builds = {
    'web-runtime-cjs-dev': {
        entry: resolve('web/entry-runtime.js'),
        dest: resolve('dist/vue.runtime.common.dev.js'),
        format: 'cjs',
        env: 'development',
        banner
  	},
    'web-full-dev': {
        entry: resolve('web/entry-runtime-with-compiler.js'),
        dest: resolve('dist/vue.js'),
        format: 'umd',
        env: 'development',
        alias: { he: './entity-decoder' },
        banner
  },
};
exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
```

通过上面的打包脚本代码，我们可以总结 `Vue` 在进行打包的时候，主要做了以下几步：

- 判断有没有 `dist` 目录，没有则创建
- 获取所有构建版本的 `rollup` 打包配置
- 查看命令是否有传参，如果有传参，则根据参数过滤出指定需要构建的包
- 执行打包主函数 `build` ，内部创建了一个迭代函数 `next`， 并且利用一个指针 `built`，让打包通过队列的形式逐个进行构建

看完上面的打包脚本，接下来分析下不同的打包脚本入口之间有什么区别。

以 `web运行时版本` 和 `web全量包` 作为对比来看：

```javascript
// src/platforms/web/runtime/index.js
import Vue from 'core/index'

// public mount method
// 定义Vue的核心挂载函数$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
export default Vue
```

```javascript
// src/platforms/web/entry-runtime-with-compiler.js
import Vue from './runtime/index'

// 基于web运行时的Vue构造器, 进行多一层封装
// 通过切片编程, 扩展$mount挂载函数使其拥有对传入的template进行模板解析的能力
const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  const options = this.$options
  // resolve template/el and convert to render function
  if (!options.render) {
    let template = options.template
    const { render, staticRenderFns } = compileToFunctions(template, {
        outputSourceRange: process.env.NODE_ENV !== 'production',
        shouldDecodeNewlines,
        shouldDecodeNewlinesForHref,
        delimiters: options.delimiters,
        comments: options.comments
    }, this)
    options.render = render
    options.staticRenderFns = staticRenderFns
  }
  return mount.call(this, el, hydrating)
}
```

通过上面的配置文件， 我们可以根据入口文件路径，看出每个入口其实最终还是会引入 `src/core/index`里面导出的 `Vue` 构造函数， 然后基于这个构造函数， 扩展额外的能力， 比如需要带有 `compiler` 的构建版本， 则利用切片编程的技术，对 `Vue.prototype.$mount`进行扩展， 使 `vue`实例在进行挂载前对传入的`template`具有编译能力转成`render` 函数再进行挂载，由`template`转成`render`的过程再后续章节会进行介绍，敬请关注该系列文章。

### 总结

本文我们主要从 `Vue` 仓库代码目录对对整体的目录结构进行了了解，通过目录结构，能够知道不同目录下的代码作用，并且了解了不同构建版本之间的区别。

接着，我们从打包文件入口开始，了解了`Vue`的构建过程，从入口文件查看后能够知道`Vue`核心代码主要是 `src/core/index.js`导出的 `Vue` 构造函数，并且不同的构建版本对该构造函数进行了不同的功能扩展，如需要编译器的版本则会对`Vue.prototype.$mount`扩展对`options.template`的解析能力。

下一篇我们主要对`Vue`的核心代码文件 `src/core/index.js` 进行学习，并且学习 Vue2.x 版本的数据响应式原理。敬请关注。



