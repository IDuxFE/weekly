# CSS Container Query Units

# 【译】CSS 容器查询单位

> 原文：[https://ishadeed.com/article/container-query-units/](https://ishadeed.com/article/container-query-units/)\
> 作者：[Ahmad Shadeed](https://ishadeed.com/hire-me/)\
> 译者：[Levix](https://juejin.cn/user/3650034331558717)

几天前，我看到 Miriam Suzanne 关于支持 CSS 容器查询单位的的一条[推特](https://twitter.com/TerribleMia/status/1436087037040414720)，这最初是由 Github 上的  [Una Kravets](https://github.com/w3c/csswg-drafts/issues/5888) 提出的。忍不住尝试使用了它们，看看我们如何从 CSS 容器查询中获得更多发挥的空间。

我将尽力解释每个单位是如何工作的，以及我们可以在哪些地方使用一个（或多个）单位来加强组件如何对其父级宽度做出适配。如果你还没听过 CSS 容器查询，我写了一篇[介绍了很多例子](https://ishadeed.com/article/say-hello-to-css-container-queries/)，以及[容器查询](https://ishadeed.com/article/container-queries-for-designers/)如何影响设计师的工作，我建议你在阅读本文之前先阅读这两篇文章。

> 温馨提示：CSS 容器查询仅支持在具备实验性标志的Chrome Canary版本浏览器中使用。其他版本要使用它，请到 chrome://flags 搜索“Enable CSS Container Queries”并启用该功能。

让我们深入了解一下。

## 简介

在 CSS 中，我们有很多单位可以用于不同的场景，最常用的是 `px` 、 `rem` 和 `em` 。如果有类似于 CSS 容器查询工作方式的单位，那么我觉得应该是视口单位。

CSS 视口单位的工作原理基于适配浏览器的视口尺寸（宽度或/和高度），这很棒，但我们并不是总想使用与视口尺寸有关的单位，如果我们想针对容器的宽度进行查询呢？这就是查询单位的作用。

为了更清楚地说明问题，我想强调视口单位和查询单位之间的区别。请看下图：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqaskxpj61e00nsab602.jpg)

左边区域， `font-size` 相对于视口宽度由 `rem` 和 `vw` 控制，在某些情况下，这种方式会起作用，但由于它是相对于**视口尺寸**可能会引发意想不到的问题。

在处理诸如 `font-size` 、 `padding` 、`margin` 等问题时，查询单位可以节省我们的时间和精力，我们可以使用查询单位来作为手动增加字体大小的替代方式。

参考下面的例子。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqb49l3j61e00rmgmk02.jpg)

我们有一个开始状态是堆叠式的卡片式组件，随着容器变大，慢慢变成主横幅的样式，对于这样的组件，我们可能需要修改以下内容：

- 缩略图大小
- 字体大小
- 元素之间的间距

如果没有 CSS 查询单位，我们需要手动更改上面的内容。

```css
.card {
    /* The stacked, base style */
}

.card__title {
    font-size: 1rem;
}

/* The horizontal style, v1 */
@container (min-width: 400px) {
    .card__title {
        font-size: 1.15rem;
    }
}

/* The horizontal style, v2 */
@container (min-width: 600px) {
    .card__title {
        font-size: 1.25rem;
    }
}

/* The hero style */
@container (min-width: 800px) {
    .card__title {
        font-size: 2rem;    
    }
}
```

注意到 `.card__title` 的字体大小是如何随着不同的容器查询而改变的，我们可以通过使用查询单位和 CSS `clamp()` 函数来加强这一点并避免重复设置 `font-size` 。

```css
.card__title {
    font-size: clamp(1rem, 3qw, 2rem);
}
```

这样，我们只需要定义一次 `font-size` ，如果我们想要直观地比较一下，它是下面这样的：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqcafcij61680o2gmt02.jpg)

如果你熟悉 CSS 视口单位，这对你来说并不陌生，这就像基于容器而不是浏览器的视口进行查询，挺简洁的，不是吗？

[演示](https://codepen.io/shadeed/pen/ZEyxzRQ?editors=0100)

## 查询单位

现在我们已经了解了查询单位应该解决的问题，让我们根据 [CSS 规范](https://drafts.csswg.org/css-contain-3/#container-lengths)继续掌握这些单位：

- 查询宽度（qw）：容器宽度的 1% 。例如， 5qw 将解析为容器宽度的 5% 。
- 查询高度（qh）：容器高度的 1% 。例如，5qh 将解析为容器高度的 5% 。
- 查询最小值（qmin）：查询宽度 `qw` 或查询高度 `qh` 的较小值。
- 查询最大值（qmax）： `qw` 或 `qh` 的较大值。

我们还有另外两个单位： `qi` 和 `qb` ，它们分别表示查询内联和查询块（尺寸），它们适用于我们有不同写作模式的情况。

## 案例与例子

### 卡片组件

除了之前讲解过关于卡片的例子，我们还可以在堆叠式卡片使用查询单位，我们希望字体大小基于容器的宽度而稍大一些。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqef1uwj61ku0m60vb02.jpg)

请注意，字体大小根据容器尺寸而进行细微变大，这对于创建一个无论放在哪里都能工作的堆叠卡片是很有用的。

[演示](https://codepen.io/shadeed/pen/yLXKBqN?editors=0100)

### 根据权重改变 `font-size` 

当一个元素比另一个元素更大时，表明它可能更加重要，一个很好的例子就是侧边栏以及主体， `<aside>` 中标题大小应该小于 `<main>` 部分的标题。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqgiisfj61e00lidgh02.jpg)

注意到 `<aside>` 中的标题是如何比 `<main>` 中的标题小的，由于有了查询单位，我们可以很容易地实现这一点。

首先，我们需要将 `<aside>` 和 `<main>` 定义为 inline-size 容器。

```css
aside,
main {
    container: inline-size;
}
```

接着，我们对章节标题设置查询单位，我们还可以对底部边距使用查询单位。

```css
.section-title {
    font-size: clamp(1.25rem, 3qw, 2rem);
    margin-bottom: clamp(0.5rem, 1.5qw, 1rem);
}
```

[演示](https://codepen.io/shadeed/pen/oNwqvap?editors=0100)

### 个人简介组件

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqjz2owj61e00fydgd02.jpg)

简介可以放在一个较小的容器中（例如：侧边栏），或显示在移动视口，或像页面标题这样的大容器中。

我们可以构建一个可以适应**其容器宽度**的组件，在本例中，用户的头像和字体大小根据其容器的宽度而改变。

```css
.bio {
    container: inline-size;
}

.c-avatar {
    --size: calc(60px + 10qw);
    width: var(--size, 100px);
    height: var(--size, 100px);
    margin-bottom: clamp(0.5rem, 3qmin, 2rem);
}
```

[演示](https://codepen.io/shadeed/pen/wvemwNb?editors=0100)

### 带计数器的标题

在某些情况下，我们需要在有文章正文标题旁边显示一个计数器，这是查询单位的一个完美用例。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqjlemlj61e00icjs602.jpg)

```css
.article-body {
    counter-reset: heading;
}

h2 {
    container: inline-size;
    font-size: clamp(1.25rem, 3qw, 2rem);
    margin-bottom: clamp(0.5rem, 1.5qw, 1rem);
}

h2:before {
    --size: calc(1.25rem + 3qw);
    content: counter(heading);
    counter-increment: heading;
    width: var(--size);
    height: var(--size);
    font-size: calc(0.85rem + 1qw);
    margin-right: calc(var(--size) / 4);
}
```

[演示](https://codepen.io/shadeed/pen/xxrWxxp?editors=0100)

### 动态间距

我们已经使用视口单位创建动态间距的 CSS 网格，它是这样的：

```css
.wrapper {
    gap: calc(1rem + 2vw);
}
```

这很酷，但是在特定包装器中的组件呢？使用视口单位是行不通的，这就是为什么这种情况下，使用查询单位是一个很好的解决方案。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv2kqj5kfmj61e00icdh902.jpg)

```css
.card {
    display: flex;
    flex-direction: column;
    gap: calc(0.5rem + 1qmin);
}
```

当组件切换到水平风格时，间距会大一些。

## 如果我们在没有定义容器的情况下使用查询单位会发生什么？

根据[规范](https://drafts.csswg.org/css-contain-3/#container-lengths)：

> 如果没有合适的查询容器，那么就使用该轴的**小视口尺寸**。

浏览器会像处理视口单位一样处理查询单位，请看下面的例子：

```css
h2 {
    font-size: clamp(1.25rem, 3qw, 2rem);
}
```

如果没有为 `<h2>` 定义容器，浏览器会认为 `3qw` 是视口宽度的 `3%` ，这意味着在没有定义容器的情况下，使用查询单位会导致意想不到的结果。

我希望你喜欢这篇文章，感谢你的阅读！

