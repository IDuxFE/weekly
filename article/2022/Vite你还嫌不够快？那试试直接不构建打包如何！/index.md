## 背景

随着IE已经退出历史舞台，且目前大多数浏览器都内置了对 module 的支持，因此构建渐渐成为了一个**优化步骤**而不是必要步骤 。那么，初衷是为了支持某些古老版本浏览器运行而衍生出来的构建步骤，我们还需要保留吗？🙈

## 当前开发流程中存在哪些问题？

-  构建流程长，成本高

目前前端的构建涉及到下图所示的多个步骤（ 依赖的拉取、依赖图分析、打包依赖... ），如果在敏捷开发的场景下，release分支需要经常更新，这无疑会造成流水线的阻塞；同时改动少数代码却导致整个构建流程进行，也造成了资源的浪费。![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/40f0e632e9094c35ad38df8d379cbc77~tplv-k3u1fbpfcp-zoom-1.image)


- 依赖包基于版本号进行发布更新，极其缓慢

因为依赖包之间是一个单向的信息流，如下图：A dependencies的更新是不会主动通知UI Component的；而对于处于最上层的Web APP来说，往往就会因为依赖包的某一点阻塞，从而导致无法快速更新版本。  
我们不妨假设一个场景：一个业务开发者**小明**发现A dependencies有无法hack的bug，那么他只能不断地去关注A dependencies的更新情况，获取更新版本后，还需要修改package.json，以便触发构建过程，修复这个bug。如果恰巧A dependencies的依赖包A-a dependencies也存在这样的bug呢？那么这个过程的阻塞时间将会成倍增长。 😹
   ![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/910e6b6f92014eef8e239bcb9f0b3613~tplv-k3u1fbpfcp-zoom-1.image)

- 构建配置复杂繁琐

虽然各种构建工具、脚手架预设配置已经尽可能简单，但是对于大型项目来说，定制化配置不可缺少。
webpack因其复杂性，甚至出现了专门的webpack配置工程师；vite虽然在一定程度上有所优化，但是并没有切底这个问题。

## 为什么ESM有望解决这些问题？

```html
ESM，即ECMA Script Modules，简介：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Modules
```

如果项目使用了ESM，会发生什么神奇的事情呢？还是拿**小明**来举例，当A dependencies的bug被修复后，用户侧将通过import直接加载fixed后的代码，完全不需要重新触发构建（如果A dependencies的模块拆分合理，甚至只需要更新fixed对应的代码），即可完成项目的更新。

## Talk is cheap，show me the code！

接下来我们尝试使用ESM+esinstall实现一个无构建的Hello World。💯

- 设置index.html

我们在script标签中声明type="module"，并引入主js文件main.js。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script type="module" src="../js/main.js"></script>
</head>
<body>
    <h1>Hello World</h1>
    <my-element></my-element>
</body>
</html>
```

- npm init

像普通项目一样新建package.json，列出依赖项。


```json
{
  "name": "no-packaging",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "build": "node --experimental-modules build-modules.mjs",
    "postinstall": "yarn build"
  },
  "keywords": [
    "esinstall"
  ],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "dom-confetti": "^0.2.2",
    "lit": "^2.2.1"
  },
  "devDependencies": {
    "esinstall": "^1.1.7"
  }
}

```


- esinstall设置

```html
esinstall简介：https://www.npmjs.com/package/esinstall
```

当前情况，放弃npm丰富的生态系统显然是不理智且不现实的。因此我们采用esinstall来将依赖包转换为ESM。

我们设置了一个buildModules选项，用于声明我们需要转换为ESM的依赖项。
并且创建一个[postinstall](https://docs.npmjs.com/cli/v8/using-npm/scripts)脚本，在每次install后自动运行，这个脚本执行build-modules.mjs。

```json
{
  "name": "no-packaging",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "build": "node --experimental-modules build-modules.mjs",
    "postinstall": "yarn build"
  },
  "keywords": [
    "esinstall"
  ],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "dom-confetti": "^0.2.2",
    "lit": "^2.2.1"
  },
  "devDependencies": {
    "esinstall": "^1.1.7"
  },
  "buildModules": {
    "install": [
      "lit",
      "dom-confetti"
    ],
    "installOptions": {
      "dest": "js/vendor"
    }
  }
}

```

而build-modules.mjs所做的就是读取package.json中需要转换的依赖列表，并交给esinstall进行转换。
```js
import { install } from 'esinstall';
import { readFile } from 'fs/promises';

const json = JSON.parse(
  await readFile(
    new URL('./package.json', import.meta.url)
  )
);
const moduleList = json.buildModules.install;
const installOptions = json.buildModules.installOptions;

await install(moduleList, installOptions);

```

- Hello World

执行yarn install后，我们使用[lit](https://lit.dev/)在main.js中创建一个自定义[Web Components](https://developer.mozilla.org/zh-CN/docs/Web/Web_Components#%E6%A6%82%E5%BF%B5%E5%92%8C%E4%BD%BF%E7%94%A8)组件，组件包含了一个输入框和事件监听器、事件处理方法；并使用[dom-confetti](https://github.com/daniel-lundin/dom-confetti)加上一点动效。💥


```js
import {LitElement, html} from './vendor/lit.js';
import { confetti } from "./vendor/dom-confetti.js";

confetti(document.body, { angle: 0, duration: 10000 });

class MyElement extends LitElement {
    static properties = {
      name: {},
    };
  
    constructor() {
      super();
      this.name = 'Your name here';
    }

    changeName(event) {
        const input = event.target;
        this.name = input.value;
    }
  
    render() {
      return html`
        <p>Hello, ${this.name}</p>
        <input placeholder="Enter your name" @input=${this.changeName}
        >
      `;
    }
  }

customElements.define('my-element', MyElement);


```


- 测试

一切准备就绪后，我们使用本地服务器，访问index.html，无需打包就可以看到编写的应用了。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ec171d01b73749c3a34c5cf265bbe604~tplv-k3u1fbpfcp-zoom-1.image)


## 当前方案的局限性

从Hello World中我们可以发现，当前ESM方案还远未成熟，无法在生产环境中使用，原因如下：

- 很多依赖库对ESM支持不佳，需要使用esinstall进行转换
- 没有构建打包，多个小文件使得浏览器解压缩开销增大，大型应用可能存在性能风险
- 没有babel，需要手动处理跨浏览器问题
- 无法使用与构建工具深度绑定的两大前端框架（Vue、React）

## ESM的未来

虽然ESM目前还存在种种问题，但是前端领域需要更快的迭代时间和更少的构建步骤的趋势是无可阻挡的。

毕竟，TypeScript团队为了不让编译类型成为TypeScript代码到运行之间的唯一步骤，已经求生欲满满的提出了直接在JavaScript中支持类型的[stage0](https://github.com/giltayar/proposal-types-as-comments)提案。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f648de7d3bd64bb384b70de99ca4a54e~tplv-k3u1fbpfcp-zoom-1.image)

再深一层想，webpack怎么也不会想到vite竟然这么不可匹敌。而谁，又是下一个vite呢？😉
