# CSS Container Queries For Designers

# 【译】写给设计师看的 CSS 容器查询

> 原文：[https://ishadeed.com/article/container-queries-for-designers/](https://ishadeed.com/article/container-queries-for-designers/)\
> 作者：[Ahmad Shadeed](https://ishadeed.com/hire-me/)\
> 译者：[Levix](https://juejin.cn/user/3650034331558717)

网页设计工作包括处理不同屏幕尺寸的设计，基于这些设计，开发人员使用 CSS 媒体查询来监测视口宽度或高度，然后基于此更改设计。我们在过去的 10 年里就是这样设计网页布局的，而且它即将变得更好，我有一些好消息要告诉你。

CSS 容器查询是网页开发者长期以来梦寐以求的功能，即将出现在 CSS 中，现在作为 Chrome Canary 的一项试验性功能。在本文，我将详细介绍它是什么，它将如何改变设计师的工作流程等等。我并不关心你是否是一个会敲代码的设计师，因为本文的重点是介绍这一概念，以便为下一篇文章做好准备。如果你碰到完全不懂的 CSS bits（概念） ，你完全可以选择跳过它们并继续学习。

废话不多说，我们开始吧！

## 响应式设计的现状

现在，在同一个网页布局的多个版本上设计仍然是可以的，以显示内部部分将如何根据视口宽度而变化，我们设计不同的尺寸，如手机、平板电脑和桌面。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpoqsbjj61nc0sagmw02.jpg)

在上图中，设计师创造了同一设计风格下的三种不同展现方式，因此开发者可以知道如何根据它进行相应开发，到目前为止，一切都挺顺畅的。

现在，我将向你展示更详细的设计和它的变化，这样我可以阐明 CSS 容器查询将为我们解决的**问题**。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpprhenj61nc0sagmw02.jpg)

注意，这是同一个组件包含着三种风格，分别是默认、card 以及 featured ，作为一名设计师，你已经使用了多个版本的布局来展示它，这就像是在说：“这是文章组件在手机上的布局风格，而这是它在平板电脑上的风格”。

在 CSS 中，开发者需要针对该组件创建三种风格，并且每一种都是独一无二的，参考下面的基础样式：

```css
.c-media {
  /* 默认样式 */
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
}

@media (min-with: 400px) {
  .c-media--card {
    display: block;
  }

  .c-media--card img {
    margin-bottom: 1rem;
  }
}

@media (min-with: 1300px) {
  .c-media--featured {
    position: relative;
    /* 其他样式 */
  }

  .c-media--featured .c-media__content {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
  }
}
```

上述（三种）风格差异取决于媒体查询或者视口宽度，意味着我们不能根据其父容器宽度去控制它们。现在你可能在想，这有什么问题？嗯，这是个好问题。

问题是开发者被限制住了，**只能在视口宽度大于某个特定值时**使用一个组件的特定样式类。例如，如果我想在平板电脑中使用“featured”类，那就行不通了，为什么？因为它的媒体查询在视口宽度大于等于 `1300px` 时启动。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpryw7kj60yk0nmq3q02.jpg)

不仅如此，我们还可能面临内容比预期少的问题，有时候，创作者只会添加一篇文章，而设计稿中包含了三篇，在这种情况下，要么我们有一个空白区域，要么文章会适配展开以填充可用空间，请看下图：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpq32lbj60yk0kkt8z02.jpg)

在第一种情况下，文章太宽导致正在使用的图像被破坏（拉伸），而在第二种情况下，效果是一样的，但有更多的网格项正在扩大以填补可用空间，这（体验）并不是很好。

如果使用容器查询，我们可以通过查询父容器来决定如何显示特定的组件去解决这些问题。请看下图，它展示了我们**如何使用容器查询来解决这个问题**。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qq1bi6vj60yk0laq3802.jpg)

因此，我们将解决思路转移到**组件的父容器**上会发生什么呢？换句话说，如果我们查询其父容器，根据其父容器的宽度或高度来决定组件的外观，该怎么办？让我们来了解下容器查询的概念。

## 什么是容器查询

首先，让我定义一个容器，**包含着**其他子元素的元素，有时被称为一个 **wrapper**（包装器），如果你有兴趣了解更多关于容器的信息，[我有一篇完整的文章](https://ishadeed.com/article/styling-wrappers-css/)。

容器查询的功能现在可以在 Chrome Canary 浏览器的 flag 下开启（译者注，指的是 `chrome://flags` ）。感谢 [Miriam Suzanne](https://css.oddbird.net/rwd/query/contain/) 以及其他朋友的努力。

当一个组件被放在一个容器中时，代表着**它被包含**在该容器中，这意味着，我们可以查询其父容器的宽度并基于此对其进行修改。请看下图：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpqk3k0j615o0p2gmp02.jpg)

注意，每张卡片都有一条黄色的轮廓线，它代表**每个组件的父容器**。通过 CSS 容器查询，我们可以基于其父组件的宽度来修改组件，为了更清楚地说明这一点，以下是上述（布局）的 HTML 代码：

```html
<div class="o-grid">
  <div class="o-grid__item">
    <article class="c-media"></article>
  </div>
    <!-- + more items -->
</div>
```

该子组件具有 `.c-media` 类，它的父容器是 `.o-grid__item` 元素，在 CSS 中，我们可以这样做：

```css
.o-grid__item {
  contain: layout inline-size style;
}

.c-media {
  /* 默认样式 */
}

@container (min-width: 320px) {
  .c-media {
    /* 样式 */
  }
}

@container (min-width: 450px) {
  .c-media {
    /* 样式 */
  }
}
```

首先，我们告诉浏览器每个具有 `.o-grid__item` 类的元素都是一个容器，接着，我们告诉浏览器，如果父元素的宽度大于或等于 320px，它就应该呈现出不同的布局，对于 450px 的查询也是如此，这就是 CSS 容器查询的工作方式。

此外，我们可以在任何地方定义它们，也就是说如果需要的话，我们可以在顶层容器上进行查询。现在你已经了解了 CSS 容器查询的基本概念，我想给你展示如下图片。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpzlqkdj61590u0q4402.jpg)

在左侧，这是一个正在调整尺寸的视口，在右侧，一个组件（布局）根据其父组件的宽度改变，这就是容器查询的强大和有用之处。

如果你想进一步了解容器查询的 CSS 细节，我写了一篇关于它的[详细文章](https://ishadeed.com/article/say-hello-to-css-container-queries/)。

## 设计时考虑容器查询

作为一名设计师，你需要适应这个革命性的 CSS 功能，因为它将改善我们设计网页和编写 CSS 的方式。我们不仅会针对屏幕尺寸进行设计，还要考虑到组件在其容器宽度改变时应该如何适配。

现在，设计系统越来越受欢迎，设计团队会构建一套规则和组件，以便其他成员可以基于此构建页面，随着 CSS 容器查询的到来，我们还需要在设计组件时，考虑应该如何根据其父容器宽度进行适配。

考虑如下设计：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpvcyuvj61350u076102.jpg)

请注意，我们有标题栏、文章部分、引言和通讯，它们每一个都应该适应视口或其父容器宽度。

我可以设想将组件分为以下几部分：

- 视口（媒体查询）
- 父容器（容器查询）
- 通用型：不受影响的组件，如按钮、标签、段落。

对于示例的用户界面，我们可以这样划分组件。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpr1vn7j61bg0u0tag02.jpg)

当我们以这种思维方式设计用户界面时，我们可以开始考虑组件不同的变化，这些变化取决于它们的父容器宽度，让我们来探讨一下。

在下图中，请注意文章组件的每个变化是如何在特定的宽度中发挥作用的。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpvtvo2j60u00uxab202.jpg)

作为一名设计师，基于父容器宽度考虑如何设计，一开始可能有点奇怪，但这就是未来的发展趋势，我们向前端开发人员提供每个组件的细节和变化，他们可以使用它们（的规范进行编码）。

不仅如此，我们可能还有一个组件的布局，它只在特定的环境下显示， 例如，事件列表页面， 在这种情况下，明确在哪里使用这个布局是很重要的。

问题是，如何告诉设计师应该在哪里使用这些组件？

## 与开发人员沟通

良好的沟通是项目成功的一个重要因素，作为一名设计师，希望你能提供指导，说明组件不同的适配应该用在哪里，它可以是一个完整的页面设计，也可以是一个显示每个组件如何使用的简单图片。

让我们将其应用于我们之前讨论的文章组件。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpyvwtmj61ag0u0abm02.jpg)

请注意我如何将每个变化映射到特定上下文，而不是视口。 为了进一步证明这一点，我想向你展示该组件在与 CSS grid 一起使用时会有什么不同。

在 CSS grid 布局中，我们可以通过使用 `auto-fit` 关键字来告诉浏览器，如果列的数量少于预期时，我们希望它能够延伸（你可以阅读更多关于它的信息[这里](https://ishadeed.com/article/css-grid-minmax/)）。这个功能很强大，因为它可以帮助我们在同一环境下显示不同的变化。请看下图：

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpx8do0j61ag0u03zu02.jpg)

拥有一个能根据其父容器宽度做出自适应的组件是非常有用的，正如你刚才看到的，我们在桌面尺寸上查看一个页面，并且有不同的部分，每一个部分的列数都不一样。

## 设计响应式组件时要避免复杂化

重要的是要记住，一个组件的内部组成结构就像乐高游戏，你可以根据当前变化对它们进行排序，但凡事都有一个度，有时候，对于前端开发人员来说，与其用容器查询来实现自适应，还不如去实现一个全新的组件。

请考虑以下几点。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qprt83yj60to0c274d02.jpg)

它有以下几点：

- 头像
- 名称
- 按钮
- 键/值对

如果内部结构保持不变，或者至少不包含新的结构，我们可以修改组件，并有以下多种不同布局。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpuxmbnj611s0c2aam02.jpg)

## CSS 容器查询的使用案例

让我们来探讨一些可以使用 CSS 容器查询实现的案例。

### 聊天列表

我在 Facebook messenger 上看到了这种模式，聊天列表根据视口宽度变化，我们可以使用 CSS 容器查询来实现。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qq0b84sj60xc0g8wes02.jpg)

当有足够的空间时，列表会展开并显示每个用户的名字，聊天列表的父元素可以是一个被动态调整大小的元素（例如：通过使用 CSS 视口单位，或 CSS 比较功能）。

下面是我们如何在 CSS 中实现这一点。

```html
<div class="content">
  <aside>
    <ul>
      <li>
        <img src="shadeed.jpg" alt="Ahmad Shadeed" />
        <span class="name">Ahmad Shadeed</span>
      </li>
    </ul>
  </aside>
  <main>
    <h2>Main content</h2>
  </main>
</div>
.content {
  display: grid;
  grid-template-columns: 0.4fr 1fr;
}

aside {
  contain: layout inline-size style;
}

@container (min-width: 180px) {
  .name {
    display: block;
  }
}
```

注意侧边栏的宽度是 `0.4f` ，所以它是动态宽度，另外，我还添加了 `contain` 属性，并且如果容器宽度大于 `180px` ，则会显示用户名。

与此类似的另一个案例是侧边导航栏，我们可以从新的一行或在图标旁边切换导航项标签的位置。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpsx1tgj60xc0fujrq02.jpg)

注意，当容器（侧边栏）较小时，导航项标签是如何切换为新的一行的，而当有足够空间时，导航项标签是如何切换到导航图标旁边的。

[演示](https://codepen.io/shadeed/pen/Popmryw?editors=0100)

### 手风琴

手风琴模式可用于像 FAQs 这样的场景，在某些情况下，我们可能需要在侧边栏或用户界面中的一个小区域内添加 FAQs 列表，容器查询可以提供帮助！

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpu91fdj60xc0omt9c02.jpg)

下面是我们如何使用 CSS 容器查询实现上述功能。

```css
@container (min-width: 180px) {
  .faq-title {
    display: flex;
    justify-content: space-between;
    font-size: 1.25rem;
  }

  .faq__icon {
    width: 60px;
    height: 60px;
    background-color: #4f96e7;
  }
}
```

[演示](https://codepen.io/shadeed/pen/yLMbmGR?editors=0100)

### 搜索框

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpwmphlj60xc0nawf902.jpg)

当我们在多个地方使用通用搜索框时，这可能非常有用，例如，它可以在一个主图使用（在右侧），也可以在一个较小的范围内使用，如侧边栏（在左侧）。

### 活动清单

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qq0yzomj60xc0o2q4w02.jpg)

我个人喜欢这种容器查询的用例，我们可以在多种情况下使用同一个组件，在上图中，我们有简单、中等和大型的布局展示。下面是一个关于如何使用它们的例子。

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpw8qddj60xc0ku75h02.jpg)

同样，这是适应其父容器宽度的**相同组件**， 这不是很棒吗？ 对我来说，是的。

### 作者简介

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qptjxcij60xc0jijs302.jpg)

作者简介是博客的常见组成部分，从上图可以看出，它可以在多种情况下，因此它应该自适应。

### 社交分享

![img](https://tva1.sinaimg.cn/large/008i3skNgy1gv5qpttl0aj60xc0j2jsf02.jpg)

大多数时候我实现一个社交分享组件，需要创建一个在视口大但父容器宽度小的情况下可以使用的版本（例如：侧边栏），通过容器查询，可以通过自适应其父容器宽度来轻松解决这个问题。

当该组件在侧边栏中使用时（在左侧），则使用小版本，当父容器较大时（例如：主区域），将使用完整版本。

感谢你的阅读 :)