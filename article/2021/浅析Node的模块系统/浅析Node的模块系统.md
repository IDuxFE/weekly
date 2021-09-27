
## 浅析 Node 的模块系统

### 模块化的背景

早期 JavaScript 是为了实现简单的页面交互逻辑, 但随着时代发展, 浏览器不单单仅只能呈现简单交互, 各种各样的网站开始大放光彩。随着网站开始变得复杂化，前端代码日渐增多，相对比起其他静态语言，JavaScript 缺少模块化的弊端开始暴露出来，如命名冲突。因此为了方便前端代码的维护和管理，社区开始了对模块化规范的定义。在这过程中，出现了很多的模块化规范，如`CommonJS`, `AMD`, `CMD`, `ES modules`，本文章主要讲解`Node`中根据`CommonJS`实现的模块化。

### CommonJS 规范

首先，在 Node 世界里,模块系统是遵守`CommonJS`规范的，`CommonJS`规范里定义，简单讲就是：
  - 每一个文件就是一个模块
  - 通过`module`对象来代表一个模块的信息
  - 通过`exports`用来导出模块对外暴露的信息
  - 通过`require`来引用一个模块

### Node 模块分类
  - 核心模块: 如 fs,http,path 等模块, 这些模块不需要安装, 在运行时已经加载在内存中
  - 第三方模块: 通过安装存放在 node_modules 中
  - 自定义模块: 主要是指 file 模块,通过绝对路径或者相对路径进行引入

### Module 对象

我们上面说过,一个文件就是一个模块,且通过一个 module 对象来描述当前模块信息,一个 module 对象对应有以下属性：
    - id: 当前模块的id
    - path: 当前模块对应的路径
    - exports: 当前模块对外暴露的变量
    - parent: 也是一个module对象,表示当前模块的父模块,即调用当前模块的模块
    - filename: 当前模块的文件名(绝对路径), 可用于在模块引入时将加载的模块加入到全局模块缓存中, 后续引入直接从缓存里进行取值
    - loaded: 表示当前模块是否加载完毕
    - children: 是一个数组,存放着当前模块调用的模块
    - paths: 是一个数组,记录着从当前模块开始查找node_modules目录,递归向上查找到根目录下的node_modules目录下


### module.exports 与 exports

说完`CommonJS`规范,我们先讲下`module.exports`与`exports`的区别。

首先，我们用个新模块进行一个简单验证

```javascript
console.log(module.exports === exports); // true
```

可以发现，`module.exports`和`epxorts`实际上就是指向同一个引用变量。

#### demo1
```javascript
// a模块
module.exports.text = 'xxx';
exports.value = 2;

// b模块代码
let a = require('./a');

console.log(a); // {text: 'xxx', value: 2}
```

从而也就验证了上面`demo1`中,为什么通过`module.exports`和`exports`添加属性,在模块引入时两者都存在, 因为两者最终都是往同一个引用变量上面进行属性的添加.根据该 demo, 可以得出结论: `module.exports`和`exports`指向同一个引用变量

#### demo2
```javascript
// a模块
module.exports = {
  text: 'xxx'
}
exports.value = 2;

// b模块代码
let a = require('./a');

console.log(a); // {text: 'xxx'}
```

上面的 demo 例子中,对`module.exports`进行了重新赋值, `exports`进行了属性的新增, 但是在引入模块后最终导出的是`module.exports`定义的值, 可以得出结论: noed 的模块最终导出的是`module.exports`, 而`exports`仅仅是对 module.exports 的引用, 类似于以下代码:
```javascript
exports = module.exports = {};
(function (exports, module) {
  // a模块里面的代码
  module.exports = {
    text: 'xxx'
  }
  exports.value = 2;

  console.log(module.exports === exports); // false
})(exports, module)
```
由于在函数执行中, exports 仅是对原`module.exports`对应变量的一个引用,当对`module.exports`进行赋值时,`exports`对应的变量和最新的`module.exports`并不是同一个变量

### require 方法
`require`引入模块的过程主要分为以下几步:
- 解析文件路径成绝对路径
- 查看当前需要加载的模块是否已经有缓存, 如果有缓存, 则直接使用缓存的即可
- 查看是否是 node 自带模块, 如 http,fs 等, 是就直接返回
- 根据文件路径创建一个模块对象
- 将该模块加入模块缓存中
- 通过对应的文件解析方式对文件进行解析编译执行(node 默认仅支持解析`.js`,`.json`, `.node`后缀的文件)
- 返回加载后的模块 exports 对象

![require执行过程](https://files.mdnice.com/user/19436/548b9231-3e2b-4371-b612-39f3a00cc2bd.jpg)

```javascript
Module.prototype.require = function(id) {
  // ...
  try {
    // 主要通过Module的静态方法_load加载模块 
    return Module._load(id, this, /* isMain */ false);
  } finally {}
  // ...
};
// ...
Module._load = function(request, parent, isMain) {
  let relResolveCacheIdentifier;
  // ...
  // 解析文件路径成绝对路径
  const filename = Module._resolveFilename(request, parent, isMain);
  // 查看当前需要加载的模块是否已经有缓存
  const cachedModule = Module._cache[filename];
  // 如果有缓存, 则直接使用缓存的即可
  if (cachedModule !== undefined) {
    // ...
    return cachedModule.exports;
  }

  // 查看是否是node自带模块, 如http,fs等, 是就直接返回
  const mod = loadNativeModule(filename, request);
  if (mod && mod.canBeRequiredByUsers) return mod.exports;

  // 根据文件路径初始化一个模块
  const module = cachedModule || new Module(filename, parent);

  // ...
  // 将该模块加入模块缓存中
  Module._cache[filename] = module;
  if (parent !== undefined) {
    relativeResolveCache[relResolveCacheIdentifier] = filename;
  }

  // ...
  // 进行模块的加载
  module.load(filename);

  return module.exports;
};
```

至此, node 的模块原理流程基本过完了。目前 node v13.2.0 版本起已经正式支持 ESM 特性。

### __filename, __dirname

在接触 node 中,你是否会困惑 `__filename`, `__dirname`是从哪里来的, 为什么会有这些变量呢? 仔细阅读该章节,你会对这些有系统性的了解。

- 顺着上面的 require 源码继续走, 当一个模块加载时, 会对模块内容读取
- 将内容包裹成函数体
- 将拼接的函数字符串编译成函数
- 执行编译后的函数, 传入对应的参数

```javascript
Module.prototype._compile = function(content, filename) {
  // ...
  const compiledWrapper = wrapSafe(filename, content, this);
  // 
  result = compiledWrapper.call(thisValue, exports, require, module,
                                    filename, dirname);
  
  // ...
  return result;
};
```
```javascript
function wrapSafe(filename, content, cjsModuleInstance) {
  // ...
  const wrapper = Module.wrap(content);
  // ...
}
let wrap = function(script) {
  return Module.wrapper[0] + script + Module.wrapper[1];
};

const wrapper = [
  '(function (exports, require, module, __filename, __dirname) { ',
  '\n});'
];

ObjectDefineProperty(Module, 'wrap', {
  get() {
    return wrap;
  },

  set(value) {
    patched = true;
    wrap = value;
  }
});
```

综上, 也就是之所以模块里面有`__dirname`,`__filename`, `module`, `exports`, `require`这些变量, 其实也就是 node 在执行过程传入的, 看完是否解决了多年困惑的问题^_^

### NodeJS 中使用 ES Modules

- 在`package.json`增加"type": "module"配置

```javascript
// test.mjs
export default {
	a: 'xxx'
}
// import.js
import a from './test.mjs';

console.log(a); // {a: 'xxx'}
```

### import 与 require 两种机制的区别

较明显的区别是在于执行时机:

- ES 模块在执行时会将所有`import`导入的模块会先进行预解析处理, 先于模块内的其他模块执行
```javascript
// entry.js
console.log('execute entry');
let a = require('./a.js')

console.log(a);

// a.js
console.log('-----a--------');

module.exports = 'this is a';
// 最终输出顺序为:
// execute entry
// -----a--------
// this is a
```
```javascript
// entry.js
console.log('execute entry');
import b from './b.mjs';

console.log(b);

// b.mjs
console.log('-----b--------');

export default 'this is b';
// 最终输出顺序为:
// -----b--------
// execute entry
// this is b
```

- import 只能在模块的顶层,不能在代码块之中(比如在`if`代码块中),如果需要动态引入, 需要使用`import()`动态加载;

ES 模块对比 CommonJS 模块, 还有以下的区别:

- 没有 `require`、`exports` 或 `module.exports`

  在大多数情况下，可以使用 ES 模块 import 加载 CommonJS 模块。(CommonJS 模块文件后缀为 cjs)
  如果需要引入`.js`后缀的 CommonJS 模块, 可以使用`module.createRequire()`在 ES 模块中构造`require`函数
  
```javascript
// test.cjs
export default {
a: 'xxx'
}
// import.js
import a from './test.cjs';

console.log(a); // {a: 'xxx'}
```

```javascript
// test.cjs
export default {
a: 'xxx'
}
// import.js
import a from './test.cjs';

console.log(a); // {a: 'xxx'}
```

```javascript
// test.cjs
export default {
a: 'xxx'
}
// import.mjs
import { createRequire } from 'module';
const require = createRequire(import.meta.url);

// test.js 是 CommonJS 模块。
const siblingModule = require('./test');
console.log(siblingModule); // {a: 'xxx'}
```
  
- 没有 __filename 或 __dirname

  这些 CommonJS 变量在 ES 模块中不可用。

- 没有 JSON 模块加载

  JSON 导入仍处于实验阶段，仅通过 --experimental-json-modules 标志支持。

- 没有 require.resolve
- 没有 NODE_PATH
- 没有 require.extensions
- 没有 require.cache

### ES 模块和 CommonJS 的相互引用

#### 在 CommonJS 中引入 ES 模块

由于 ES Modules 的加载、解析和执行都是异步的，而 require() 的过程是同步的、所以不能通过 require() 来引用一个 ES6 模块。

ES6 提议的 import() 函数将会返回一个 Promise，它在 ES Modules 加载后标记完成。借助于此，我们可以在 CommonJS 中使用异步的方式导入 ES Modules：

```javascript
// b.mjs
export default 'esm b'
// entry.js
(async () => {
	let { default: b } = await import('./b.mjs');
	console.log(b); // esm b
})()
```

#### 在 ES 模块中引入 CommonJS

在 ES6 模块里可以很方便地使用 import 来引用一个 CommonJS 模块，因为在 ES6 模块里异步加载并非是必须的：

```javascript
// a.cjs
module.exports = 'commonjs a';
// entry.js
import a from './a.cjs';

console.log(a); // commonjs a
```

至此,提供 2 个 demo 给大家测试下上述知识点是否已经掌握,如果没有掌握可以回头再进行阅读。

#### demo module.exports&exports
```javascript
// a模块
exports.value = 2;

// b模块代码
let a = require('./a');

console.log(a); // {value: 2}
```

#### demo module.exports&exports
```javascript
// a模块
exports = 2;

// b模块代码
let a = require('./a');

console.log(a); // {}
```
------
#### require&_cache 模块缓存机制
```javascript
// origin.js
let count = 0;

exports.addCount = function () {
	count++
}

exports.getCount = function () {
	return count;
}

// b.js
let { getCount } = require('./origin');
exports.getCount = getCount;

// a.js
let { addCount, getCount: getValue } = require('./origin');
addCount();
console.log(getValue()); // 1
let { getCount } = require('./b');
console.log(getCount()); // 1
```

### require.cache

根据上述例子, 模块在 require 引入时会加入缓存对象`require.cache`中。 如果需要删除缓存， 可以考虑将该缓存内容清除，则下次`require`模块将会重新加载模块。

```javascript
let count = 0;

exports.addCount = function () {
	count++
}

exports.getCount = function () {
	return count;
}

// b.js
let { getCount } = require('./origin');
exports.getCount = getCount;

// a.js
let { addCount, getCount: getValue } = require('./origin');
addCount();
console.log(getValue()); // 1
delete require.cache[require.resolve('./origin')];
let { getCount } = require('./b');
console.log(getCount()); // 0
```

### 结语

至此，本文主要介绍了 Node 中基于`CommonJS`实现的模块化机制，并且通过源码的方式对模块化的整个流程进行了分析，有关于模块的介绍可查看下面参考资料。有疑问的欢迎评论区留言，谢谢。

### 参考资料

[CommonJS 模块](https://nodejs.org/dist/latest-v16.x/docs/api/modules.html)

[ES 模块](https://nodejs.org/dist/latest-v16.x/docs/api/esm.html)
