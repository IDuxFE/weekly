# Say Hello To CSS Container Queries

# 【译】初识 CSS 容器查询

> 原文：[https://ishadeed.com/article/say-hello-to-css-container-queries/](https://ishadeed.com/article/say-hello-to-css-container-queries/)\
> 作者：[Ahmad Shadeed](https://ishadeed.com/hire-me/)\
> 译者：[Levix](https://juejin.cn/user/3650034331558717)

在我过去六年的前端开发生涯里，我从没像现在这样对一个 CSS 功能感到如此兴奋。容器查询功能现在可以在 Chrome Canary 中的 flag（chrome://flags） 配置使用，感谢 [Miriam Suzanne](https://css.oddbird.net/rwd/query/contain/) 以及其他朋友的努力。

我记得看到过很多有关于 CSS 容器查询的调侃，但最终它出现了。在本篇文章中，我将带你了解**为什么**我们需要容器查询，它如何让你的生活（CSS 编码）变得更轻松，最重要的是你可以用它实现更强大的组件和布局。

如果你跟我一样为之兴奋，让我们开始吧。你准备好了吗？

## CSS 媒体查询的问题

一个网页由不同的内容和组件组成，我们可以使用 CSS 媒体查询让它具备响应性，这看起来没毛病，但它存在一定的局限性。例如，我们可以使用媒体查询在移动设备以及桌面设备上显示一个组件的最小化版本。

通常，响应式网页设计不是基于视口（viewport）或者屏幕尺寸，而是基于容器尺寸，参考下面的例子：

![img](https://ishadeed.com/assets/cq/problem-1.jpg)

我们有一个非常经典的卡片组件布局，它存在两种布局方式：

- 堆叠的（见上图左侧）
- 横向的（见上图主区域）

CSS 有多种方式可以实现上述布局，但最常见的方式如下代码所示，我们需要创建一个**基础**组件，基于它做一些适配。

```css
.c-article {
  /* 默认状态，堆叠版本 */
}

.c-article > * + * {
  margin-top: 1rem;
}

/* 水平版本 */
@media (min-width: 46rem) {
  .c-article--horizontal {
    display: flex;
    flex-wrap: wrap;
  }

  .c-article > * + * {
    margin-top: 0;
  }

  .c-article__thumb {
    margin-right: 1rem;
  }
}
```

注意，我们创建了 `.c-article--horizontal` 类来处理该组件的水平布局，当视口宽度大于 **46rem** 时，则该组件会切换为水平布局。

这看起来没什么问题，但我觉得这样处理存在其局限性，我希望的组件响应式是基于其父容器宽度，而不是浏览器视口或者屏幕尺寸。

考虑到我们希望在主要区域使用了默认的 `.c-card` 类，将会发生什么呢？卡片将扩展到其父容器的宽度，因此整体布局看起来会比较大，见下图：

![img](https://ishadeed.com/assets/cq/problem-1-2.jpg)

我们可以使用 CSS 容器查询解决这个问题（是的，这是终极方案），在深入研究区它们之前，让我带你先了解一下我们想要的结果。

![img](https://ishadeed.com/assets/cq/cq-story.png)

我们需要告诉该组件如果它的直属父级容器宽度大于 400px，就需要切换为水平样式，以下是 CSS 代码配置：

```html
<div class="o-grid">
  <div class="o-grid__item">
    <article class="c-article">
      <!-- 内容 -->
    </article>
  </div>
  <div class="o-grid__item">
    <article class="c-article">
      <!-- 内容 -->
    </article>
  </div>
</div>
.o-grid__item {
  contain: layout inline-size;
}

.c-article {
  /* 默认样式 */
}

@container (min-width: 400px) {
  .c-article {
    /* 使文章水平而非卡片式的样式 */
  }
}
```

## CSS 容器查询将如何帮助我们

> 警告：CSS 容器查询目前只在 Chrome Canary 浏览器的实验标志下支持。

通过 **CSS 容器查询**，我们可以解决上述问题，并制作一个丝滑的组件。这意味着，我们可以把组件放到一个狭窄的父容器中，它将会变成堆叠的版本，或者放到宽大的父容器中，它会自适应为水平版本，而这些自适应变化都独立于视口宽度。

以下是我的设想。

![img](https://ishadeed.com/assets/cq/intro.jpg)

紫色的轮廓代表父容器的宽度，注意当它宽度变大时，该组件是如何适配它？这就是 CSS 容器查询的强大之处。

## 容器查询是如何工作的

我们现在在 Chrome canary 上尝试使用容器查询，请到 `chrome://flags` 回车后搜索“container queries”，启用它（译者注：Chrome 94 版本开启后也能正常使用）。

第一步是添加 `contain` 属性。由于组件将根据它的父容器宽度适配，我们需要告知浏览器只对受影响的区域重绘，而不是整个页面，有了 `contain` 属性，我们可以让浏览器提前知道这一点。

`inline-size` 值意味着只对父容器的宽度变化进行适配，我尝试使用了 `block-size` ，但发现它无法满足需求，如果我存在错误请纠正我。

```html
<div class="o-grid">
  <div class="o-grid__item">
    <article class="c-article">
      <!-- 内容 -->
    </article>
  </div>
  <div class="o-grid__item">
    <article class="c-article">
      <!-- 内容 -->
    </article>
  </div>
  <!-- 其他文章.. -->
</div>
.o-grid__item {
  contain: layout inline-size;
}
```

这是第一步，我们定义了 `.o-grid__item` 元素作为 `.c-article` 的父容器。

下一步我们添加需要的样式，让容器查询功能生效。

```css
.o-grid__item {
  contain: layout inline-size;
}

@container (min-width: 400px) {
  .c-article {
    display: flex;
    flex-wrap: wrap;
  }

  /* 其他 CSS.. */
}
```

`@container` 指向的是 `.o-grid__item` 元素，其中 `min-width: 400px` 是它的宽度（适配），我们甚至可以更进一步添加更多的样式。下面的视频介绍了卡片组件可以实现的功能（需要梯子）：

<video src="https://ishadeed.com/assets/cq/cq-1.mp4" controls="" style="box-sizing: inherit; margin-bottom: 24px; max-width: 100%; color: rgb(85, 85, 85); font-family: Rubik, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial; width: initial;"></video>

我们在视频中的样式配置有：

1. 默认情况下，是卡片式的外观。
2. 一张带有小缩略图的横向卡片。
3. 一张带有大缩略图的横向卡片。
4. 如果父容器太大，将显示一个主图的样式，以表明它是一篇有**特色**的文章。

让我们来探讨一下 CSS 容器查询的使用案例。

## CSS 容器查询的使用案例

### 容器查询和 CSS Grid `auto-fit`

在某些情况下，在 CSS grid 布局中使用 `auto-fit` 会出现意想不到的结果，例如，组件太宽，其内容难以阅读。

为了更直观了解问题现象，这里有一个视觉效果，它展示了 CSS grid 布局中 `auto-fit` 和 `auto-fill` 的区别。

![img](https://ishadeed.com/assets/cq/auto-fit-fill.png)

注意，当使用 `auto-fit` 时，子项会自适应以覆盖可用空间，然而，在 `auto-fill` 例子里， 网格子项不会延伸，我们将会有一个自由空间（最右边虚线部分）。

你现在可能会很困惑，这跟 CSS 容器查询有什么关联？每个网格子项都是一个容器，当它展开时（也就是我们使用的 `auto-fit`），组件需要在此基础上改变。

```html
<div class="o-grid">
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
  <div class="o-grid__item">
    <article class="c-article"></article>
  </div>
</div>
.o-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-gap: 1rem;
}
```

当我们有 4 个元素，大致看起来像下面那样。

![img](https://ishadeed.com/assets/cq/use-case-1-1.png)

当文章数量减少时，此时会发生变化，下图展示发生了什么。我们的文章越少，它们就会变得越宽。原因是使用了 `auto-fit` ，第一个看起来还不错，但最后两个（每行 2 个，每行 1 个）看起来就不怎么样了，因为它们太宽了。

![img](https://ishadeed.com/assets/cq/use-case-1-2.png)

如果每个文章组件都根据父容器宽度改变布局呢？这样的话，我们就可以充分享受 `auto-fit` 带来的好处。以下是我们需要做的：

> 如果网格子项的宽度大于 400px 时，那么文章应该切换到水平风格。

我们可以这样做：

```css
.o-grid__item {
  contain: layout inline-size;
}

@container (min-width: 400px) {
  .c-article {
    display: flex;
    flex-wrap: wrap;
  }
}
```

另外，如果文章是网格中唯一一项，我们想用主图来展示。

```css
.o-grid__item {
  contain: layout inline-size;
}

@container (min-width: 700px) {
  .c-article {
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 350px;
  }

  .card__thumb {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    object-fit: cover;
  }
}
```

![img](https://ishadeed.com/assets/cq/use-case-1-3.png)

以上，我们有了一个自适应其父级宽度的组件，而且它可以在任何环境下工作，这不是很好吗？

查看 CodePen 上的[演示](https://codepen.io/shadeed/pen/ExZEEjZ?editors=0100)。

### 侧边栏和主体

通常，我们需要调整适配一个组件，能够让它在宽度较小的容器中展示，如 `<aside>` 。

一个经典案例是通讯部分，当它宽度较小时，我们需要将其堆叠展示，而当宽度足够时，我们需要它能够水平展开。

![img](https://ishadeed.com/assets/cq/use-case-2.jpg)

正如你上图所看到的，我们有一个适配两种不同场景下的时事通讯组件：

- 侧边栏部分
- 主要区域

如果没有容器查询，估计不太可能实现，除非我们在 CSS 中增加一个变体类，例如， `.newsletter--stacked` 之类的。

我知道我们可以[强制包裹这些子项](https://css-tricks.com/useful-flexbox-technique-alignment-shifting-wrapping/)，防止弹性布局没有足够的空间，但还是不够的。我需要更多的掌控力来做以下事情：

- 隐藏特定元素。
- 按钮撑满容器。

```css
.newsletter-wrapper {
  contain: layout inline-size;
}

/* 默认样式, 堆叠版本 */
.newsletter {
  /* CSS 样式 */
}

.newsletter__title {
  font-size: 1rem;
}

.newsletter__desc {
  display: none;
}

/* 水平版本 */
@container (min-width: 600px) {
  .newsletter {
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .newsletter__title {
    font-size: 1.5rem;
  }

  .newsletter__desc {
    display: block;
  }
}
```

这里有个视频展示。

<video src="https://ishadeed.com/assets/cq/use-case-newsletter.mp4" controls="" style="box-sizing: inherit; margin-bottom: 24px; max-width: 100%; color: rgb(85, 85, 85); font-family: Rubik, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial; width: initial;"></video>

查看 CodePen 上的[演示](https://codepen.io/shadeed/pen/poRLxvO?editors=0100)。

### 分页

我发现分页场景非常适合使用容器查询，最初，我们可以有“上一页”和“下一页”按钮，我们可以隐藏它们，并在有足够空间的情况下显示全部分页。

参考下图。

![img](https://ishadeed.com/assets/cq/use-case-3.jpg)

要处理上述图中状态，我们需要首先处理默认样式(堆叠按钮)，接着处理其余两种状态。

```css
.wrapper {
  contain: layout inline-size;
}

@container (min-width: 250px) {
  .pagination {
    display: flex;
    flex-wrap: wrap;
    gap: 0.5rem;
  }

  .pagination li:not(:last-child) {
    margin-bottom: 0;
  }
}

@container (min-width: 500px) {
  .pagination {
    justify-content: center;
  }

  .pagination__item:not(.btn) {
    display: block;
  }

  .pagination__item.btn {
    display: none;
  }
}
```

查看 CodePen 上的[演示](https://codepen.io/shadeed/pen/VwPQORy?editors=0100)。

### 个人资料卡片

![img](https://ishadeed.com/assets/cq/use-case-4.jpg)

这是另一个适合在多个场景中使用的案例，小状态适用于小的视口大小以及像侧边栏那样的场景，而更大的状态可以在更大的环境下工作，比如把它放在 2-col 网格中。

```css
.p-card-wrapper {
  contain: layout inline-size;
}

.p-card {
  /* 默认样式 */
}

@container (min-width: 450px) {
  .meta {
    display: flex;
    justify-content: center;
    gap: 2rem;
    border-top: 1px solid #e8e8e8;
    background-color: #f9f9f9;
    padding: 1.5rem 1rem;
    margin: 1rem -1rem -1rem;
  }

  /* 其他样式 */
}
```

这样，我们可以看到在不使用单一媒体查询的情况下，component words 在不同场景下是如何使用的。

![img](https://ishadeed.com/assets/cq/use-case-4-in-action.jpg)

查看 CodePen 上的[演示](https://codepen.io/shadeed/pen/NWdYaGY?editors=0100)。

### 表单元素

我还没有深入研究表单的案例，但我想到的是将标签从水平状态切换到堆叠状态。

![img](https://ishadeed.com/assets/cq/use-case-5.jpg)

```css
.form-item {
  contain: layout inline-size;
}

.input-group {
  @container (min-width: 350px) {
    display: flex;
    align-items: center;
    gap: 1.5rem;

    input {
      flex: 1;
    }
  }
}
```

在下面的演示中自己尝试一下。在 CodePen 上查看[演示](https://codepen.io/shadeed/pen/XWpEEzR?editors=0100)。

## 测试组件

现在我们已经尝试了在几个案例中使用 CSS 容器查询，那我们如何测试组件（快速验证）？值得庆幸的是，我们可以通过组件的父容器 CSS `resize` 属性来做到这一点。

![img](https://ishadeed.com/assets/cq/cq-resize.jpg)

```css
.parent {
  contain: layout inline-size;
  resize: horizontal;
  overflow: auto;
}
```

我从 Bramus Van Damme 的[文章](https://www.bram.us/2020/05/15/css-only-resizable-elements/)中学到了这个技巧。

## 在 DevTools 中调试容器查询容易吗

暂时不能，你无法看到像 `@container (min-width: value)` 这样的东西，但我认为这只是时间问题，最终会被支持。

## 是否有可能提供备选方案

是的！当然可以，在某些方面提供备选方案是可能的，这里有两篇很棒的文章解释了如何做到这一点:

- [Container Query Solutions with CSS Grid and Flexbox](https://moderncss.dev/container-query-solutions-with-css-grid-and-flexbox/) 作者 Stephanie Eckles
- [Container Queries are actually coming](https://piccalil.li/blog/container-queries-are-actually-coming) 作者 Andy Bell

## 总结

我很喜欢学习 CSS 容器查询并在浏览器中使用它。我知道它还没有被官方支持，但现在是在浏览器中使用它的好时机。

作为前端开发人员，我们工作的一部分就是给那些致力于实现这些新功能的人提供测试和帮助，我们测试的越多，一旦所有主流浏览器都支持该功能，我们看到的问题就越少。

感谢您的阅读。
