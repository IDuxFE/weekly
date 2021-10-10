>  假期最后一天，我们来卷一下 `SFC` 中 `customBlocks` 的使用及其工作原理。

![](https://files.mdnice.com/user/15734/69af9ee0-394f-410d-a8f7-1cc018e89dd9.png)

本文大纲：

- 通过 `vue-i18n` 的  `<i18n>` 了解 `customBlocks` 和基本配置；

- 从源码层面了解 `vue-loader` 对 `customBlocks`  的处理

### vue-i18n

[vue-i18n](https://kazupon.github.io/vue-i18n/ "vue-i18n") 是 `Vue` 的国际化插件。如果使用 `SFC` 的方式写组件的话，可以在 `.vue` 文件中定义  `<i18n> ` 块 ，然后在块内写入对应的词条。这个 `i18n` 标签就是 `customBlocks`。举个例子：

```vue

<template>
  <p>{{ $t('hello') }}</p>
</template>

<script>
// App.vue
export default {
  name: 'App'
}
</script>

<i18n locale="en">
{
  "hello": "hello, world!!!!"
}
</i18n>

<i18n locale="ja">
{
  "hello": "こんにちは、世界！"
}
</i18n>
```

```js
// main.js
import Vue from 'vue'
import VueI18n from 'vue-i18n'
import App from './App.vue'

Vue.use(VueI18n)

const i18n = new VueI18n({
  locale: 'ja',
  messages: {}
})

new Vue({
  i18n,
  el: '#app',
  render: h => h(App)
})
```

上述代码定义了日文和英文两种语法，只要改变 `locale` 的值，就能达到切换语言的效果。除了上述用法，还支持支持引入 `yaml` 或者 `json` 等文件：

```vue
<i18n src="./locales.json"></i18n>
```

```json
// locales.json
{
  "en": {
    "hello": "hello world"
  },
  "ja": {
    "hello": "こんにちは、世界"
  }
}
```

`<i18n>` 其他用法可以查阅[<i18n>使用文档](https://kazupon.github.io/vue-i18n/guide/sfc.html#basic-usage "<i18n>使用文档")；

要让 `customBlock` 起作用，需要指定 `customBlock` 的 `loader`，如果没有指定，对应的块会默默被忽略。🌰 中的 `webpack` 配置：

```js
const path = require('path')
const VueLoaderPlugin = require('vue-loader/lib/plugin')

module.exports = {
  mode: 'development',
  entry: path.resolve(__dirname, './main.js'),
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js',
    publicPath: '/dist/'
  },
  devServer: {
    stats: 'minimal',
    contentBase: __dirname
  },
  module: {
    rules: [
      {
        test: /\.vue$/,
        loader: 'vue-loader'
      },
      {
        test: /\.js$/,
        loader: 'babel-loader'
      },
      //  customBlocks 对应的 rule
      {
        // 使用 resourceQuery 来为一个没有 lang 的自定义块匹配一条规则
        // 如果找到了一个自定义块的匹配规则，它将会被处理，否则该自定义块会被默默忽略
        resourceQuery: /blockType=i18n/,
        // Rule.type 设置类型用于匹配模块。它防止了 defaultRules 和它们的默认导入行为发生
        type: 'javascript/auto',
        // 这里指的是 vue-i18n-loader
        use: [path.resolve(__dirname, '../lib/index.js')]
      }
    ]
  },
  plugins: [new VueLoaderPlugin()]
}
```

从上述代码可以看到，如果你要在 `SFC` 中使用 `customBlock` 功能，只需要下面两步：

1. 实现一个处理 `customBlock` 的 `loader` 函数；
2. 配置 `webpack.module.rules` ，指定 `resourceQuery: /blockType=你的块名称/` 然后使用步骤一的 `loader` 去处理即可；

### 源码分析

通常一个 `loader` 都是具体某一种资源的转换、加载器，但 `vue-loader` 不是，它能够处理每一个定义在 `SFC` 中的块：通过**拆解 block** -> **组合 loader** -> **处理 block** -> **组合每一个 block 的结果为最终代码**的工作流，完成对 `SFC` 的处理。下面我们就依次详细地拆解这条流水线！

#### 拆解 block

我们知道，使用 `vue-loader` 一定需要引入 `vue-loader-plugin`，不然的话就会给你报一个大大的错误：

```js
`vue-loader was used without the corresponding plugin. Make sure to include VueLoaderPlugin in your webpack config.`
```

`VueLoaderPlugin`  定义在 `vue-loader\lib\plugin-webpack4.js`：

```js
const id = 'vue-loader-plugin'
const NS = 'vue-loader'

class VueLoaderPlugin {
  apply (compiler) {
    // add NS marker so that the loader can detect and report missing plugin
    if (compiler.hooks) {
      // webpack 4
      compiler.hooks.compilation.tap(id, compilation => {
        const normalModuleLoader = compilation.hooks.normalModuleLoader // 同步钩子，管理所有模块loader
        normalModuleLoader.tap(id, loaderContext => {
          loaderContext[NS] = true
        })
      })
    }

    // use webpack's RuleSet utility to normalize user rules
    const rawRules = compiler.options.module.rules
    // https://webpack.js.org/configuration/module/#modulerules
    const { rules } = new RuleSet(rawRules)
    
    // 将你定义过的 loader 复制并应用到 .vue 文件里相应语言的块
    const clonedRules = rules
      .filter(r => r !== vueRule)
      .map(cloneRule)

   	// ...
    // 个人对这个命名的理解是 pitcher 是投手的意思，进球得分，所以可以理解成给当前的块和 loader 丰富功能 😁
    // 给 template 块加 template-loader，给 style 块加 stype-post-loader
    // 其他功能...后面再看
    const pitcher = {
      loader: require.resolve('./loaders/pitcher'),
      resourceQuery: query => {
        const parsed = qs.parse(query.slice(1))
        return parsed.vue != null
      },
      options: {
        cacheDirectory: vueLoaderUse.options.cacheDirectory,
        cacheIdentifier: vueLoaderUse.options.cacheIdentifier
      }
    }

    // 覆盖原来的rules配置
    compiler.options.module.rules = [
      pitcher,
      ...clonedRules,
      ...rules
    ]
  }
}
```

`VueLoaderPlugin` 作用是将你定义的其他 `loader` 添加到 `SFC` 的各个块中并修改配置中的 `module.rules`。[pitcher-loader](https://webpack.docschina.org/api/loaders/#pitching-loader "pitcher-loader") 是后续一个重要的角色。阿宝哥的[多图详解，一次性搞懂Webpack Loader](https://juejin.cn/post/6992754161221632030#heading-3 "多图详解，一次性搞懂Webpack Loader")有详细的分享，没了解过滴童鞋可以先去认识一下这个“投手”的作用。

了解完 `VueLoaderPlugin`，我们看到 `vue-loader`：

```js
module.exports = function (source) {
  const loaderContext = this

  // ...
  // 编译 SFC —— 解析.vue文件，生成不同的 block
  const descriptor = parse({
    source,
    compiler: options.compiler || loadTemplateCompiler(loaderContext),  // 默认使用 vue-template-compiler
    filename,
    sourceRoot,
    needMap: sourceMap
  })
  
  // ...
}
```

本小节核心就是这个 `parse` 方法。将 `SFC` 代码传通过自定义编译器或者默认的 `@vue/component-compiler-utils` 去解析。具体执行过程这里就不展开详细分析了，感兴趣童鞋可以前往[[咖聊] “模板编译”真经](https://mp.weixin.qq.com/s/egEkIOHDWt7bbgUKdXdY2w)。生成的 `descriptor` 结果如下图所示：
  
![](https://files.mdnice.com/user/15734/0de88e84-9a9c-4881-87ac-25b587b59bea.png)

接下来就针对 `descriptor` 的每一个 `key` 去生成第一次代码：

```js
module.exports = function (source) {
  const loaderContext = this

  // ...
  // 编译 SFC —— 解析.vue文件，生成不同的 block
  const descriptor = parse({
    source,
    compiler: options.compiler || loadTemplateCompiler(loaderContext),  // 默认使用 vue-template-compiler
    filename,
    sourceRoot,
    needMap: sourceMap
  })
  
  // ...
  // template
  let templateImport = `var render, staticRenderFns`
  let templateRequest
  if (descriptor.template) {
    const src = descriptor.template.src || resourcePath
    const idQuery = `&id=${id}`
    const scopedQuery = hasScoped ? `&scoped=true` : ``
    const attrsQuery = attrsToQuery(descriptor.template.attrs)
    const query = `?vue&type=template${idQuery}${scopedQuery}${attrsQuery}${inheritQuery}`
    const request = templateRequest = stringifyRequest(src + query)
    templateImport = `import { render, staticRenderFns } from ${request}`
  }

  // script
  let scriptImport = `var script = {}`
  if (descriptor.script) {
    const src = descriptor.script.src || resourcePath
    const attrsQuery = attrsToQuery(descriptor.script.attrs, 'js')
    const query = `?vue&type=script${attrsQuery}${inheritQuery}`
    const request = stringifyRequest(src + query)
    scriptImport = (
      `import script from ${request}\n` +
      `export * from ${request}` // support named exports
    )
  }

  // styles
  let stylesCode = ``
  if (descriptor.styles.length) {
    stylesCode = genStylesCode(
      loaderContext,
      descriptor.styles,
      id,
      resourcePath,
      stringifyRequest,
      needsHotReload,
      isServer || isShadow // needs explicit injection?
    )
  }

  let code = `
${templateImport}
${scriptImport}
${stylesCode}

/* normalize component */
import normalizer from ${stringifyRequest(`!${componentNormalizerPath}`)}
var component = normalizer(
  script,
  render,
  staticRenderFns,
  ${hasFunctional ? `true` : `false`},
  ${/injectStyles/.test(stylesCode) ? `injectStyles` : `null`},
  ${hasScoped ? JSON.stringify(id) : `null`},
  ${isServer ? JSON.stringify(hash(request)) : `null`}
  ${isShadow ? `,true` : ``}
)
  `.trim() + `\n`
  
  // 判断是否有customBlocks，调用genCustomBlocksCode生成自定义块的代码
  if (descriptor.customBlocks && descriptor.customBlocks.length) {
    code += genCustomBlocksCode(
      descriptor.customBlocks,
      resourcePath,
      resourceQuery,
      stringifyRequest
    )
  }
  // ...省略一些热更代码
  
  return code
}

// vue-loader\lib\codegen\customBlocks.js
module.exports = function genCustomBlocksCode (
  blocks,
  resourcePath,
  resourceQuery,
  stringifyRequest
) {
  return `\n/* custom blocks */\n` + blocks.map((block, i) => {
    // i18n有很多种用法，有通过src直接引入其他资源的用法，这里就是获取这个参数
    // 对于demo而言，没有定义外部资源，这里是''
    const src = block.attrs.src || resourcePath
    // 获取其他属性，demo中就是&locale=en和&locale=ja
    const attrsQuery = attrsToQuery(block.attrs)
    // demo中是''
    const issuerQuery = block.attrs.src ? `&issuerPath=${qs.escape(resourcePath)}` : ''
    // demo中是''
    const inheritQuery = resourceQuery ? `&${resourceQuery.slice(1)}` : ''
    const query = `?vue&type=custom&index=${i}&blockType=${qs.escape(block.type)}${issuerQuery}${attrsQuery}${inheritQuery}`
    return (
      `import block${i} from ${stringifyRequest(src + query)}\n` +
      `if (typeof block${i} === 'function') block${i}(component)`
    )
  }).join(`\n`) + `\n`
}
```

`template`、`style`、`script` 这些块我们直接略过，重点看看 `customBlocks` 的处理逻辑。逻辑比较简单，遍历 `customBlocks` 去获取一些 `query` 变量，最终返回 `customBlocks  code`。我们看看最终通过第一次调用 `vue-loader` 返回的 `code`：

```js
/* template块 */
import { render, staticRenderFns } from "./App.vue?vue&type=template&id=a9794c84&"
/* script 块 */
import script from "./App.vue?vue&type=script&lang=js&"
export * from "./App.vue?vue&type=script&lang=js&"


/* normalize component */
import normalizer from "!../node_modules/vue-loader/lib/runtime/componentNormalizer.js"
var component = normalizer(
  script,
  render,
  staticRenderFns,
  false,
  null,
  null,
  null
  
)

/* 自定义块，例子中即 <i18n> 块的代码 */
import block0 from "./App.vue?vue&type=custom&index=0&blockType=i18n&locale=en"
if (typeof block0 === 'function') block0(component)
import block1 from "./App.vue?vue&type=custom&index=1&blockType=i18n&locale=ja"
if (typeof block1 === 'function') block1(component)

/* hot reload */
if (module.hot) {
  var api = require("C:\\Jouryjc\\vue-i18n-loader\\node_modules\\vue-hot-reload-api\\dist\\index.js")
  api.install(require('vue'))
  if (api.compatible) {
    module.hot.accept()
    if (!api.isRecorded('a9794c84')) {
      api.createRecord('a9794c84', component.options)
    } else {
      api.reload('a9794c84', component.options)
    }
    module.hot.accept("./App.vue?vue&type=template&id=a9794c84&", function () {
      api.rerender('a9794c84', {
        render: render,
        staticRenderFns: staticRenderFns
      })
    })
  }
}
component.options.__file = "example/App.vue"
export default component.exports
```

紧接着继续处理 `import`：

```js
/* template块 */
import { render, staticRenderFns } from "./App.vue?vue&type=template&id=a9794c84&"
/* script 块 */
import script from "./App.vue?vue&type=script&lang=js&"

/* 自定义块，例子中即 <i18n> 块的代码 */
import block0 from "./App.vue?vue&type=custom&index=0&blockType=i18n&locale=en"
import block1 from "./App.vue?vue&type=custom&index=1&blockType=i18n&locale=ja"
```

#### 组合 loader

我们可以看到，上述所有资源都有 `?vue` 的 `query` 参数，匹配到了 `pitcher-loader` ，该“投手”登场了。分析下 `import block0 from "./App.vue?vue&type=custom&index=0&blockType=i18n&locale=en"` 处理：

```js
module.exports.pitch = function (remainingRequest) {
  const options = loaderUtils.getOptions(this)
  const { cacheDirectory, cacheIdentifier } = options
  const query = qs.parse(this.resourceQuery.slice(1))

  let loaders = this.loaders

  // if this is a language block request, eslint-loader may get matched
  // multiple times
  if (query.type) {
    // 剔除eslint-loader
    if (/\.vue$/.test(this.resourcePath)) {
      loaders = loaders.filter(l => !isESLintLoader(l))
    } else {
      // This is a src import. Just make sure there's not more than 1 instance
      // of eslint present.
      loaders = dedupeESLintLoader(loaders)
    }
  }

  // 提取pitcher-loader
  loaders = loaders.filter(isPitcher)

  // do not inject if user uses null-loader to void the type (#1239)
  if (loaders.some(isNullLoader)) {
    return
  }

  const genRequest = loaders => {
    // Important: dedupe since both the original rule
    // and the cloned rule would match a source import request.
    // also make sure to dedupe based on loader path.
    // assumes you'd probably never want to apply the same loader on the same
    // file twice.
    // Exception: in Vue CLI we do need two instances of postcss-loader
    // for user config and inline minification. So we need to dedupe baesd on
    // path AND query to be safe.
    const seen = new Map()
    const loaderStrings = []

    loaders.forEach(loader => {
      const identifier = typeof loader === 'string'
        ? loader
        : (loader.path + loader.query)
      const request = typeof loader === 'string' ? loader : loader.request
      if (!seen.has(identifier)) {
        seen.set(identifier, true)
        // loader.request contains both the resolved loader path and its options
        // query (e.g. ??ref-0)
        loaderStrings.push(request)
      }
    })

    return loaderUtils.stringifyRequest(this, '-!' + [
      ...loaderStrings,
      this.resourcePath + this.resourceQuery
    ].join('!'))
  }

  // script、template、style...

  // if a custom block has no other matching loader other than vue-loader itself
  // or cache-loader, we should ignore it
  // 如果除了vue-loader没有其他的loader，就直接忽略
  if (query.type === `custom` && shouldIgnoreCustomBlock(loaders)) {
    return ``
  }

  // When the user defines a rule that has only resourceQuery but no test,
  // both that rule and the cloned rule will match, resulting in duplicated
  // loaders. Therefore it is necessary to perform a dedupe here.
  const request = genRequest(loaders)
  return `import mod from ${request}; export default mod; export * from ${request}`
}
```

`pitcher-loader` 做了 3 件事：

- 剔除 `eslint-loader`，避免重复 `lint`；
- 剔除 `pitcher-loader` 自身；
- 根据不同的 `query.type`，生成对应的 `request`，并返回结果；

 🌰 中 `customBlocks` 返回的结果如下：

```js
// en
import mod from "-!../lib/index.js!../node_modules/vue-loader/lib/index.js??vue-loader-options!./App.vue?vue&type=custom&index=0&blockType=i18n&locale=en";
export default mod;
export * from "-!../lib/index.js!../node_modules/vue-loader/lib/index.js??vue-loader-options!./App.vue?vue&type=custom&index=0&blockType=i18n&locale=en"

// ja
import mod from "-!../lib/index.js!../node_modules/vue-loader/lib/index.js??vue-loader-options!./App.vue?vue&type=custom&index=1&blockType=i18n&locale=ja";
export default mod;
export * from "-!../lib/index.js!../node_modules/vue-loader/lib/index.js??vue-loader-options!./App.vue?vue&type=custom&index=1&blockType=i18n&locale=ja"
```

#### 处理 block

根据 `import` 的表达式，我们可以看到，此时会通过 `vue-loader` -> `vue-i18n-loader` 依次处理拿到结果，此时再进入到 `vue-loader` 跟前面第一次生成 `code` 不一样的地方是：此时 `incomingQuery.type` 是有值的。对于 `custom` 而言，这里就是 `custom`：

```js
// ...
// if the query has a type field, this is a language block request
// e.g. foo.vue?type=template&id=xxxxx
// and we will return early
if (incomingQuery.type) {
    return selectBlock(
        descriptor,
        loaderContext,
        incomingQuery,
        !!options.appendExtension
    )
}
// ...
```

会执行到 `selectBlock`：

```js
module.exports = function selectBlock (
  descriptor,
  loaderContext,
  query,
  appendExtension
) {
  // template
  // script
  // style

  // custom
  if (query.type === 'custom' && query.index != null) {
    const block = descriptor.customBlocks[query.index]
    loaderContext.callback(
      null,
      block.content,
      block.map
    )
    return
  }
}
```

最后会执行到 `vue-i18n-loader`：

```js
const loader: webpack.loader.Loader = function (
  source: string | Buffer,
  sourceMap: RawSourceMap | undefined
): void {
  if (this.version && Number(this.version) >= 2) {
    try {
      // 缓存结果，在输入和依赖没有发生改变时，直接使用缓存结果
      this.cacheable && this.cacheable()
      // 输出结果
      this.callback(
        null,
        `module.exports = ${generateCode(source, parse(this.resourceQuery))}`,
        sourceMap
      )
    } catch (err) {
      this.emitError(err.message)
      this.callback(err)
    }
  } else {
    const message = 'support webpack 2 later'
    this.emitError(message)
    this.callback(new Error(message))
  }
}

/**
 * 将i18n标签生成代码
 * @param {string | Buffer} source
 * @param {ParsedUrlQuery} query
 * @returns {string} code
 */
function generateCode(source: string | Buffer, query: ParsedUrlQuery): string {
  const data = convert(source, query.lang as string)
  let value = JSON.parse(data)

  if (query.locale && typeof query.locale === 'string') {
    value = Object.assign({}, { [query.locale]: value })
  }

  // 特殊字符转义，\u2028 -> 行分隔符，\u2029 -> 段落分隔符，\\ 反斜杠
  value = JSON.stringify(value)
    .replace(/\u2028/g, '\\u2028')
    .replace(/\u2029/g, '\\u2029')
    .replace(/\\/g, '\\\\')

  let code = ''
  code += `function (Component) {
  Component.options.__i18n = Component.options.__i18n || []
  Component.options.__i18n.push('${value.replace(/\u0027/g, '\\u0027')}')
  delete Component.options._Ctor
}\n`
  return code
}

/**
 * 转换各种用法为json字符串
 */
function convert(source: string | Buffer, lang: string): string {
  const value = Buffer.isBuffer(source) ? source.toString() : source

  switch (lang) {
    case 'yaml':
    case 'yml':
      const data = yaml.safeLoad(value)
      return JSON.stringify(data, undefined, '\t')
    case 'json5':
      return JSON.stringify(JSON5.parse(value))
    default:
      return value
  }
}

export default loader
```

上述代码就比较简单了，拿到 `source` 生成 `value`，最终 `push` 到 `Component.options.__i18n` 中，针对不同的情况有不同的处理方式（`json`、`yaml`等）。

至此，整个 `vue` 文件就构建结束了，`<i18n>` 最终构建完的代码如下：

```js
"./lib/index.js!./node_modules/vue-loader/lib/index.js?!./example/App.vue?vue&type=custom&index=0&blockType=i18n&locale=en":
(function (module, exports) {

    eval("module.exports = function (Component) {\n  Component.options.__i18n = Component.options.__i18n || []\n  Component.options.__i18n.push('{\"en\":{\"hello\":\"hello, world!!!!\"}}')\n  delete Component.options._Ctor\n}\n\n\n//# sourceURL=webpack:///./example/App.vue?./lib!./node_modules/vue-loader/lib??vue-loader-options");

})
```

至于 `vue-i18n` 怎么识别 `Component.options.__i18n` 就放一段代码，感兴趣可以去阅读 [vue-i18n](https://github.com/kazupon/vue-i18n "vue-i18n") 的代码哦。

```js
if (options.__i18n) {
    try {
        let localeMessages = options.i18n && options.i18n.messages ? options.i18n.messages : {};
        options.__i18n.forEach(resource => {
            localeMessages = merge(localeMessages, JSON.parse(resource));
        });
        Object.keys(localeMessages).forEach((locale) => {
            options.i18n.mergeLocaleMessage(locale, localeMessages[locale]);
        });
    } catch (e) {
        {
            error(`Cannot parse locale messages via custom blocks.`, e);
        }
    }
}
```

### 总结

本文从 `vue-i18n` 的工具切入，分享了如何在 `SFC` 中定义一个自定义块。然后从 `vue-loader` 源码分析了 `SFC` 的处理流程，整个过程如下图所示：
  
![](https://files.mdnice.com/user/15734/8a80dd71-4c0c-4675-aa58-39357107c0b4.png)

1. 从 `webpack` 构建开始，会调用到插件，`VueLoaderPlugin` 在 `normalModuleLoader` 钩子上会被执行；
2. 在引入 `SFC` 时，第一次匹配到 `vue-loader`，会通过 `@vue/component-compiler-utils` 将代码解析成不同的块，例如 `template`、`script`、`style`、`custom`；
3. 生成的 `code`，会继续匹配 `loader`，`?vue` 会匹配上“投手”`pitcher-loader`；
4. `pitcher-loader` 主要做 3 件事：首先因为 `vue` 整个文件已经被 `lint` 处理过了，所以局部代码时过滤掉 `eslint-loader`；其次过滤掉自身 `pitcher-loader`；最后通过 `query.type` 去生成不同的 `request` 和 `code`；
5. 最终 `code` 会再次匹配上 `vue-loader`，此时第二次执行，`incomingQuery.type` 都会指定对应的块，所以会根据 `type` 调用 `selectBlock` 生成最终的块代码。
