# CSS 遮罩

> 原文：[https://ishadeed.com/article/css-masking/](https://ishadeed.com/article/css-masking/)\
> 作者：[Ahmad Shadeed](https://ishadeed.com/hire-me/)\
> 译者：Levix

在设计世界中，遮罩（masking）是实现独特设计效果的流行技术，作为一名设计师，我已经使用过很多次，但我在网页上使用 CSS 遮罩的次数很少，我认为我不使用 CSS 遮罩的原因是浏览器支持情况，它们在 blink 浏览器（Chrome 和 Edge）中只支持了部分特性，而在 Safari 和 Firefox 中完全支持。

好消息是 CSS 遮罩将成为 [Interop 2023](https://web.dev/interop-2023/) 的一部分，这意味着我们应该期待跨浏览器支持（耶！！）。

在这篇文章中，我将介绍 CSS 遮罩是什么，它是如何工作的，以及输出一些结合它的用例和示例。

让我们开始吧。

## 什么是遮罩？

简单地说，遮罩的工作方式是在不擦除元素的情况下将其部分隐藏。

参考以下图片：

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eeb44a70603e4ebb925dab0869a72ea7~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d83ad0b3ad74ee3996da922f6a972df~tplv-k3u1fbpfcp-zoom-1.image)

我们有一张图片和一个遮罩，在像 Photoshop 这样的设计应用程序中，我们可以把图片插入到灰色形状内，这样就会产生一个蒙版的图像。

它的工作原理是隐藏图像的某些部分而不是擦除它（它们仍然在那里，只是被隐藏了）。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f594f47e256646eab0e7430bb54d7cfb~tplv-k3u1fbpfcp-zoom-1.image)

这就是遮罩的核心概念，使用一个形状来显示和隐藏元素的某部分，我们可以进一步探索更独特的遮罩内容。

例如，我们可以创建一个如下图那样的渐变遮罩。

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a641d661248f4f41a73a0c4383b785c1~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0dac290373f649549c9f7c70688b759e~tplv-k3u1fbpfcp-zoom-1.image)

在渐变中，有填充和透明像素，填充的部分是元素的一部分可见区域，透明的部分则是隐藏的部分。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8803a956ee584e28b1de377873e97416~tplv-k3u1fbpfcp-zoom-1.image)

在 Photoshop 中，我们可以向一组图层添加图层蒙版，该组内的内容将被遮罩。它的工作原理是使用画笔工具隐藏组的一部分从而实现遮罩。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1be9910a80484675bce3b7df5508bbf8~tplv-k3u1fbpfcp-zoom-1.image)

被遮罩的内容并没有被擦除，而是被隐藏了（请注意组项目）。

好了，以上都偏理论，在接下来的文章中，我将介绍 CSS 遮罩的使用方式，它是如何工作的以及一些用例和示例。

## 如何在 CSS 中使用遮罩

在 CSS 中，有几种遮罩元素的方式：

-   `mask` 属性
-   `clip-path` 属性
-   SVG `<mask>`

`mask` 属性和 `clip-path` 的主要区别在于前者用于图像和渐变，而后者用于路径，本文的重点将放在 `mask` 属性上。

在 CSS 中，我们有 `mask` 简写属性，类似于 `background` 属性一样，是的，你没看错。这就是我认为它容易记住语法的原因，因为它的语法与 `background` 属性完全相同，但具有**少量其他**属性。

而不是列出所有的 CSS 遮罩属性，我将逐步添加一个遮罩功能的示例，让你可以在视觉上发现差异。

CSS 背景如下所示：

```
.card__thumb {
    background-image: url('hero-cool.png');
}
```

CSS 遮罩如下所示：

```
.card__thumb {
    mask-image: url('hero-cool.png');
}
```

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ad1e12dcd0b41759bce5edb160ea5ba~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/54371b26b9c641219731966dcd2ab215~tplv-k3u1fbpfcp-zoom-1.image)

很酷，对吧？这让你理解和掌握 CSS 遮罩变得更加容易。

现在，让我们用 CSS 重新做一下最初的示例。

首先，我需要将形状导出为 png 图像。

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37895d18f5fd462ab7f15b9c2fcde9c7~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f4eed338ad84cc18e7504006c90234e~tplv-k3u1fbpfcp-zoom-1.image)

假设我想将遮罩应用于该图像。

```
<img src="ahmad-shadeed-web-directions.jpg" alt="" />
```

```
img {
    mask-image: url("shape.png");
}
```

你能预测出结果吗？默认情况下，遮罩会重复，其大小将等于遮罩图像本身，如下图所示：

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dfffd42b05e74891a90c3f25f3eeaaae~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed4542a2b213462497a56ae14a16c23e~tplv-k3u1fbpfcp-zoom-1.image)

为了恢复，我们需要将 `mask-repeat` 设置为 `no-repeat`，就像 CSS 背景图像一样。

```
img {
    mask-image: url("shape.png");
    mask-repeat: no-repeat;
}
```

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ee977ebbb714383a2831c21f320580c~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/668d045db14b487c860ae8390f149202~tplv-k3u1fbpfcp-zoom-1.image)

太棒了！请注意，遮罩位于左上角，可以通过 `mask-position` 更改它，再次注意，语法与 CSS 背景图片配置完全相同！

```
img {
    mask-image: url("shape.png");
    mask-repeat: no-repeat;
    mask-position: center;
}
```

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e3aa31aaa8ad4fc4880c16ad786f32c2~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc3d9368348c45d085070ba5097a4633~tplv-k3u1fbpfcp-zoom-1.image)

除了位置，我们还可以更改遮罩大小，这对于使用遮罩图像响应元素的大小很有用。

```
img {
    mask-image: url("shape.png");
    mask-repeat: no-repeat;
    mask-position: center;
    mask-size: 60%;
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dc23878f501b4c1dadedcb7b90086bd0~tplv-k3u1fbpfcp-zoom-1.image)

还有一些遮罩属性，但我现在不想用它们来打击你学习的兴趣，稍后我将用实际用例来介绍它们。

## 使用 CSS 渐变进行遮罩

CSS 遮罩不仅仅是使用图像，我们还可以利用渐变来创建强大且有用的遮罩效果。

稍后我将展示一些有用的用例，但现在，我想重点介绍渐变如何与遮罩配合使用的基本原理。

在以下示例中，`mask-image` 由从全黑到透明的 CSS 线性渐变组成。

```
img {
    mask-image: linear-gradient(#000, transparent);
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8a47630ca7d844edb392291e4ac17d8c~tplv-k3u1fbpfcp-zoom-1.image)

根据 [MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/mask-image) 的说法：

> 默认情况下，这意味着遮罩图像的 alpha 通道将乘以元素的 alpha 通道，可以使用 mask-mode 属性来控制。

这意味着，你可以使用除黑色以外的任何颜色，遮罩仍将起作用，因为默认遮罩模式设置为 `alpha`（稍后我将详细介绍此内容）。

```
img {
    mask-image: linear-gradient(red, transparent);
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b9140bcb25234910be7208b675539c8d~tplv-k3u1fbpfcp-zoom-1.image)

同样，遮罩的概念是透明像素将被隐藏，以下是具有硬色停止的渐变的简化示例：

```
img {
    mask-image: linear-gradient(#000 50%, transparent 0);
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a903deaefd76420bbb8386815226df96~tplv-k3u1fbpfcp-zoom-1.image)

太酷了！现在，核心遮罩概念已经清晰了（我希望如此），让我们探索一些实际用例的CSS遮罩。

## 实际使用案例和示例

### 淡化图像

使用遮罩的一个有趣用途是使图像淡出并与其下面的背景混合。

考虑以下图像：

css-masking-use-case-fade-image-light.png

乍一看，你可能会想要添加一个与背景颜色相同的渐变。像这样：

```
.hero__thumb:after {
    position: absolute;
    inset: 0;
    background: linear-gradient(to top, #f1f1f1, transparent)
}
```

虽然这可能会起作用，但当主背景颜色改变时，它就会失败，注意到主页横幅会出现顿挫感：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b766273bf30843deb660b44b86bb57ab~tplv-k3u1fbpfcp-zoom-1.image)

使用 CSS 遮罩，我们可以遮盖主页横幅，使其与任何背景颜色配合使用。

```
.hero__thumb {
    mask-image: linear-gradient(#000, transparent);
}
```

就这样！现在淡化过渡效果是真实的，并且不会在更改主页面背景时失效，请参见以下示例：

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7b134dee8bca4b839d927dad17813c5b~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/73e4a816f7714d2b8a95fa5564b2696e~tplv-k3u1fbpfcp-zoom-1.image)

### 遮罩文本内容：示例 1

当我们想要显示长文本但空间不足以完全显示时，解决方案是在开头和结尾处淡化文本，然后，文本将在任一方向上动画以显示剩余内容。

考虑以下图像：

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3fb3cc8f15ad4464a7c7438a3228a72a~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/963d7e51ba7f4ef685122b615e56ebef~tplv-k3u1fbpfcp-zoom-1.image)

再次，使用渐变的 hack 不起作用，因为内容下面的背景被更改了，它可以是纯色或图像。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95d507566e544f088e4b6994f755761d~tplv-k3u1fbpfcp-zoom-1.image)

要在 CSS 中实现它，我们需要添加一个渐变遮罩，以在开头和结尾处淡化内容。

我喜欢在 CSS 渐变中做到这一点并查看结果，然后将其应用为遮罩，这有助于在使用其作为遮罩之前将渐变可视化。

```
.c-card__footer {
    background-image: linear-gradient(90deg, transparent, #000 15%, #000 85%, transparent 100%);
}
```

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d098ef27933847d9aa0060ae8ad7fbb2~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e6d53c60bd594a35a6b9af6c4954b3c8~tplv-k3u1fbpfcp-zoom-1.image)

有了上面的内容，我们就可以将其应用为遮罩。

```
.c-card__footer {
    mask-image: linear-gradient(90deg, transparent, #000 15%, #000 85%, transparent 100%);
}
```

还不完美？我们可以微调渐变值，直到结果完美为止。

### 遮罩文本内容：示例 2

这与前面的示例相同，但应用于垂直方向，我在 Instagram 的直播视频中看到过这种情况。

考虑以下图像：

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e430dce52b4e45809ac307323f2d5a16~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6fc3b5c4f89d4e139e9534229fb3c26d~tplv-k3u1fbpfcp-zoom-1.image)

请注意，内容从顶部被冲洗出来？这个小地方可以有供稿评论，操作和其他内容。使用 CSS 遮罩非常适合这样做。

首先，让我们看一下渐变。

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6b61045352c6478ca4c241fcd2340887~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f24e9b3779e4c1892aac23e2f5e90c5~tplv-k3u1fbpfcp-zoom-1.image)

在 CSS 中，它可能是这样的：

```
.whatever-the-class-name {
    mask-image: linear-gradient(to bottom, transparent, #000);
}
```

### 遮罩列表

我在研究 CSS 遮罩时看到了这个[很酷的例子](https://www.youtube.com/watch?v=UPJeMtYFMn8)，这个想法是我们有一系列特征、课程或其他任何东西，我们想淡化文本来使用户对其中的内容更加好奇。

考虑以下示例：

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/49cd372a6e0949d4be4f7c4a6625960d~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2996e46532ea4d528fbe36aafaea87af~tplv-k3u1fbpfcp-zoom-1.image)

你可以在左边看到一个列表，在最底部淡化，使用 CSS 遮罩非常适合此操作，因为它可以与下面的背景混合，无论是图像还是深色背景。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c499189de704e36987803f000c79e6d~tplv-k3u1fbpfcp-zoom-1.image)

让我们看看遮罩渐变，了解其工作原理。

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/766121f339d74105b00f1ad3092e02d9~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/427c6bfa5f4848aaa7058e8928e3b0c7~tplv-k3u1fbpfcp-zoom-1.image)

在 CSS 中，它看起来像这样：

```
.list {
    mask-image: linear-gradient(to bottom, #000, transparent 95%);
}
```

### 有趣的图像效果

使用 CSS 遮罩和渐变，创建视觉效果的可能性是无限的。这是一个简单的示例，说明我们如何为图像创建视觉效果。

这是我为此演示制作的快速设计。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/cbf3354fa56a4c30a915641105994ca6~tplv-k3u1fbpfcp-zoom-1.image)

你看到的图像效果包括 5 个线性渐变，通过为每个渐变添加不同的渐变位置，我们可以获得类似的效果。

让我们更详细地看看渐变：

```
.thumb {
    mask-image: linear-gradient(to bottom, #000, #000),
    linear-gradient(to bottom, #000, #000),
    linear-gradient(to bottom, #000, #000),
    linear-gradient(to bottom, #000, #000),
    linear-gradient(to bottom, #000, #000);
    mask-size: 18% 70%;
    mask-position: 0 100%, 25% 25%, 50% 50%, 75% 0, 100% 50%;
    mask-repeat: no-repeat;
}
```

从视觉上看，遮罩如下所示：

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2a3bf0be1b4942828f0d5a9bbd24279b~tplv-k3u1fbpfcp-zoom-1.image)

要在每个矩形的遮罩上创建淡出效果，我们需要更新每个渐变并包括“transparent”关键字。

```
.thumb {
    mask-image: linear-gradient(to bottom, transparent, #000),
    linear-gradient(to bottom, #000, transparent),
    linear-gradient(to bottom, transparent, #000),
    linear-gradient(to bottom, #000, transparent),
    linear-gradient(to bottom, transparent, #000);
    mask-size: 18% 70%;
    mask-position: 0 100%, 25% 25%, 50% 50%, 75% 0, 100% 50%;
    mask-repeat: no-repeat;
}
```

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9999642ceb6d48308f1c7281fad89224~tplv-k3u1fbpfcp-zoom-1.image)

我们还可以在悬停时对遮罩大小或位置进行动画处理。

[css-mask-image-effect.mp4](https://ishadeed.com/assets/css-masking/css-mask-image-effect.mp4)

这很强大，想象一下，将其与基于滚动的动画结合起来，事情可能会失去控制（不过是以一种好的方式）。

### 圆角标签页

我考虑为 UI 特效尝试 CSS mask，这个特效叫做 [圆角标签页](https://css-tricks.com/tabs-with-round-out-borders/)。

这个想法是我们希望以一种与元素的 `border-radius` 融合的方式将元素的侧面圆角化。

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/259a4b1c180743168feaf25e855022ab~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bab99147124642ea94c04ad52bb59032~tplv-k3u1fbpfcp-zoom-1.image)

在这篇博客文章中，Chris Coyier 解释了一种使用多个伪元素实现的技巧，更动态的解决方案是使用 CSS mask。

首先，让我们更仔细地看一下我想要实现的形状。

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3d51e7bdf9024e9392e7f4daebcccc10~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d99611b89cc4f208947cd48cbcb4e27~tplv-k3u1fbpfcp-zoom-1.image)

形状由一个正方形和一个圆形组成，我们需要的是它们的交集。

如何实现呢？我们可以使用多层 mask 以及 `mask-composite` 属性在它们上面执行 composting 操作。

首先，我们需要创建一个元素来容纳 mask。

```
.nav-item.active:before {
    content: "";
    position: absolute;
    left: 100%;
    bottom: 0;
    width: 24px;
    height: 24px;
    background-color: var(--active-bg);
}
```

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/50781a08b2d64b2ead3f7dba3feaed74~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/92cb7cae14e842f6a73e082296019b47~tplv-k3u1fbpfcp-zoom-1.image)

在这个空间中，我们需要绘制一个圆和一个正方形以进行组合，幸运的是，我们可以通过混合线性和径向渐变来实现。

```
.nav-item.active:before {
    content: "";
    position: absolute;
    left: 100%;
    bottom: 0;
    width: 24px;
    height: 24px;
    background-color: var(--active-bg);
    background-image: linear-gradient(to top, #000, #000),
    radial-gradient(circle 15px at center, #000 80%, transparent 81%);
    background-size: 12px 12px, 100%;
    background-position: bottom left, center;
    background-repeat: no-repeat, repeat;
}
```

注意以下几点：

-   我为正方形添加了 `12px 12px` 作为大小。
-   正方形位于左下角。
-   正方形形状不需要重复。

![https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4721c2f91ce64fcfa61fbb57fa1f5882~tplv-k3u1fbpfcp-zoom-1.image](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8897cf5fc9d547188608f84e6daa1c29~tplv-k3u1fbpfcp-zoom-1.image)

上面只是为了视觉上说明两个渐变的效果，下一步是实现它们！在 CSS 遮罩中，我们可以使用 `mask-composite` 属性来组合两个形状。

```
.nav-item.active:before {
    content: "";
    position: absolute;
    left: 100%;
    bottom: 0;
    width: 24px;
    height: 24px;
    background-color: var(--active-bg);
    mask-image: linear-gradient(to top, red, red),
    radial-gradient(circle 15px at center, green 80%, transparent 81%);
    mask-size: 12px 12px, 100%;
    mask-position: bottom left, center;
    mask-repeat: no-repeat, repeat;
    mask-composite: subtract;
}
```

这是上述形状的 CSS 右侧。对于另一个，我们只需更改 `mask-position` 即可翻转。

```
.nav-item.active:after {
    /* 其他样式 */
    mask-position: bottom right, center;
}
```

[演示](https://codepen.io/shadeed/pen/YzObqbw/f8ac9db5ddde52a6214f1a8c2859e4f8?editors=0100)


### 多个头像裁剪

在我关于[镂空效果](https://ishadeed.com/article/thinking-about-the-cut-out-effect/)的文章中，我探讨了使用 CSS 创建镂空的不同方法。

其中一个例子非常适合 CSS 遮罩。

使用 CSS 遮罩，我们可以使用径向渐变来实现该效果。

```
.avatar {
    -webkit-mask-image: radial-gradient(ellipse 54px 135px at 11px center, #0000 30px, #000 0);
}
```

我强烈建议阅读这篇文章，因为有很多像这样的详细例子。

## 结论

当我开始学习 CSS 遮罩时，资源有限，更重要的是，我们可以在日常工作流程中使用的实际用例非常少。我希望通过本文让你知道在下一个项目中在哪里可以使用 CSS 遮罩。

顺便说一句，我没有深入研究 `mask-mode` 等属性，因为说实话，我没有发现用它们来解决的问题（直到现在）。当我有一个更有说服力的使用这种属性的例子时，我会更新这篇文章。

## 更多资源

如果您想在更多的 CSS 遮罩示例中提高自己的技能，那么您必须查看以下内容：

-   [为你的头像提供华丽的悬停效果](https://css-tricks.com/a-fancy-hover-effect-for-your-avatar/) by Temani Afif
-   [Mask Compositing: 快速入门课程](https://css-tricks.com/mask-compositing-the-crash-course/) by Ana Tudor
-   [仅使用 CSS 实现的语音泡](https://codepen.io/t_afif/pen/eYMbrJN?editors=0100) by Temani Afif
-   [酷炫的梯度边框悬停效果](https://codepen.io/t_afif/pen/qBpRbWr?editors=0100) by Temani Afif

感谢您的阅读。