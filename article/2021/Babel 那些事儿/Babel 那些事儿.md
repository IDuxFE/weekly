## What is Babel?

`Babel` 是一个工具链，主要用于将采用 `ECMAScript` 2015+ 语法编写的代码转换为向后兼容的 `JavaScript` 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。（我摊牌了，直接从 [`Babel` 中文官网](https://www.babeljs.cn/docs/ "`Babel` 中文官网")复制），我们一般用 `Babel` 做下面几件事：

- 语法转换（`es-higher` -> `es-lower`）；
- 通过 Polyfill 处理在目标环境无法转换的特性（通过 `core-js` 实现）；
- 源码转换（`codemods`、`jscodeshift`）；
- 静态分析（`lint`、根据注释生成 `API` 文档等）;

`Babel` 真可以为所欲为！😎😎😎

##  Babel7 最小配置

不知道你刚玩 Babel 时，有没那种被“初恋伤害”的感觉？如果有，那么请细品这一小节，它会让你欲罢不能。

在开始品“初恋的味道”前，咱们先做一些准备：  

新建一个目录 `babel-test` 然后创建 `package.json` 文件:

```bash
mkdir babel-test && cd babel-test

// 本文全部用 yarn
yarn init -y
```

安装好 `@babel/core`、`@babel/cli`：

```bash
yarn add @babel/core @babel/cli -D
```

万事俱备，现在只需要买一盘 🌰 ，你就可以牵她的手啦：

```js
// 在根目录新建 index.js 文件，然后键入下面的 🌰
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
console.log(x); // 1
console.log(y); // 2
console.log(z); // { a: 3, b: 4 }
```

初恋嘛，刚碰到对方的汗毛，你就脸红了，然后想看看对方的反应，对吧？所以执行下面的命令看看有什么结果：

```bash
// babel 是前面安装了 @babel/cli 才能用哦~
npx babel ./index.js --out-file build.js
```

执行完上面的命令，会在根目录输出一个 `build.js` 文件，打开一看：

```js
let {
  x,
  y,
  ...z
} = {
  x: 1,
  y: 2,
  a: 3,
  b: 4
};
console.log(x); // 1

console.log(y); // 2

console.log(z); // { a: 3, b: 4 }

```

`What the xxx?` 这 ** 不就只是格式化了嘛！放在 `IE10` 上一跑，又是一个不眠之夜的信号：

![ie-error](https://files.mdnice.com/user/15734/1d6d618f-bfd6-4628-8424-661f3a50b319.png)

惊不惊喜意不意外？[caniuse](https://caniuse.com/ "caniuse") 一查，我尼玛，哪个*逼用扩展运算符啊，不知道我们要兼容`IE` 啊！

![object-rest-spread-caniuse](https://files.mdnice.com/user/15734/6581b45d-6bfa-4ad0-82f5-0b60b862d728.png)

但是作为勇猛的追求者，我们怎能因为对方手缩了一下就放弃呢！进到 [Babel 插件页面](https://www.babeljs.cn/docs/plugins-list#es2018 "Babel 插件页面")，看需要什么插件能处理扩展运算符——可以看到这是一个 `ES2018` 的特性，通过 [@babel/plugin-proposal-object-rest-spread](https://www.babeljs.cn/docs/babel-plugin-proposal-object-rest-spread "@babel/plugin-proposal-object-rest-spread") 插件就可以用啦。

冲！再一次伸出你黝黑的手。在项目的根目录（`package.json` 文件所在的目录）下创建一个名为 `babel.config.json` 的文件（具体创建 `.babelrc`、还是 `babel.config.js` ，可以依据自己的场景选择，文件可以参考[配置 Babel](https://www.babeljs.cn/docs/configuration "配置 Babel")），并输入如下内容：

```json
// 先到终端输入 yarn add @babel/plugin-proposal-object-rest-spread -D，安装依赖先
{
    "plugins": ["@babel/plugin-proposal-object-rest-spread"]
}
```

然后再执行：

````bash
npx babel ./index.js --out-file build.js
````

然后再打开 `build.js` 文件，这时可以看到扩展运算符已经见不到啦：

```js
function _objectWithoutProperties(source, excluded) { if (source == null) return {}; var target = _objectWithoutPropertiesLoose(source, excluded); var key, i; if (Object.getOwnPropertySymbols) { var sourceSymbolKeys = Object.getOwnPropertySymbols(source); for (i = 0; i < sourceSymbolKeys.length; i++) { key = sourceSymbolKeys[i]; if (excluded.indexOf(key) >= 0) continue; if (!Object.prototype.propertyIsEnumerable.call(source, key)) continue; target[key] = source[key]; } } return target; }

function _objectWithoutPropertiesLoose(source, excluded) { if (source == null) return {}; var target = {}; var sourceKeys = Object.keys(source); var key, i; for (i = 0; i < sourceKeys.length; i++) { key = sourceKeys[i]; if (excluded.indexOf(key) >= 0) continue; target[key] = source[key]; } return target; }

let _x$y$a$b = {
  x: 1,
  y: 2,
  a: 3,
  b: 4
},
    {
  x,
  y
} = _x$y$a$b,
    z = _objectWithoutProperties(_x$y$a$b, ["x", "y"]);

console.log(x); // 1

console.log(y); // 2

console.log(z); // { a: 3, b: 4 }

```

刷新 IE 浏览器，打开 F12 看看调试程序面板：

![ie-error-destructuing](https://files.mdnice.com/user/15734/fb147add-b2c0-4cf4-ac82-d956f1f36158.png)

她再一次缩手了，心痛不？但是作为一名戴着红领巾，头上印着小红花的男人，绝不气馁！看到错误的代码位置，能识别到 IE 连解构赋值都不支持。同样的过程，查 [caniuse](https://caniuse.com/?search=Destructuring "caniuse") 和 [@babel/plugin-transform-destructuring](https://www.babeljs.cn/docs/babel-plugin-transform-destructuring "@babel/plugin-transform-destructuring") （提示：点击可以直接跳转到对应页面哦！）

这一次，再去牵她的手，`gogogo`：

```json
// 先到终端输入 yarn add @babel/plugin-transform-destructuring -D，安装依赖先
{
    "plugins": [
        "@babel/plugin-proposal-object-rest-spread",
        "@babel/plugin-transform-destructuring"
    ]
}
```

安装完之后再编译一次，可以看到生成的代码如下：

```js
function _objectWithoutProperties(source, excluded) { if (source == null) return {}; var target = _objectWithoutPropertiesLoose(source, excluded); var key, i; if (Object.getOwnPropertySymbols) { var sourceSymbolKeys = Object.getOwnPropertySymbols(source); for (i = 0; i < sourceSymbolKeys.length; i++) { key = sourceSymbolKeys[i]; if (excluded.indexOf(key) >= 0) continue; if (!Object.prototype.propertyIsEnumerable.call(source, key)) continue; target[key] = source[key]; } } return target; }

function _objectWithoutPropertiesLoose(source, excluded) { if (source == null) return {}; var target = {}; var sourceKeys = Object.keys(source); var key, i; for (i = 0; i < sourceKeys.length; i++) { key = sourceKeys[i]; if (excluded.indexOf(key) >= 0) continue; target[key] = source[key]; } return target; }

let _x$y$a$b = {
  x: 1,
  y: 2,
  a: 3,
  b: 4
},
    x = _x$y$a$b.x,
    y = _x$y$a$b.y,
    z = _objectWithoutProperties(_x$y$a$b, ["x", "y"]);

console.log(x); // 1

console.log(y); // 2

console.log(z); // { a: 3, b: 4 }

```

再次看 `IE`浏览器的反应：

![ie-success](https://files.mdnice.com/user/15734/1dbb802e-268a-461a-a29b-5086f080fd95.png)

皇天不负有心人，`IE` 成了，你也牵手成功了！

细心的你不知道有没发现，在这两个 `Babel` 插件名字底下都有一个显眼的 NOTE：

> NOTE: This plugin is included in `@babel/preset-env`

啥意思呢？女生说希望你下次胆子再大点，一次就能牵上然后不放，为啥要多次尝试呢！

就此引出 [@babel/preset-env](https://www.babeljs.cn/docs/babel-preset-env "@babel/preset-env") ，跟着文档先把这个包装上，配置文件的 `presets` 字段配上。然后前面两个插件去掉。

```bash
yarn remove @babel/plugin-transform-destructuring @babel/plugin-proposal-object-rest-spread

yarn add @babel/preset-env -D
```

`babel.config.json` 改成如下：

```json
{
  "presets": [
    [
      "@babel/preset-env"
    ]
  ]
}
```

然后再执行一次构建命令，可以看到输出的 `build.js` 文件是一样的！

惊叹的同时也在想：

- 为什么一个预设就能满足转换需求呢？它是怎么做到的？
- `Babel` 怎么知道我要支持 `IE` 浏览器，如果我只使用 `Chrome`，那么这个转换不是多余了么？而且不仅仅是浏览器，Babel 在桌面端、node 的场景都不少，它是怎么精确控制转换的？

回答上面的问题之前，突然想到一件事，之前在公司 `review` 代码时，看到很多童鞋为了使用 `TypeScript` 而被 `TypeScript` 支配（比如 `AnyScript` 的叫法由来）。希望都能从技术、工具、框架本身的诞生背景、作用去思考！如果不做到比较精细的类型声明和限制，为何用它？

`Babel` 也一样，[Babel6 到 Babel7 的升级](https://www.babeljs.cn/docs/v7-migration "Babel6 到 Babel7 的升级")：

- 废弃了 [`stage-x`](https://tc39.es/process-document/) 和 [`es20xx`](https://www.babeljs.cn/blog/2017/12/27/nearing-the-7.0-release.html#deprecated-yearly-presets-eg-babel-preset-es20xx) 的 `preset`，改成 `preset-env` 和 `plugin-proposal-xx`，具体的提案信息都在 [TC39/proposals](https://github.com/tc39/proposals/blob/master/README.md) 查阅，这样能更好地控制需要支持的特性；
- [preset-env](https://www.babeljs.cn/docs/babel-preset-env "preset-env") 依赖 [`browserslist`](https://github.com/browserslist/browserslist "`browserslist`"), [`compat-table`](https://github.com/kangax/compat-table "`compat-table`"), and [`electron-to-chromium`](https://github.com/Kilian/electron-to-chromium "`electron-to-chromium`") 实现了特性的精细按需引入。

### compat-table

这个库维护着每个特性在不同环境的支持情况，来看看上面用到的解构赋值的支持：

```js
{
  name: 'destructuring, declarations',
  category: 'syntax',
  significance: 'medium',
  spec: 'http://www.ecma-international.org/ecma-262/6.0/#sec-destructuring-assignment',
  mdn: 'https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment',
  subtests: [
    {
      name: 'with arrays',
      exec: function(){/*
        var [a, , [b], c] = [5, null, [6]];
        return a === 5 && b === 6 && c === void undefined;
      */},
      res: {
        tr: true,
        babel6corejs2: babel.corejs,
        ejs: true,
        es6tr: true,
        jsx: true,
        closure: true,
        typescript1corejs2: true,
        firefox2: true,
        opera10_50: false,
        safari7_1: true,
        ie11: false,
        edge13: edge.experimental,
        edge14: true,
        xs6: true,
        chrome49: true,
        node6: true,
        node6_5: true,
        jxa: true,
        duktape2_0: false,
        graalvm19: true,
        graalvm20: true,
        graalvm20_1: true,
        jerryscript2_0: false,
        jerryscript2_2_0: true,
        hermes0_7_0: true,
        rhino1_7_13: true
      }
    }
```

`ie11` 是 `false` 的，怪不得第一次牵手失败了。

### browserslist

这个包应该比较熟悉了，可以通过 `query` 查询具体的浏览器列表，下面安装上这个包然后来实操一波：

```bas
yarn add browserslist -D

// 查询的条件各种骚操作都有，具体可参考 https://github.com/browserslist/browserslist#queries
npx browserslist "> 0.25%, not dead"
and_chr 91
and_ff 89
and_uc 12.12
android 4.4.3-4.4.4
chrome 91
chrome 90
chrome 89
chrome 87
chrome 85
edge 91
firefox 89
firefox 88
ie 11
ios_saf 14.5-14.7
ios_saf 14.0-14.4
ios_saf 13.4-13.7
op_mini all
opera 76
safari 14.1
safari 14
safari 13.1
samsung 14.0
samsung 13.0
```

有了上面两个包，那 `preset-env` 实现特性精细控制岂不是洒洒水。继续实操，我们把开头那个 🌰 改成不需要支持 `ie11` 试试看：

```json
{
  "presets": [
    [
	    "@babel/preset-env",
        {
    		"targets": {
                "chrome": 55
            }        
        }
    ]
  ]
}
```

`preset-env` 控制浏览器版本是通过配置 `targets` 字段。构建一下看看结果：

```js
"use strict";

function _objectWithoutProperties(source, excluded) { if (source == null) return {}; var target = _objectWithoutPropertiesLoose(source, excluded); var key, i; if (Object.getOwnPropertySymbols) { var sourceSymbolKeys = Object.getOwnPropertySymbols(source); for (i = 0; i < sourceSymbolKeys.length; i++) { key = sourceSymbolKeys[i]; if (excluded.indexOf(key) >= 0) continue; if (!Object.prototype.propertyIsEnumerable.call(source, key)) continue; target[key] = source[key]; } } return target; }

function _objectWithoutPropertiesLoose(source, excluded) { if (source == null) return {}; var target = {}; var sourceKeys = Object.keys(source); var key, i; for (i = 0; i < sourceKeys.length; i++) { key = sourceKeys[i]; if (excluded.indexOf(key) >= 0) continue; target[key] = source[key]; } return target; }

let _x$y$a$b = {
  x: 1,
  y: 2,
  a: 3,
  b: 4
},
    {
  x,
  y
} = _x$y$a$b,
    z = _objectWithoutProperties(_x$y$a$b, ["x", "y"]);

console.log(x); // 1

console.log(y); // 2

console.log(z); // { a: 3, b: 4 }

```

从上面源码可以看出来，**扩展运算符被转换了，解构赋值没有被转换**。被转换的特性通过模块内定义了两个方法 `_objectWithoutProperties` 和 `_objectWithoutPropertiesLoose`。如果我**有两个文件都使用了扩展运算符**，然后输出一个文件，结果会怎样呢？根目录下新建一个 `index2.js` 文件：

```js
let { x, y, ...z } = { x: 1, y: 2, a: 3, b: 4 };
console.log(x); // 1
console.log(y); // 2
console.log(z); // { a: 3, b: 4 }
```

然后分别执行下面两条命令：

```bas
npx babel ./index.js ./index2.js --out-file build.js
```

结果是 `_objectWithoutProperties` 和 `_objectWithoutPropertiesLoose` 居然都会重复声明两次。这对于需要转换的特性，我使用很多次，转换后输出的文件不是爆炸了么？此时需要一个插件来控制代码量——[@babel/plugin-transform-runtime](https://www.babeljs.cn/docs/babel-plugin-transform-runtime "@babel/plugin-transform-runtime") 。对于这种转换函数，在外部模块化，用到的地方直接引入即可。实操：

```ba
// 先安装 @babel/plugin-transform-runtime 包
yarn add @babel/plugin-transform-runtime -D
```

然后配置 `babel`：

```json
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": {
                    "chrome": "55"
                }
            }
        ]
    ],
    "plugins": [
        "@babel/plugin-transform-runtime"
    ]
}
```

再执行上面的构建命令，得到以下结果：

```js
"use strict";

var _interopRequireDefault = require("@babel/runtime/helpers/interopRequireDefault");

var _objectWithoutProperties2 = _interopRequireDefault(require("@babel/runtime/helpers/objectWithoutProperties"));

let _x$y$a$b = {
  x: 1,
  y: 2,
  a: 3,
  b: 4
},
    {
  x,
  y
} = _x$y$a$b,
    z = (0, _objectWithoutProperties2.default)(_x$y$a$b, ["x", "y"]);
console.log(x); // 1

console.log(y); // 2

console.log(z); // { a: 3, b: 4 }

let _a$b$c = {
  a: 1,
  b: 2,
  c: 3
},
    {
  a,
  b
} = _a$b$c,
    c = (0, _objectWithoutProperties2.default)(_a$b$c, ["a", "b"]);

```

对比上面的转换结果，这次转换结果精简了不少。并且函数声明都是通过外部引入。

再来看下面这段代码：

```js
const a = [1,2,3,4,6];
console.log(a.includes(7))
```

通过 `@babel/compat-data` 可以看下 includes 特性的兼容性：

```json
"es7.array.includes": {
    "chrome": "47",
    "opera": "34",
    "edge": "14",
    "firefox": "43",
    "safari": "10",
    "node": "6",
    "ios": "10",
    "samsung": "5",
    "electron": "0.36"
}
```

`chrome` 47+ 支持数组 `includes API`，我们把 `babel.config.json` 的 `targets` 改成 **45** 然后执行转换命令，结果如下：

```js
"use strict";

var a = [1, 2, 3, 4, 6];
console.log(a.includes(7));
```

可以得出结论：虽然不支持 `Array.prototype.inlcudes`，但是 `babel` 默认不会对实例方法做转换。这时候就需要引入 `@babel/polyfill` 打补丁。（⚠️ 安装 `polyfill` 包是 `dependency` 哦！因为在**生产环境**上垫片是要在你的代码前执行。）

在项目入口文件或者在打包工具比如 `webpack` 的 `entry` 一把梭把全部 `polyfill` 引进来：

```js
// app.js
import '@babel/polyfill';

// webpack.config.js
module.exports = {
  entry: ["@babel/polyfill", "./app/js"],
};
```

其中很多特性的垫片我们都用不着，那么能不能也结合上述的 `broswer targets` 和代码中使用到的函数去做定制的垫片呢？ `Of course`，在这里推荐一个在线定制 `polyfill` 的[网站 ](https://polyfill.io/v3/url-builder/ "网站 ")，选择完自己的垫片，然后生成一个 `CDN URL`。在项目中直接引入就可以啦，这可以用于微型的网站，对于超大型的项目，不可能自己一个一个方法去选择吧。这就要引出 [useBuiltIns](https://babeljs.io/docs/en/babel-preset-env#usebuiltins "useBuiltIns") 配置，它定义了 `@babel/preset-env` 怎么处理垫片。可选的值有：

- `usage`：每个文件引用使用到的特性；
- `entry`：入口处全部引入；
- `false`：不引入。

![have-one-example](https://files.mdnice.com/user/15734/4e1cf27e-0547-467b-910f-a018154887b9.jpg)

```js
// index.js
const a = [1,2,3,4,6];

console.log(a.includes(7))

new Promise(() => {})
```

然后将 babel 的配置改成如下：

```json
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "targets": {
                    "chrome": "45",
                    "ie": 11
                },
                "useBuiltIns": "usage",
                "corejs": 3
            }
        ]
    ],
    "plugins": [
        "@babel/plugin-transform-runtime"
    ]
}
```

在终端执行 `babel ./index.js --out-file build.js`，看看 `build.js` 的结果：

```js
"use strict";

require("core-js/modules/es.array.includes.js");

require("core-js/modules/es.object.to-string.js");

require("core-js/modules/es.promise.js");

var a = [1, 2, 3, 4, 6];
console.log(a.includes(7));
new Promise(function () {});

```

`niubility`！对于不支持的特性都引入了特定的 `core-js` 垫片。这怎么做到的呢？这还是归功于 `AST`，它可以结合代码的实际情况，进行超级细的按需引用。感兴趣的童鞋可以看看 `core-js` 和 `babel` 的协作方式哦。

### 小结

通过 🌰 去一步一步分析 `Babel7` 最小最优配置的产生，其中还涉及一些写配置中无感知的处理机制，比如 `compat-table`、`browserslist`。读完本节，相信你对 `babel7` 配置方法有一个清晰的了解。

## @babel 系列包

`Babel` 是一个 `Monorepo` 项目，`packages` 下面有 **146** 个包。Unbelievable！包虽多，我们可以将它们划分为几个类别：

`@babel/helper-xx` 有 28 个，`@babel/plugin-xx` 有 98 个。剩下的工具包、集成包总共也才 20 个。我们挑一些有意思的 `package` 来了解它们的作用。

### @babel/standalone

[babel-standalone](https://babeljs.io/docs/en/babel-standalone "babel-standalone") 提供独立构建的 `Babel` 用于浏览器和其他非 `Node` 环境，比如在线 `IDE`： [JSFiddle](https://jsfiddle.net/ "JSFiddle")、[JS Bin](https://jsbin.com/?html,js,console,output "JS Bin")、还有 Babel 官方的 [try it out](https://babeljs.io/repl#?browsers=defaults%2C%20not%20ie%2011%2C%20not%20ie_mob%2011&build=&builtIns=false&corejs=3.6&spec=false&loose=false&code_lz=Q&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=env%2Creact%2Cstage-2&prettier=false&targets=&version=7.14.7&externalPlugins= "try it out") 都是基于这个包。我们也来玩儿~

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>standalone</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.0.0-beta.3/babel.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
</head>
<body>
    <div id="app"></div>
    <script type="text/babel">
        const codeStr = `const getMessage = () => "Babel, 为所欲为";`;
        const code = Babel.transform(codeStr, { presets: ["env"] }).code;
        document.querySelector('#app').innerHTML = code;
    </script> 
</body>
</html>
```

上述代码直接运行在浏览器，得到的 `code` 如下：

```js
"use strict"; var getMessage = function getMessage() { return "Babel, 为所欲为"; };
```

跟在 node 环境构建出来的结果是一样的。

### @babel/plugin-xx

满足这种标记的都是 `Babel` 插件。主要用来加强 `transform`、`parser` 能力。举个 🌰：

```js
// index.js
const code = require("@babel/core").transformSync(
    `var b = 0b11;var o = 0o7;const u="Hello\\u{000A}\\u{0009}!";`
).code;

console.log(code)
```

 执行 `node index.js`，返回结果：

```js
var b = 0b11;
var o = 0o7;
const u = "Hello\u{000A}\u{0009}!";
```

原样返回，如果我要识别二进制整数、十六进制整数、`Unicode` 字符串文字、换行符和制表符，那么就需要加上 `@babel/plugin-transform-literals` 。加上之后执行结果如下：

```js
var b = 3;
var o = 7;
const u = "Hello\n\t!";
```

通过上述 `Demo` 了解到 `plugin` 的作用。

打开 `babel/packages`，我们可以看到 `plugins` 主要有三种类型：

![babel-plugin-type](https://files.mdnice.com/user/15734/7acfc546-c973-4d61-b7bd-af60cbfffee5.png)

1. **babel-plugin-transform-xx**：转换插件，主要用来加强转换能力，上面的 `@babel/plugin-transform-literals` 就属于这种；
2. **babel-plugin-syntax-xx**：语法插件，主要是扩展编译能力，比如不在 async 函数作用域里面使用 await，如果不引入 `@babel/plugin-syntax-top-level-await`，是没办法编译成 `AST` 树的。并且会报 `Unexpected reserved word 'await'` 这种类似的错误。
3. **babel-plugin-proposal-xx**：用来编译和转换在提案中的属性，在 [Plugins List](https://babeljs.io/docs/en/plugins-list "Plugins List") 中可以看到这些插件，比如 [class-properties](https://babeljs.io/docs/en/babel-plugin-proposal-class-properties "class-properties")、[decorators](https://babeljs.io/docs/en/babel-plugin-proposal-decorators "decorators")。

##  总结

从平时工作角度切入，一步一步分享 `babel7` 的最小、最优配置的由来，然后简单了解 `babel` 的 `packages`，分享了 `@babel/standalone` 这个有意思的包和插件系列的分类。下一篇文章将通过手写 `babel plugin` 去深入学习底层的 `packages`，例如 `@babel/core`、`@babel/parser`、`@babel/generator`、`@babel/code-frame`。

