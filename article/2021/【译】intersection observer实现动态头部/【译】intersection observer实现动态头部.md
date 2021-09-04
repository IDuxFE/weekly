# 【译】Intersection Observer 实现动态header

> 原文：[https://www.smashingmagazine.com/2021/07/dynamic-header-intersection-observer/](https://www.smashingmagazine.com/2021/07/dynamic-header-intersection-observer/)\
> 作者：[Michelle Barker](https://www.smashingmagazine.com/author/michelle-barker/)\
> 译者：[wongtsuizhen](https://juejin.cn/user/3069492196017053)

## 前言
你是否有过这样的一个UI需求：页面上的某些组件需要根据它们所在视口滚动到某个阈值，或者从所处视口进入或移出？在`JavaScript`中，可以通过监听滚动条事件进而在回调函数中来处理，但这会影响性能，如果使用不当，就可能引发卡顿的用户体验。但现在有个更好的实现方式-`Intersection Observer（交叉观察器）`来解决这个问题。

[Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)是一个JavaScript API，它可以让我们观察某一个元素，并检测它何时通过可滚动容器中的指定位置（通常(但不总是)在视口中），同时触发回调函数。

可以说`Intersection Observer`比在主线程上监听滚动事件性能更高，因为它是异步的，并且回调只会在我们监听的元素达到指定位置时才会启动，而不是每次滚动时就更新。在这篇文章中，我们将通过一个示例说明如何使用 `Intersection Observer` 构建一个固定的header，当其与网页的不同部分相交时会发生变化。

## 基本使用
使用`Intersection Observer`，我们首先需要创建一个observer，它有两个参数：`callback`是我们希望在监听的元素（称为目标元素）与根元素（视口，它必须是目标元素的祖先节点）相交时执行的回调函数； `options`配置对象（参数可选）。

```js
const options = {
  root: document.querySelector('[data-scroll-root]'),
  rootMargin: '0px',
  threshold: 1.0
}

const callback = (entries, observer) => {
  entries.forEach((entry) => console.log(entry))
}

const observer = new IntersectionObserver(callback, options)
```
当我们创建了观察者实例之后，我们还需要指示它去监视一个目标元素:

```js
const targetEl = document.querySelector('[data-target]')

observer.observe(targetEl)
```
options中的选项值都是可选的，他们有如下默认值：
```js
const options = {
  rootMargin: '0px',
  threshold: 1.0
}
```
### `rootMargin`
`rootMargin`值有点像向根元素添加margin, 与margin一样，可以接受多个值，包括负值。目标元素将被认为是相对于边界相交的。
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/df54da347b564f10908f565947d92cb2~tplv-k3u1fbpfcp-watermark.image)
> 具有正和负边距值的滚动根。假设默认阈值为1，橙色方块将被定位在“相交”的点上

这意味着一个元素在技术上可以被归类为“相交”，即使它不在视图中（如果滚动根是视口）；

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/58f0a3181ccb4fb5847cb55d31904715~tplv-k3u1fbpfcp-watermark.image)
> 橙色的正方形与根相交，即使它在可见区域之外。

`rootMargin`默认为`0px`，但可以接受由多个值组成的字符串，就像在CSS中使用`margin`属性一样。

### `threshold`

`threshold`可以由单个值或0到1之间的值数组组成。它表示**必须在根边界内才能被认为相交的比例**用默认值1，当根目录中100%的目标元素可见时，回调将触发。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f1e4531dcf30428783d45d6869fc4db5~tplv-k3u1fbpfcp-watermark.image)
> threshold为1、0和0.5分别导致在目标元素100%、0%和50%可见时触发回调。

当使用这些选项将元素分类为可见时并不总是容易的。所以我已经创建了一个[小工具](https://codepen.io/michellebarker/full/xxwLpRG)来帮助大家掌握`Intersection Observer`。

## 实现动态header
既然我们现在已经掌握了基本原理，让我们开始实现一个动态header。我们将从一个分成几个部分的网页开始。下面这张图片显示了我们将要创建的页面的完整布局:

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/426416c567c8452e9fb8f8d690391b81~tplv-k3u1fbpfcp-watermark.image)
我在本文的末尾展示了完整的一个demo，所以如果您想要直接查看代码，可以直接跳转到[demo](https://codepen.io/michellebarker/pen/QWpzwYN)。([Github仓库](https://github.com/mbarker84/smashing-io-header))

每个部分的最小高度为100vh（它们可能更长，取决于内容）。我们的标题是固定在页面的顶部，并在用户滚动时保持在原地（使用`position: fixed`）。各部分有不同颜色的背景，当它们遇到标题时，标题的颜色就会改变，以适配当前section的颜色。还有一个标记以显示用户所处的当前部分，当下一个部分到达时，它会滑动。如果你想跟着后面继续了解的话，我会通过讲解文章开头的小demo（[点击查看](https://codepen.io/michellebarker/pen/488760bfec40302fcf0e8b7cd3e6093e)）（在开始使用Intersection Observer API之前），来让我们更快而且直观的了解它的代码实现。

### 页面制作
我们将从header的HTML开始。这将是一个相当简单的页面，包含标题、链接、导航，它没有什么特别的，但我们要用几个数据属性：`data-header`来标记 header 本身（我们可以目标元素 JS），三个用户点击就滚动到对应模块的锚用`data-link`标记：
```html
<header data-header>
  <nav class="header__nav">
    <div class="header__left-content">
      <a href="#0">Home</a>
    </div>
    <ul class="header__list">
      <li>
        <a href="#about-us" data-link>About us</a>
      </li>
      <li>
        <a href="#flavours" data-link>The flavours</a>
      </li>
      <li>
        <a href="#get-in-touch" data-link>Get in touch</a>
      </li>
    </ul>
  </nav>
</header>
```
接下来是页面其余部分的HTML，我们把它们也分成几个部分。为了简洁起见，我只包含了与本文相关的部分，但是demo中包含了完整的内容。每个section都包含一个data 属性来指定背景颜色的名称，以及一个id，该id对应于标题中a标签的`href`:

```html
<main>
  <section data-section="raspberry" id="home">
    <!--Section content-->
  </section>
  <section data-section="mint" id="about-us">
    <!--Section content-->
  </section>
  <section data-section="vanilla" id="the-flavours">
    <!--Section content-->
  </section>
  <section data-section="chocolate" id="get-in-touch">
    <!--Section content-->
  </section>
</main>
```
我们将用CSS来定位header，这样当用户滚动时，它就会固定在页面顶部：

```css
header {
  position: fixed;
  width: 100%;
}
```
我们也会给我们每个secion设置一个最小的高度，并将内容居中。（这段代码并不是使用`Intersection Observer`所必需的，它只是样式的优化。）

```css
section {
  padding: 5rem 0;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

```
### IFRAME中的告警
在构建这个 Codepen 演示时，我遇到了一个令人困惑的问题，我的`Intersection Observer`代码本应完美地工作，但却未能在交集点正确触发回调，而是在目标元素与视口边缘相交时触发。经过一番思索之后，我意识到这是因为在Codepen中，内容是在iframe中加载的，它的处理方式是不同的。（详见MDN文档中关于[剪切和交集矩形的部分](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)）

作为一个解决方案，在演示中，我们可以将标记包装在另一个元素中，它将充当滚动容器（`Intersection Observeroptions`中的根元素root），而不是我们所期望的浏览器视区：

```html
<div class="scroller" data-scroller>
  <header data-header>
    <!--Header content-->
  </header>
  <main>
    <!--Sections-->
  </main>
</div>
```
如果您想要了解如何使用视口作为根，而不是demo中的根，可访问[Github仓库](https://github.com/mbarker84/smashing-io-header)。
### CSS样式
在我们的CSS中，我们将为我们使用的颜色定义一些自定义属性。我们还将为标题文本和背景颜色定义两个额外的自定义属性，并设置一些初始值。（我们将在后面更新这两个自定义属性）

```css
:root {
  --mint: #5ae8d5;
  --chocolate: #573e31;
  --raspberry: #f2308e;
  --vanilla: #faf2c8;
  
  --headerText: var(--vanilla);
  --headerBg: var(--raspberry);
}
```
我们将在header中使用这些自定义属性：

```css
header {
  background-color: var(--headerBg);
  color: var(--headerText);
}
```
我们还将为不同的部分设置颜色。我使用数据属性作为选择器，您也可以简单地使用类。

```css
[data-section="raspberry"] {
  background-color: var(--raspberry);
  color: var(--vanilla);
}

[data-section="mint"]  {
  background-color: var(--mint);
  color: var(--chocolate);
}

[data-section="vanilla"] {
  background-color: var(--vanilla);
  color: var(--chocolate);
}

[data-section="chocolate"] {
  background-color: var(--chocolate);
  color: var(--vanilla);
}

```
当每个section都在视图中时，我们也可以为header设置一些样式：
```css
/* Header */
[data-theme="raspberry"]  {
  --headerText: var(--raspberry);
  --headerBg: var(--vanilla);
}

[data-theme="mint"] {
  --headerText: var(--mint);
  --headerBg: var(--chocolate);
}

[data-theme="chocolate"]  {
  --headerText: var(--chocolate);
  --headerBg: var(--vanilla);
}

```
这里更需要使用数据属性，因为我们要在每次相交时切换header的`data-theme`属性。

## 创建Observer 
现在我们已经把页面基本的HTML和CSS都写好了，我们可以创建一个观察者来监视进入视图的每个部分。我们在当向下滚动页面，某个section与header底部接触时触发一个回调，这意味着我们需要设置一个负的`rootMargin`，以对应header的高度。

```js
const header = document.querySelector('[data-header]')
const sections = [...document.querySelectorAll('[data-section]')]
const scrollRoot = document.querySelector('[data-scroller]')

const options = {
  root: scrollRoot,
  rootMargin: `${header.offsetHeight * -1}px`,
  threshold: 0
}
```
我们设置了一个阈值为0，因为我们想让它在section的任何部分与根边界相交时触发。

首先，我们将创建一个回调来更改header的`data-theme`值（这比添加和删除类更简单，特别是当header元素可能应用了其他类时）。
```js
/* The callback that will fire on intersection */
const onIntersect = (entries) => {
  entries.forEach((entry) => {
    const theme = entry.target.dataset.section
    header.setAttribute('data-theme', theme)
  })
}
```
然后我们将创建观察者来监听相交的部分：
```js
/* Create the observer */
const observer = new IntersectionObserver(onIntersect, options)

/* Set our observer to observe each section */
sections.forEach((section) => {
  observer.observe(section)
})
```
现在我们应该看到我们的标题颜色更新时，每个部分满足标题。

[点击查看demo](https://codepen.io/smashingmag/embed/poPgpjZ?height=500&theme-id=light&slug-hash=poPgpjZ&user=smashingmag&default-tab=result)

然而，您可能会注意到，当我们向下滚动时，颜色没有正确地更新颜色。事实上，标题每次都在更新前一节的颜色。但向上滚动时，它工作得很完美。因此我们需要确定滚动方向并相应地改变它的行为。

### 寻找滚动方向
我们将在 JS 中设置一个变量，用于滚动的方向，初始值为`up`，另一个变量用于最后已知的滚动位置（`prevYPosition`）。然后，在回调函数中，如果滚动位置大于之前的值，我们可以设置方向值为`down`，反之为`up`。

```js
let direction = 'up'
let prevYPosition = 0

const setScrollDirection = () => {
  if (scrollRoot.scrollTop > prevYPosition) {
    direction = 'down'
  } else {
    direction = 'up'
  }

  prevYPosition = scrollRoot.scrollTop
}

const onIntersect = (entries, observer) => {
  entries.forEach((entry) => {
    setScrollDirection()
          
    /* ... */
  })
}
```
我们还将创建一个新函数来更新header颜色，将target部分作为参数传入：

```js
const updateColors = (target) => {
  const theme = target.dataset.section
  header.setAttribute('data-theme', theme)
}

const onIntersect = (entries) => {
  entries.forEach((entry) => {
    setScrollDirection()
    updateColors(entry.target)
  })
}

```
到目前为止，我们应该没有看到header的行为发生变化。但是现在我们知道了滚动方向，我们可以为`updateColors()`函数传入一个不同的目标。如果滚动方向是向上，我们将使用入口目标。如果它是向下的，我们将使用下一section（如果有）。

```js
const getTargetSection = (target) => {
  if (direction === 'up') return target
  
  if (target.nextElementSibling) {
    return target.nextElementSibling
  } else {
    return target
  }
}

const onIntersect = (entries) => {
  entries.forEach((entry) => {
    setScrollDirection()
    
    const target = getTargetSection(entry.target)
    updateColors(target)
  })
}
```
但是，还有一个问题：header不仅会在section到达header时更新，而且会在视图底部的下一个元素进入视图时更新。这是因为我们的观察者触发了两次回调：一次是在元素进入时，一次是在元素离开时。

为了确定header是否应该更新，我们可以使用`entry`对象中的`isIntersecting`键。让我们创建另一个函数来返回一个布尔值，用于判断header颜色是否应该更新：

```js
const shouldUpdate = (entry) => {
  if (direction === 'down' && !entry.isIntersecting) {
    return true
  }
  
  if (direction === 'up' && entry.isIntersecting) {
    return true
  }
  
  return false
}

```
我们将相应地更新`onIntersect()`函数：
```js
const onIntersect = (entries) => {
  entries.forEach((entry) => {
    setScrollDirection()
    
    /* Do nothing if no need to update */
    if (!shouldUpdate(entry)) return
    
    const target = getTargetSection(entry.target)
    updateColors(target)
  })
}
```
现在我们的颜色应该正确更新了。我们可以设置一个CSS过渡动画，这样效果会更好一些：

```css
header {
  transition: background-color 200ms, color 200ms;
}
```

[(点击查看demo)](//codepen.io/smashingmag/embed/bGWEaEa?height=500&theme-id=light&slug-hash=bGWEaEa&user=smashingmag&default-tab=result)

### 添加动态标记
接下来，我们将在标题中添加一个标记，当我们滚动到不同的部分时，它会更新它的位置。我们可以使用伪元素来实现，这样我们就不需要在HTML中添加任何东西。我们将给它一些简单的CSS样式，将它定位在header的左上角，并给它一个背景颜色。我们在这里使用`currentColor`，因为它将接受标题文本颜色的值：

```css
header::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  height: 0.4rem;
  background-color: currentColor;
}
```
我们可以为宽度使用自定义属性，默认值为0。我们还将为`transform x`值使用自定义属性。当用户滚动时，我们会在回调函数中设置这些值。

```css
header::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  height: 0.4rem;
  width: var(--markerWidth, 0);
  background-color: currentColor;
  transform: translate3d(var(--markerLeft, 0), 0, 0);
}
```
现在我们可以编写一个函数来更新标记在交点的宽度和位置：

```js
const updateMarker = (target) => {
  const id = target.id
  
  /* Do nothing if no target ID */
  if (!id) return
  
  /* Find the corresponding nav link, or use the first one */
  let link = headerLinks.find((el) => {
    return el.getAttribute('href') === `#${id}`
  })
  
  link = link || headerLinks[0]
  
  /* Get the values and set the custom properties */
  const distanceFromLeft = link.getBoundingClientRect().left
  
  header.style.setProperty('--markerWidth', `${link.clientWidth}px`)
  header.style.setProperty('--markerLeft', `${distanceFromLeft}px`)
}
```
我们可以在更新颜色的同时调用这个函数：

```js
const onIntersect = (entries) => {
  entries.forEach((entry) => {
    setScrollDirection()
    
    if (!shouldUpdate(entry)) return
    
    const target = getTargetSection(entry.target)
    updateColors(target)
    updateMarker(target)
  })
}
```
我们还需要为标记设置一个初始位置，这样它就不会突然出现。当文档被加载时，我们将调用`updateMarker()`函数，使用第一部分作为目标：

```js
document.addEventListener('readystatechange', e => {
  if (e.target.readyState === 'complete') {
    updateMarker(sections[0])
  }
})

```
最后，让我们添加一个CSS过渡，以便标记从一个链接滑过标题到下一个链接。当我们转换宽度属性时，我们可以使用will-change来让[浏览器执行优化](https://developer.mozilla.org/en-US/docs/Web/CSS/will-change)。
```css
header::after {
  transition: transform 250ms, width 200ms, background-color 200ms;
  will-change: width;
}
```
### 平滑滚动
最后，如果当用户点击链接时，他们能平稳地向下滚动页面，而不是跳转到该部分，那就再好不过了。目前我们直接通过CSS来做，不需要JS！为了获得更好的体验，那么最好尊重用户的设置偏好，如果用户在系统设置中没有开启动画减弱功能，才使用平滑滚动：

```css
@media (prefers-reduced-motion: no-preference) {
  .scroller {
    scroll-behavior: smooth;
  }
}

```
### 完整demo
将上述所有步骤放在一起，就得到了完整的demo。
[完整demo查看](//codepen.io/smashingmag/embed/XWRXVXQ?height=500&theme-id=light&slug-hash=XWRXVXQ&user=smashingmag&default-tab=result)

## 浏览器支持
`Intersection Observer`在现代浏览器中得到广泛支持（[浏览器支持](https://caniuse.com/?search=intersection%20observer)）。在需要的地方，它可以为旧的浏览器填充——但我更喜欢采取渐进的增强方法。在我们的header例子中，为不支持的浏览器提供一个简单的、不变的版本不会对用户体验造成很大的损害。

要检测是否支持`Intersection Observer`，我们可以使用以下方法：
```js
if ('IntersectionObserver' in window && 'IntersectionObserverEntry' in window && 'intersectionRatio' in window.IntersectionObserverEntry.prototype) {
  /* Code to execute if IO is supported */
} else {
  /* Code to execute if not supported */
}
```

## 资源
阅读更多关于`Intersection Observer API`的文档：
- [MDN API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)
- [`Intersection Observer`可视化工具](https://codepen.io/michellebarker/full/xxwLpRG)
- [Timing Element Visibility with the Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API/Timing_element_visibility) MDN的另一个教程，看看IO如何用来跟踪广告的可见性
- [Now You See Me: How To Defer, Lazy-Load And Act With IntersectionObserver](https://www.smashingmagazine.com/2018/01/deferring-lazy-loading-intersection-observer-api/) - Denys Mishunov撰写的这篇文章涵盖了IO的其他一些用途，包括资源懒加载。虽然现在没有那么必要了（多亏了loading属性），但这里仍然有很多东西需要学习。

