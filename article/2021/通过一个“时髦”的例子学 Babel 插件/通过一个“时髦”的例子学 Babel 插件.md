>  前文[Babel 那些事儿](https://juejin.cn/post/6992371845349507108)分享了 `Babel7.x` 最小最优配置和一些 `packages`，本文接着深入了解“她”（`Babel plugin`），让我们的感情更进一步。

## 工作流

![](https://files.mdnice.com/user/15734/a22a12bc-61c3-4574-b642-58a1e29e27a3.png)

Babel 的编译流程如上图所示，主要有三步：`parse`、`transform`、`generate`。`parse` 编译源代码，生成抽象语法树；`transform` 对 `AST` 树做各种操作（编译、删除、更新、新增等）；最后由 `generate` 将处理后的 `AST` 生成新的代码，并可以附带 `sourcemap`。

每一个过程中都有很多的工具包能够帮助你更好地完成任务：

- [@babel/parser](https://www.babeljs.cn/docs/babel-parser) 用来解析源代码；

- [@babel/traverse](https://www.babeljs.cn/docs/babel-traverse) 用于遍历和修改 `AST` 的节点；

- [@babel/types](https://www.babeljs.cn/docs/babel-types) 用于创建节点和判断节点的类型；

- [@babel/template](https://www.babeljs.cn/docs/babel-template) 用于快速地创建 `AST` 节点，比一个一个用 `@babel/types` 生成拼接的 `AST` 好用太多了；

- [@babel/generator](https://www.babeljs.cn/docs/babel-generator) 将 `AST` 转换成目标代码；

- [@babel/core](https://www.babeljs.cn/docs/babel-core) 是大哥大，涵盖了上述所有包的功能，可以完成从编译、转换到生成代码和 `sourcemap` 中的所有流程。

  

### AST Node Type

下图 Babel 中常见的抽象语法树节点的类型：
![](https://files.mdnice.com/user/15734/ba0079e0-c631-4c61-8617-e292d2e91e7a.png)

图中的 `Babel ast node type` 结合 [@babel/types](https://babeljs.io/docs/en/babel-types)，能够协助我们更好地完成节点的**判断、创建、查找**。（😏😏 可以收藏哦！）



## 从 Options 到 Composition

`Vue3.2` 支持 `script setup` 的真香写法，不了解的童鞋可以看下 [RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0040-script-setup.md) 哦。

![](https://files.mdnice.com/user/15734/71ea5f36-47ba-48bd-bfef-4dd919540eb6.png)
是时候了，需求来了！对于一些老代码中的 `options api`，通过 `Babel` 插件自动将它们转成 [composition api](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0013-composition-api.md)。

先通过简单的 🌰 代码感受两种写法的差异：

```js
import HelloWorld from 'HelloWorld'

export default {
  name: 'App',

  components: {
    HelloWorld
  },

  props: {
    firstName: {
      type: String,
      default: 'Jour'
    }
  },

  data () {
    return {
      lastName: 'Tom'
    }
  },

  computed: {
    name () {
      return this.firstName + this.lastName
    },

    secondName: {
      get () {
        return this.lastName + this.firstName
      }
    } 
  },

  watch: {
    name: {
      deep: true,
      handler (cur, prev) {
        console.log(cur, prev)
      }
    }
  },

  methods: {
    sayHello (aa) {
      console.log('say Hi')
      return aa
    }
  },

  beforeUnmount () {
    console.log('before unmount!')
  }
}
```

代码覆盖了写一个组件常见的属性，转成 `composition api` 写法的代码如下：

```js
import { computed, defineProps, onBeforeUnmount, reactive, watch } from 'vue'

const props = defineProps({
  firstName: {
    type: String,
    "default": 'Jour'
  }
});

const state = reactive({
  lastName: 'Tom'
});

const name = computed(function name() {
  return props.firstName + state.lastName;
});
const secondName = computed({
  get: function get() {
    return state.lastName + props.firstName;
  }
});

watch('name', (cur, prev) => {
  console.log(cur, prev);
}, {
  deep: true
});

const sayHello = aa => {
  console.log('say Hi');
  return aa;
};

onBeforeUnmount(() => {
  console.log('before unmount!');
});
```

### 分析

从上面 🌰 中可以得出我们要做的事：  

1. 基本写法转换，`props` 到 `defineProps`，`data` 到 `reactive`，`computed` 到 `computed` 函数，`watch` 到 `watch` 函数，`methods` 到单独的函数，生命周期钩子到 `onXXX` 函数；
2. 去除 `this` ，`composition` 跟 `this`没有半毛钱关系，所以要将 `this` 替换成对应的变量，比如 🌰 中用到的 `this.lastName`、`this.firstName` 要替换成 `state.lastName` 和 `state.firstName`；
3. 自动 `import` 用到的方法，用到的 `defineProps`、`reactive`、`computed` 等方法都需要单独 `import` 进来。

### 编码

先通过 [@babel/helper-plugin-utils](https://babeljs.io/docs/en/babel-helper-plugin-utils) 搭好插件的架子：

```js
import { declare } from '@babel/helper-plugin-utils'

export default declare((api) => {
  api.assertVersion(7)

  return {
    name: 'transform-options-to-composition',

    visitor: {
      Program (path) {
       
      },
    },
  }
})

```

我们先来看转换前的整体结构：

![](https://files.mdnice.com/user/15734/1e5d8b6e-0db7-4b6f-a473-f506cb3416eb.png)

再看看转换之后的整体结构：

![](https://files.mdnice.com/user/15734/5aeaef34-83d7-4ecf-b6f4-7374d3ba54df.png)

可以看到， 🌰 采用 `setup` 写法之后，就是要将 `Program` 节点下的 `body` 整个替换成新的内容，然后对红色框框的全部 `property` 做一个转换，全部转成新的写法，插件代码改成：

```js
import { declare } from '@babel/helper-plugin-utils'

export default declare((api) => {
  api.assertVersion(7)
   
  // types 即是 @babel/types
  const { types } = api
  
  // 定义一个变量，用来存储所有变化之后的结果，最后将这个节点数组赋值给原body即可
  const bodyContent = []

  return {
    name: 'transform-options-to-composition',

    // 访问者模式，访问到 type=Program 节点时就会进入这个函数
    // visitor 下的属性可以定义成函数（默认是 enter ）或者对象 { enter () {}, exit () {} }，这个 enter 和 exit 后面会讲到，这里略过 🤫🤫🤫
    visitor: {
      Program (path) {
        // 通过 types.isExportDefaultDeclaration 定位到 export default 结构体
        const exportDefaultDeclaration = path.node.body.filter((item) =>
          types.isExportDefaultDeclaration(item)
        )
        // 获取到上图中的 properties
        const properties = exportDefaultDeclaration[0].declaration.properties
      	
        // ...
          
        path.node.body = bodyContent
      },
    },
  }
})
```

OK，获取到了全部的 `properties`，接下来就是对每一个 `property` 做基础写法的转换，这里通过**策略模式**去获取处理函数，并将转换结果存入到 `bodyContent`：

```js
import { declare } from '@babel/helper-plugin-utils'

export default declare((api) => {
  api.assertVersion(7)
   
  // types 即是 @babel/types
  const { types } = api
  
  // 定义一个变量，用来存储所有变化之后的结果，最后将这个节点数组赋值给原body即可
  const bodyContent = []
  
  function genDefineProps (property) {}
  function genReactiveData (property) {}
  function genComputed (property) {}
  function genWatcher (property) {}
  function genMethods (property) {}
  function genBeforeUnmount (property) {}

  return {
    name: 'transform-options-to-composition',

    // 访问者模式，访问到 type=Program 节点时就会进入这个函数
    // visitor 下的属性可以定义成函数（默认是 enter ）或者对象 { enter () {}, exit () {} }，这个 enter 和 exit 后面会讲到，这里略过 🤫🤫🤫
    visitor: {
      Program (path) {
        // 通过 types.isExportDefaultDeclaration 定位到 export default 结构体
        const exportDefaultDeclaration = path.node.body.filter((item) =>
          types.isExportDefaultDeclaration(item)
        )
        // 获取到上图中的 properties
        const properties = exportDefaultDeclaration[0].declaration.properties
      	
        // 这里只列举了 🌰 中的属性
        // key 是 options api 中的配置项
        const GEN_MAP = {
          props: genDefineProps,
          data: genReactiveData,
          computed: genComputed,
          watch: genWatcher,
          methods: genMethods,
          beforeUnmount: genBeforeUnmount,
        }

        properties.forEach((property) => {
          // 获取key的名称，比如name、components、props、data...
          const keyName = property.key.name
          // 对于一些不需要的属性比如name、components直接不处理
          let newNode = GEN_MAP?.[keyName]?.(property)

          if (newNode) {
             Array.isArray(newNode)
               ? bodyContent.push(...newNode)
               : bodyContent.push(newNode)
            }
        })
          
        path.node.body = bodyContent
      },
    },
  }
})
```

到这里，基本的框架就算搭建完成了，接下来就是对各个属性的场景分析。本文因为主要是分享 `Babel` 的插件，所以场景不会做得特别完善，但是插件涉及到的工具接下来都会提及。接下来就来补充上述**分析**小节中的逻辑即完成 `genXXX` 函数。



#### 基本写法转换

先看看 `props` 编译之前的节点结构：

![](https://files.mdnice.com/user/15734/cd01719c-b064-4e31-8604-480a3702c527.png)

![](https://files.mdnice.com/user/15734/01f9f705-e776-463b-bca3-b082ce19eac8.png)

再看看转化之后的节点结构：

![](https://files.mdnice.com/user/15734/e4a234f8-dbcf-4bce-a70d-aaa3097c359e.png)

根据前后结构的对比：要做这个转换，把 `options api` 中 `props` 的值当作 `defineProps` 的参数即可。`genDefineProps` 通过 `@babel/types` 来完成：

```js
// options api
props: { ... }

// composition api
const props = defineProps({ ... })                          
```

这会可以翻到上面的 [AST Node Type](#AST Node Type) ，可以看到 `const props = defineProps({ ... })   ` 是一个变量声明语句，然后到 [variabledeclaration](https://www.babeljs.cn/docs/babel-types#variabledeclaration) 了解这个节点的创建参数：

![](https://files.mdnice.com/user/15734/fd3ad243-720a-4593-8707-c6e1d0f0d391.png)

根据 `variableDeclaration` 定义，可以得出 `genDefineProps` 函数：

```js
/**
 * props options api => compostion api
 */
function genDefineProps(property) {
    return types.variableDeclaration('const', [
        types.variableDeclarator(
            types.identifier('props'),
            // 将 options api 中 props 的值（property.value）作为 defineProps 的参数
            types.callExpression(types.identifier('defineProps'), [property.value])
        ),
    ])
}
```

接下来我们来看看 `data` 函数的处理过程，先对比转换前后的 `AST` 结构：

转换前：

![](https://files.mdnice.com/user/15734/bd8fb2f6-0b03-4023-9b02-52552e0d0cc3.png)

转换后：

![](https://files.mdnice.com/user/15734/52d14701-2ac1-4e98-b8ec-d3637347fca3.png)

转换分析：将 `options api` 中 `data` 结构中的 `ReturnStatement` 作为 `compostion api` 中 `reactive` 的参数。`data` 中其他逻辑通过 `IIFE` 去执行（这部分不是本文重点，省略！感兴趣的童鞋可以 `fork` [transform-options-to-compositions](https://github.com/Jouryjc/babel-plugin-transform-options-to-compositions) 玩玩~~😊😊😊）

```js
// options api
data () {
    return {
        lastName: 'Tom'
    }
}

// compostion api
const state = reactive({
  lastName: 'Tom'
})
```

这部分逻辑通过 `@babel/template` 来完成：

```js
/**
 * data options api => compostion api
 */
function genReactiveData(property) {
    // 通过template的ast参数生成"const state = reactive();"声明的ast
    const reactiveDataAST = template.ast(`const state = reactive();`)

    // 获取 data 中的 ReturnStatement 表达式
    const returnStatement = property.value.body.body.filter(node => types.isReturnStatement(node))
    
    // 将 reactive 的参数赋值 ReturnStatement
    reactiveDataAST.declarations[0].init.arguments.push(
        returnStatement[0].argument
    )

    return reactiveDataAST
}
```

可以看到，通过 `template` 去生成一段源码的 `AST`，然后通过修改 `AST` 节点的信息来快速完成转换过程，比直接使用 `@babel/types` 去生成和组合节点更快捷、更清晰。

`computed`、`watch`、`methods` 和生命周期函数就不展开细讲了，思路都是通过 `@babel/template` 快速生成 `composition` 的 `AST`，然后对修改参数即可。

#### 去除 this

在 `options api` 中，会存在大量的 `this.XXX` 的 `ThisExpression`，但是在 `composition api` 中，是没办法访问到 `this` 滴。那么如何将 `this` 去除呢？这里也要建立一个假设：假设我们 `this` 是访问存在 `props`、`data`、`computed`、`methods` 下的 `key`。一些特殊的情况，比如 `vue2` 中通过 `this.$set` 的变量，本文也不会处理，还是聚焦在 `Babel` 插件上。

简化场景之后，我们可以通过识别`props`、`data`、`computed`、`methods` 下有哪些 `key`，然后遍历 `ThisExpression`，将其父级的 `object` 设置成我们通过 `genXXX` 产生的变量。文字描述太抽象，下面我们通过 `AST` 来分析：

![](https://files.mdnice.com/user/15734/edbbb6ba-0e21-4b4c-9dff-888c9c2f1547.png)

![](https://files.mdnice.com/user/15734/40f6f592-5895-42a5-a928-9d1a63d8c61e.png)

我们在遍历 `props`、`data` 这些 `node` 时，将底下的 `key` 映射到我们在转换之后的生成的变量上（`const props = defineProps({ ... })`、`const state = reactive({ ... })`），最后通过遍历 `ThisExpression`，通过节点的关系替换表达式：

```js
import { declare } from '@babel/helper-plugin-utils'

export default declare((api) => {
  api.assertVersion(7)
   
  // types 即是 @babel/types
  const { types } = api
  
  // 定义一个变量，用来存储所有变化之后的结果，最后将这个节点数组赋值给原body即可
  const bodyContent = []
  const thisExpressionMap = {}
  
  // ...

  return {
    name: 'transform-options-to-composition',
      
    visitor: {
      // 进入 type = ObjectProperty 时执行 enter 函数
      ObjectProperty: {
        enter (path) {
          // 获取当前key的名称，值是options api下export default出去的key
          const keyName = path.node.key.name
          // 如果是以下这些key值，将它们子节点的key的名称映射到当前key上，举个🌰：
          // props: { firstName: 'xx' }，会生成 { firstName: 'props' }
          if (['props', 'computed', 'methods'].includes(keyName)) {
            path.node.value.properties.map(property => {
              thisExpressionMap[property.key.name] = keyName
            })
          }

           // data 是通过 ReturnStatement 去收集的，独立出来
          if (keyName === 'data') {
            const returnStatement = path.node.value.body.body.filter(node => types.isReturnStatement(node))
            returnStatement[0].argument.properties.map(property => {
              thisExpressionMap[property.key.name] = 'state'
            })
          }
        }
      },
      Program: {
        // 区别前面的写法！区别前面的写法！区别前面的写法！
        // 在 type=Program 退出时执行，因为Program是第二级节点，所以执行这个函数时，thisExpressionMap 已经缓存完了全部 this
        exit(path) {
          path.traverse({
            ThisExpression (p) {
              const propertyName = p.parent.property.name
			 // 通过this的缓存信息，可以得出当前this绑定的属性属于哪个变量
              if (thisExpressionMap[propertyName]) {
                p.parent.object = types.identifier(thisExpressionMap[propertyName]);
              }
            }
          })
       }
    },
  }
})
```

再补充一点：

![](https://files.mdnice.com/user/15734/7c56aaa2-b3ad-4896-a1d2-9961f1a951c5.jpg)

```js
// options api
export default {
    props: {
        age: 1
    },

    computed: {
        info () {
            return `My age：${this.age}`
        }
    }
}
```

生成的 `thisExpressionMap`：

```js
thisExpressionMap = {
    age: 'props'
}
```

最终生成结果：

```js
const props = defineProps({
    age: 1
})

const info = computed(() => {
    return `My age：${props.age}`
})
```

#### import 方法

简单说一下过程：在遍历 properties 时，识别对哪些 key 做了处理，做一个简单的 property 到方法的映射，最终通过 template 生成 AST，塞到 bodyContent 中。直接上代码：

```js
import { declare } from '@babel/helper-plugin-utils'

export default declare((api) => {
  api.assertVersion(7)
  const { types, template } = api

  // 缓存用了哪些API
  const importIdentifierMap = {}

  function hasImportIndentifier(item) {
    return ['props', 'data', 'computed', 'watch', 'beforeUnmount'].includes(
      item
    )
  }

  function genImportDeclaration(map) {
    const importSpecifiers = []
    const importMap = {
      props: 'defineProps',
      data: 'reactive',
      computed: 'computed',
      watch: 'watch',
      beforeUnmount: 'onBeforeUnmount',
    }

    Object.keys(map).forEach((item) => {
      const importIdentifier = hasImportIndentifier(item) ? importMap[item] : ''

      if (importIdentifier) {
        importSpecifiers.push(importIdentifier)
      }
    })
    return template.ast(`import {${importSpecifiers.join(',')}} from 'vue'`)
  }

  return {
    name: 'transform-options-to-composition',

    visitor: {
      // ...
      Program: {
        exit(path) {
          // ...

          properties.forEach((property) => {
            // ...

            if (newNode) {
              // 对于用到的函数，缓存起来
              importIdentifierMap[keyName] = true

              // ...
            }
          })

          // 根据引入了哪些函数，去生成 import 声明语句
          bodyContent.unshift(genImportDeclaration(importIdentifierMap))

          path.node.body = bodyContent
        },
      },
    },
  }
})
```

## 总结

至此，对于 🌰 中的插件需求算完成了。但仅仅是例子中的需求！实际使用还有很多很多功能需要实现，很多细节需要补充！感兴趣的童鞋可以 fork [源码](https://github.com/Jouryjc/babel-plugin-transform-options-to-compositions) 玩玩哦！

本文通过从 `vue2` 的 `options api` 的组件写法到 `vue3.2` 的 `setup` 写法，开发体验真的提高了很多。确实是时候去学习和使用了！历史包袱不可避免，本文通过 `Babel` 插件切入，提供升级的思路。从示例中使用到了 `@babel/core`、`@babel/types`、`@babel/traverse`、`@babel/template` 等工具包，也穿插地提到 `path` 相关概念（具体使用到时可参考 [Babel 插件手册](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md)）。

相信你读完本文，一定蠢蠢欲动，何不趁热打铁，一起来玩儿~ 🤙🤙🤙

