大家好，我是码农小余。作为手摸手实现编译器的终（下）篇，调皮地改了一个标题。回顾前两篇内容：

- [手摸手实现一个编译器（上）](https://mp.weixin.qq.com/s?__biz=MzUxOTcxNDU0Nw==&mid=2247484988&idx=1&sn=7e37700d9aef7150cf32b5d2667203c3&chksm=f9f425a0ce83acb6818655b8a89783edc76a52e0096cdacc2d45a267993500e588ae9065c506&token=1981764126&lang=zh_CN#rd) 通过 `PEG` 的 `API` 和讲解官方案例去了解了这个工具的基础用法；

- [手摸手实现一个编译器（中）](https://mp.weixin.qq.com/s?__biz=MzUxOTcxNDU0Nw==&mid=2247485013&idx=1&sn=b688fcf53115a3b71a60c13535910a73&chksm=f9f425c9ce83acdf2d982344baa2ec35737fa15f64038969b30a69009bead38101de5114690b&token=1981764126&lang=zh_CN#rd)我们实践了一个用中文写模板并最终解析成 `AST` 的例子，加深对 `PEG API` 的理解；

在上篇文末有讲到，编译成 `AST` 的之后需要 `transform`，最终 `generate` 代码。在 `vue` 的[模板编译](https://mp.weixin.qq.com/s?__biz=MzUxOTcxNDU0Nw==&amp;mid=2247483932&amp;idx=1&amp;sn=a0b50c4e28655c88721eba77b6824fff&amp;chksm=f9f42180ce83a8965119f8303897595f055052a41ffb77ca30fd091ec61433c2403bc6ab94f6&token=1981764126&lang=zh_CN#rd)中有 `optimize` 标记静态节点的优化和 `generate` 生成代码；在 `babel` 用 `@babel/traverse` 做节点遍历，用 `@babel/generator` 生成代码。今天小余就结合 `vue` 框架将生成的 `AST` 生成浏览器真实的 `DOM` ，以此来实践 `AST generate code` 的过程。

### 目标

要结合 vue 去生成有以下四种方式：

![](https://files.mdnice.com/user/15734/a4a36b55-a45e-49b3-842c-b60a1306ed50.png)


1. 通过 `AST` 生成 `render` 函数字符串（本文细讲这种方式，其他感兴趣的童鞋可以尝试练手）;
2. 通过转换 `AST`，生成 `vue` 中 `VNode` 的结构；
3. 通过 `AST` 生成 `SFC` 中的 `template`；
4. 通过 `AST` 去封装一套 `patch` 逻辑，通过 `DOM-API` 去处理；

### 简析
没有阅读过系列中篇的童鞋可能不太清楚状况，这里简单提一下。中篇我们有以下中文的模板：
```html
<下拉框 值="番茄">
  <选项 值="番茄">番茄</选项>
  <选项 值="香蕉">香蕉</选项>
</下拉框>
```
通过 `zh-template-compiler` 生成的 `AST` 结构如下：

```js
{
  "type": 1,
  "tag": "下拉框",
  "attrs": [
    {
      "isBind": false,
      "name": "值",
      "value": "番茄"
    }
  ],
  "children": [
    {
      "type": 1,
      "tag": "选项",
      "attrs": [
        {
          "isBind": false,
          "name": "值",
          "value": "番茄"
        }
      ],
      "children": [
        "番茄"
      ]
    },
    {
      "type": 1,
      "tag": "选项",
      "attrs": [
        {
          "isBind": false,
          "name": "值",
          "value": "香蕉"
        }
      ],
      "children": [
        "香蕉"
      ]
    }
  ]
}
```

将上述结构要生成在浏览器中能够显示的 `DOM`，我们需要的 `html` 代码就如下面这个样子（这里可以结合任意 `UI` 组件库去发挥，为了突出本文的重点，不把场景整复杂了）：

```html
<select value="番茄">
  <option value="番茄">番茄</option>
  <option value="香蕉">香蕉</option>
</select>
```

上述 `DOM` 转换成 `vue` 中 `render` 函数的写是：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>zh-template-compiler</title>
</head>
<body>
  <div id="app"></div>
  <script src="../node_modules/vue/dist/vue.global.js"></script>
  <script>
    const { createApp, h, ref } = Vue

    const app = createApp({     
      render (_ctx) {
        return h('select', {
          value: '番茄'
        }, [
          h('option', { value: '番茄' }, '番茄'),
          h('option', { value: '香蕉' }, '香蕉')
        ])
      }
    })

    app.mount('#app')
  </script>
</body>
</html>
```

既然这样就能显示，我们的任务就更加详细和明确了，将 `AST` 转换成 `render` 函数：

1. 中文和标签的映射，“下拉框”转换成 `select`，“选项”转换成 `option`；
2. `attrs` 转换成标签上的属性，`name` 等于“值”的 `attr` 全部生成 `value` = "xx" 的形式；
3. `children` 递归执行上述一、二步骤；

我们将 `AST` 转换成代码片段至此就分析完成了，接下来就开始撸代码。

### 测试

之前这一步骤都是直接贴测试代码了，那样可能对于问题的思考和测试代码的编写没有太大的指导意义。本文就由简入难一步一步地来写测试，使用的测试工具是 antfu 使用两周时间就冲上了 2021 测试框架排行榜第九名的 [Vitest](https://vitest.dev/ "Vitest") ，非常非常好用，快入坑。

首先，写单测一定要从**最小场景开始梳理**，对于本文的 🌰而言，最小的 `AST` 即：

```js
const ast: NODE = {
  type: 1,
  tag: "下拉框",
  attrs: [],
  children: []
}
```

上述 `AST` 对应的中文模板代码是 `<下拉框></下拉框>`，对应的 `html` 代码是 `<select></select>`，对应的 `render` 函数就很明了了：

```js
render (_ctx) {
    return h('select', {})
}
```

第一个测试用例也就出来了：

```js
describe("中文 ast 生成 render 函数", () => {
  test("单个不带属性节点", () => {
    const ast: NODE = {
      type: 1,
      tag: "下拉框",
      attrs: [],
      children: []
    }

    expect(generate(ast)).toBe(`render (_ctx) {
    return h('select', {})}`)
  })
})
```

然后第二个场景是组件带有属性的情况，`<下拉框 值="番茄"></下拉框>` 对应的 html 代码是 `<select value="番茄"></select>`，对应的 render 函数即：

```js
render (_ctx) {
    return h('select', {value: '番茄'})
}
```

第二个测试用例就出来了

```js
test('单个带属性的节点', () => {
  const ast: NODE = {
    type: 1,
    tag: '下拉框',
    attrs: [
      {
        isBind: false,
        name: '值',
        value: '番茄'
      }
    ],
    children: []
  }

  expect(generate(ast)).toBe(`render (_ctx) {
    return h('select', {"value":"番茄"})}`)
})
```

第三个用例自然就考虑 `children` 了，此时应该抛开第二个测试用例 `attrs` 的值，在写 `children` 的时候，因为 `children` 支持字符串和节点类型，所以按照由简入深原则，我们先考虑文本的场景 ：

```js
// 中文模板
<选项>番茄</选项>

// 对应的 html 代码
<option>番茄</option>

// 对应的 render 函数
render (_ctx) {
    return h('option', {}, '番茄')
}

// 生成的测试用例代码
test('带文本孩子的节点', () => {
  const ast: NODE = {
    type: 1,
    tag: '选项',
    attrs: [],
    children: ['番茄']
  }

  expect(generate(ast)).toBe(`render (_ctx) {
    return h('option', {}, '番茄')}`)
})
```

上述思考过程中有一个细节，为什么就不用 “下拉框”？突然转用“选项”了呢？如果考虑到这一点，就可能会因为好奇去查 [MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/select "MDN 文档")，顺道补充了自己的基础知识，这难道不是单元测试的魅力吗？

写完 children 是文本的情况，接下来就写 children 是节点的情况：

```js
// 中文模板
<下拉框>
  <选项></选项>  
</下拉框>

// 对应的 html 代码
<select>
  <option></option>
</select>

// 对应的 render 函数
render (_ctx) {
    return h('select', {}, [h('option', {})])
}

// 生成的测试用例代码
 test('只带节点孩子的节点', () => {
   const ast: NODE = {
     type: 1,
     tag: '下拉框',
     attrs: [],
     children: [
       {
         type: 1,
         tag: '选项',
         attrs: [],
         children: [
         ]
       }
     ]
   }

   expect(generate(ast)).toBe(`render (_ctx) {
    return h('select', {}, [h('option', {})])}`)
 })
```

到这里，所有单个属性的基本类型都考虑完了。接下来就是组合属性之间的测试代码，这部分就不一一列举了，排列组合，要将全部属性的搭配都考虑完全，直接给出代码：

```js
describe("ast 生成器", () => {

  // 省略上述已经分析过的用例...

  test('带两种类型孩子的节点', () => {
    const ast: NODE = {
      type: 1,
      tag: '下拉框',
      attrs: [],
      children: [
        {
          type: 1,
          tag: '选项',
          attrs: [],
          children: [
            '番茄'
          ]
        }
      ]
    }

    expect(generate(ast)).toBe(`render (_ctx) {
      return h('select', {}, [h('option', {}, '番茄')])}`)
  })

  test('带标签孩子的节点', () => {
    const ast: NODE = {
      type: 1,
      tag: '下拉框',
      attrs: [],
      children: [
        {
          type: 1,
          tag: '选项',
          attrs: [
            {
              isBind: false,
              name: '值',
              value: '番茄'
            }
          ],
          children: [
            '番茄'
          ]
        }
      ]
    }

    expect(generate(ast)).toBe(`render (_ctx) {
      return h('select', {}, [h('option', {"value":"番茄"}, '番茄')])}`)
  })

  test('带2个标签孩子的节点', () => {
    const ast: NODE = {
      type: 1,
      tag: '下拉框',
      attrs: [],
      children: [
        {
          type: 1,
          tag: '选项',
          attrs: [],
          children: [
            '番茄'
          ]
        },
        {
          type: 1,
          tag: '选项',
          attrs: [],
          children: [
            '香蕉'
          ]
        }
      ]
    }

    expect(generate(ast)).toBe(`render (_ctx) {
      return h('select', {}, [h('option', {}, '番茄'), h('option', {}, '香蕉')])}`)
  })
});
```

写完了测试用例，此时运行测试：

```sh
vitest -c vite.config.ts -u
```

因为我们一行代码都没写，所以自然全红：

![](https://files.mdnice.com/user/15734/3b87b094-5941-46e5-a74e-99bb345c6455.png)

### 编码

有了单元测试，接下来就将我们的注意力全部集中到代码上，这个环节我们只需怎么把代码写好即可，不会出现一边想需求一边写代码，中间发现有不满足的需求还有各种补充逻辑的窘境。大部分时候，每个人写的第一手代码都很 beautiful，但因为后续补充需求场景和迭代，就成了“屎山”。

回到 🌰 中需求，深度遍历 `AST`，去生成代码即可，核心代码如下：

```typescript
function generateItem (node: NODE) {
  const { attrs, tag, } = node

  // 根据中文 tag 获取具体的 html 标签
  const dom = getTag(tag)
  // 根据中文属性名获取具体的 dom 属性
  const props = generateAttrs(attrs)

  return `h('${dom}', ${JSON.stringify(props)}`
}

export function generate (ast: NODE): string {
  let code = `render (_ctx) {
    return `

  function dfs (node: NODE) {
    if (!node) {
      return;
    }

    let str = generateItem(node)
    let children = node.children
    let len = children.length

    // 文本的情况
    if (len === 1 && typeof children[0] === 'string') {
      str += `, '${children[0]}')`
    // 没有子节点
    } else if (len === 0) {
      str += ')'
    } else {
      // 子节点数组
      let childrenArr = []
      for (let item of children) {
        childrenArr.push(dfs(item as NODE))
      }

      str += `, [${childrenArr.join(', ')}])`
    }

    return str
  }

  code += `${dfs(ast)}}`
  return code
}
```

代码比较简单，感兴趣的可以前往 [github](https://github.com/Jouryjc/zh-template-compiler "github") 查看整体代码，整个过程类似 `vue` 中的 `VNode` 通过 `patch` 渲染 `DOM` 过程。经测试，测试用例全部变成绿色：

![](https://files.mdnice.com/user/15734/642f4080-ce47-4a95-bd4b-d041bc117d0d.png)

`compiler` 和 `generate` 都全部没问题了，接下来就整一个 `DEMO`，将二者结合起来：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  </style>
  <title>zh-template-compiler</title>
</head>

<body>
  <div id="app"></div>
  <script src="../packages/parser/dist/zh-template-compiler.global.js"></script>
  <script src="../packages/generate/dist/zh-template-generate.global.js"></script>
  <script src="../node_modules/vue/dist/vue.global.js"></script>
  <script>
    
    const template = `<下拉框 值="番茄">
      <选项 值="番茄">番茄</选项>
      <选项 值="香蕉">香蕉</选项>
    </下拉框>`;
    const ast = zhTemplateCompiler.parse(template)

    const { createApp, h, ref } = Vue

    const app = createApp({
      render (_ctx) {
        const fn = new Function(`return ${zhTemplateGen.generate(ast)}`)

        return fn()
      }
    })
    app.mount('#app')
  </script>
</body>

</html>
```

最后运行 `html`：

![](https://files.mdnice.com/user/15734/e786544e-115b-4f20-b01d-7853452984b3.gif)

### 总结

作为编译器系列的最后一篇文章，将中篇中文模板生成的 `AST` 通过遍历并生成最终 `render` 代码后，基本就走过了 `parse`、`traverse`、`generate` 三个步骤。除了使用简单有趣的例子辅助理解之外，文中还有大量的热点技术使用，比如 `pnpm`、`vitest`；最后还有一些常用的开发技巧，比如 `TDD` 的详细步骤指引，使用 `pnpm workspace` 的组织方式等等。
