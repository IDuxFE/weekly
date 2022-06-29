在谈论Shadow DOM，先聊聊在平常浏览各个网站过程中，经常遇到的一种现象：页面广告。
如图：

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/506bd0431c154cd4a33e5e4fd3e6f4be~tplv-k3u1fbpfcp-watermark.image?)

这种广告按照来源可分为两种：
1.访问站点、站点自身的广告；
2.运营商劫持流量，从而插入的广告；
从开发者的角度来看，对于第一种类型、站点自身的广告，开发起来思路很明确，无需赘述；而对于类似第二种场景的开发者，需要在对代理流量的目标站点里，插入开发者想要的元素，并且与此同时要保证插入的代码与原站点之间的影响降至最低。

因此，需要一种有效的“隔离”手段。

iframe似乎是一个不错的选择。但是这似乎还不够：
1.iframe需要一个明确的src资源链接，而有时候我们似乎没必要再单独为其去发布一个资源；
2.iframe并未实现完全的“隔离”，原有站点还是能拿到iframe节点，并可对其进行DOM操作；

[web component](https://zhuanlan.zhihu.com/p/340087848) 应运而生。
> Web components 的一个重要属性是封装——可以将标记结构、样式和行为隐藏起来，并与页面上的其他代码相隔离，保证不同的部分不会混在一起，可使代码更加干净、整洁。其中，Shadow DOM 接口是关键所在，它可以将一个隐藏的、独立的 DOM 附加到一个元素上。

与iframe相比，Shadow DOM 不需要资源链接，在以后web component飞速发展的日子里，作用将越来越大。

### Shadow DOM在Document tree里的体现

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4fcf7d81d7904ba68bab1fa52e1c4004~tplv-k3u1fbpfcp-watermark.image?)
-   Shadow host：一个常规 DOM 节点，Shadow DOM 会被附加到这个节点上。
-   Shadow tree：Shadow DOM 内部的 DOM 树。
-   Shadow boundary：Shadow DOM 结束的地方，也是常规 DOM 开始的地方。
-   Shadow root: Shadow tree 的根节点。

### 兼容性

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cd930239e1df4a08ae3c63c6757da4c8~tplv-k3u1fbpfcp-watermark.image?)

### 基本用法

可以使用 `Element.attachShadow()` 方法来将一个 shadow root 附加到任何一个元素上。它接受一个配置对象作为参数，该对象有一个 `mode` 属性，值可以是 `open` 或者 `closed`：

```js
let shadow = elementRef.attachShadow({mode: 'open'});
let shadow = elementRef.attachShadow({mode: 'closed'});
```


`open` 表示可以通过页面内的 JavaScript 方法来获取 Shadow DOM，例如使用 [`Element.shadowRoot`](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/shadowRoot) 属性：

```js
let myShadowDom = myCustomElem.shadowRoot;
```

如果你将一个 Shadow root 附加到一个 Custom element 上，并且将 `mode` 设置为 `closed`，那么就不可以从外部获取 Shadow DOM 了——`myCustomElem.shadowRoot` 将会返回 `null`。浏览器中的某些内置元素就是如此，例如`<video>`，包含了不可访问的 Shadow DOM。

### 使用外部引入的样式
```js
// 将外部引用的样式添加到 Shadow DOM 上
const linkElem = document.createElement('link');
linkElem.setAttribute('rel', 'stylesheet');
linkElem.setAttribute('href', 'style.css');

// 将所创建的元素添加到 Shadow DOM 上

shadow.appendChild(linkElem);
```

### 实际应用

```js
// 创建顶层 document 环境的 dom 节点
function createContainerInDocument () {
     const container = document.createElement('section');
     container.classList.add('shadow-container');
     container.style.position = 'fixed';
     container.style.zIndex = 2147483647; // 举个例子，css可根据实际情况自行添加
     return container;
}
// 创建 shadowRoot 环境下的 container 节点（用于挂在 Vue 示例）
function containerInShadowRoot () {
    const div = document.createElement('div');
    div.id = 'app';
    return div;
}
// DOM 创建流程：步骤 ① - ③
function createDom () {
    const container = createContainerInDocument();
    const shadowRoot = container.attachShadow({ mode: 'close' });
    const containerInShadow = containerInShadowRoot(shadowRoot);
    shadowRoot.appendChild(containerInShadow);
    document.body.appendChild(container);
    loadCss(shadowRoot);
    return containerInShadow;
}

// 加载 css 样式
function loadCss (root) {
    const link = document.createElement('link');
    let version = '';
    link.rel = 'stylesheet';
    link.href = `${process.env.BASE_URL}css/app.css?v=${version}`;// 加载 css 样式文件，路径自行修改
    root.appendChild(link);
}

let rootDom = createDom(); // rootDom即可用来作为挂载相关组件的根节点
```

这样，我们便可以在目标站点中插入想要的自定义组件了，并且能最大幅度的减少原有站点的影响。