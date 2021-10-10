#### 一、flex弹性的概念：

弹性盒子是一种用于按行或按列布局元素的一维布局方法，元素可以膨胀以填充额外的空间，收缩以适应更小的空间，适用于任何元素上，如果一个元素使用了flex弹性布局（以下都会简称为：flex布局），则会在内部形成[BFC](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)，flex布局已经得到了所有浏览器的支持，这意味着，现在就能放心，安全的使用这项技术。

![](https://files.mdnice.com/user/20608/a8cee127-6b82-4fe4-b9f5-8a104632be09.jpg)

#### 二、主轴与交叉轴：

学习flex布局需要明白”主轴“与”交叉轴“的概念，采用flex布局的元素，称为”容器“ （ flex container），它的所有子元素都是容器的”项目“（flex item），容器默认存在两根轴：水平的主轴（main axis）和垂直的交叉轴（cross axis）。主轴的开始位置（与边框的交叉点）叫做 `main start` ，结束位置叫做 `main end` ；交叉轴的开始位置叫做 `cross start` ，结束位置叫做 `cross end` 。

![](https://files.mdnice.com/user/20608/57175a50-e845-4829-8e30-3bc06365d9ab.png)

#### 三、容器的属性

容器和项目各有6个属性，见下图

![](https://files.mdnice.com/user/20608/7f7cc3f3-0fb8-4b82-99fd-82c37b583b5f.png)

![](https://files.mdnice.com/user/20608/6dc90a33-4b6d-4269-97ed-f8420cc18556.png)

##### 3.1 flex-direction

`flex-direction` 属性决定主轴的方向（即项目的排列方向）

```css
flex-direction: row | row-reverse | column | column-reverse;
```

* `row`（默认值）：主轴为水平方向，起点在左端。
* `row-reverse`：主轴为水平方向，起点在右端。
* `column`：主轴为垂直方向，起点在上沿。
* `column-reverse`：主轴为垂直方向，起点在下沿。

![](https://files.mdnice.com/user/20608/0cf1b3bd-df61-4f01-8535-fef6113559b7.png)

##### 3.2 flx-wrap

默认情况下，项目都排在一条线上，无论是否给定宽度，都是不会主动换行的：

```css
.main {
    width: 500px;
    height: 300px;
    background: skyblue;
    display: flex;
}

.main div {
    width: 100px;
    height: 100px;
    background: pink;
    font-size: 20px;
}
```

![](https://files.mdnice.com/user/20608/a01a845a-2740-4a29-8826-02a4017ad796.png)

如果需要换行，需要设置flex-wrap

```css
  flex-wrap: nowrap | wrap | wrap-reverse;
```

* `nowrap`（默认值）：不换行。
* `wrap`：换行，第一行在上方。
* `wrap-reverse`：换行，第一行在下方。

![](https://files.mdnice.com/user/20608/b8df9005-fe03-4a44-ad53-d83dca2a293f.png)

##### 3.3 flex-flow

`flex-flow` 属性是 `flex-direction` 属性和 `flex-wrap` 属性的简写形式，默认值为 `row nowrap`

```css
flex-flow: <flex-direction>|| <flex-wrap>;
```

##### 3.4 justify-content

`justify-content` 属性定义了项目在主轴上的对齐方式

```css
justify-content: flex-start | flex-end | center | space-around | space-between | space-between;
```

* `flex-start`（默认值）：左对齐
* `flex-end`：右对齐
* `center`： 居中
* `space-around`：每个项目两侧的间隔相等。
* `space-between`：两端对齐，项目之间的间隔都相等。
* `space-evenly`：每个项目的间隔与项目和容器之间的间隔是相等的。

![](https://files.mdnice.com/user/20608/93594377-9253-4f45-a6a5-9631718be234.png)

##### 3.5 align-items

`align-items` 属性定义项目在交叉轴上如何对齐

```css
align-items: flex-start | flex-end | center | baseline | stretch;
```

* `flex-start`：交叉轴的起点对齐。
* `flex-end`：交叉轴的终点对齐。
* `center`：交叉轴的中点对齐。
* `baseline`:  项目的第一行文字的基线对齐。
* `stretch`（默认值）: 如果项目未设置高度或设为auto，将占满整个容器的高度。

![](https://files.mdnice.com/user/20608/0e854044-a410-42d8-8c5c-11f60785452e.png)

##### 3.6 align-content

`align-content` 属性定义了多根轴线的对齐方式，前提是需要设置flex-wrap: wrap，否则不会有效

```css
align-content: flex-start | flex-end | center | space-between | space-around | stretch;
```

* `flex-start`：与交叉轴的起点对齐。
* `flex-end`：与交叉轴的终点对齐。
* `center`：与交叉轴的中点对齐。
* `space-between`：与交叉轴两端对齐，轴线之间的间隔平均分布。
* `space-around`：每根轴线两侧的间隔都相等。
* `stretch`（默认值）：轴线占满整个交叉轴。

![](https://files.mdnice.com/user/20608/efa33f8b-200a-443f-a3c5-099c20407cb3.png)

#### 四、项目的属性

##### 4.1 order

`order` 属性定义项目的排列顺序。数值越小，排列越靠前，默认为0，可以是负数。

```css
order: <integer>;
```

![](https://files.mdnice.com/user/20608/42165a22-3f59-464e-8a1a-fcbbf1689551.png)

##### 4.2 flex-grow

`flex-grow` flex容器中剩余空间的多少应该分配给项目，也称为扩展规则。最终的项目的宽度为：自身宽度 + 容器剩余空间分配宽度，flex-grow最大值是1，超过1按照1来扩展

```css
flex-grow: <number>;
/* default 0 */
```

![](https://files.mdnice.com/user/20608/547e5360-91d5-4698-bb34-bc6543bf4148.png)

##### 4.3 flex-shrink

`flex-shrink` 属性指定了 flex 元素的收缩规则。flex 元素仅在默认宽度之和大于容器的时候才会发生收缩，其收缩的大小是依据 flex-shrink 的值，默认值是1

```css
flex-shrink: <number>;
/* default 1 */
```

默认情况下，第一个div宽度是200，第二个div宽度是300，两个相加应该超过父元素的400，但是由于 flex-shrink 都设置为1，将两个div都收缩在父元素中

```
.item {    width: 400px;    height: 300px;    background: skyblue;    display: flex;    padding: 5px; }.item div {    height: 100px;    font-size: 20px;}.item div:nth-child(1) {    flex-shrink: 1;    width: 200px;    background: pink;}.item div:nth-child(2) {    flex-shrink: 1;    width: 300px;    background: cadetblue;}
```

![](https://files.mdnice.com/user/20608/9a9b0de8-bc43-407c-b3dc-c291c8ad3dea.png)

![](https://files.mdnice.com/user/20608/8b3defc3-4edf-46c2-bdeb-3d2dd4c91d57.png)

![](https://files.mdnice.com/user/20608/ef094fcf-a20b-4528-bf20-c84b8cddc5b3.png)

如果将第一个div设置为 `flex-shrink: 0; ` 不收缩，则按容器实际宽度展示

![](https://files.mdnice.com/user/20608/5f7a9ed9-a495-4a64-8b4f-e8c7df154dc8.png)

那收缩后的子项宽度是怎么样计算的呢？实际上有一个公示：

1. `(200+300)`所有子项的宽度的和 - `(400)`容器的宽度  = `(100)`
2. 第一个子项的宽度占比：`2/5`，第二个子项的宽度占比：`3/5`
3. 则第一个子项的的宽度为：`200`  - `2/5 * 100` = 160，第二个子项的宽度为：`300`  - `3/5 * 100` = 240

##### 4.4 flex-basis

`flex-basis` 指定了子项在容器主轴方向上的初始大小，优先级高于自身的宽度width

```css
flex-basis: 0 | 100% | auto | <length>
```

宽度是200，但是由于设置了 `flex-basis: 300px; ` ，所以子项最终宽度是大于自身设置的宽度

```
.item {    width: 400px;    height: 300px;    background: skyblue;    display: flex;    padding: 5px;}.item div {  height: 100px;  font-size: 20px;}.item div {  width: 200px;  flex-basis: 300px;  background: pink;}
```

![](https://files.mdnice.com/user/20608/c5996ef9-4e11-4cb6-b063-6b990844a2a0.png)

##### 4.5 flex

`flex` 属性是 `flex-grow` , `flex-shrink` 和 `flex-basis` 的简写，默认值为 `0 1 auto` 。后两个属性可选。

```css
flex: none | [ <'flex-grow'><'flex-shrink'>? || <'flex-basis'>]
```

##### 4.6 align-self

`align-self` 属性允许单个项目有与其他项目不一样的对齐方式，可覆盖 `align-items` 属性。默认值为 `auto` ，表示继承父元素的 `align-items` 属性，如果没有父元素，则等同于 `stretch` 。

```css
align-self: auto | flex-start | flex-end | center | baseline | stretch;
```

```
.item {    width: 400px;    height: 300px;    background: skyblue;    display: flex;    padding: 5px;}.item div {    height: 100px;    font-size: 20px;}.item div {    width: 200px;    flex-basis: 300px;}.item div:nth-child(1) {    background: pink;    align-self: flex-start;}   .item div:nth-child(2) {    background: violet;    align-self: center;}   .item div:nth-child(3) {    background: greenyellow;    align-self: flex-end;}  
```

![](https://files.mdnice.com/user/20608/75a615ca-a5d5-4a09-9428-a93ec4b56b97.png)

#### 五、关于flex布局在IE浏览器上的坑

虽然 `flex布局` 已经得到了 `IE浏览器` 的支持，但是部分属性在 `IE浏览器` 上会不生效，或者效果与其他浏览器不一致，在[Flexbugs](https://github.com/philipwalton/flexbugs)中可以看到 `flex布局` 在 `IE浏览器` 糟糕表现的详情描述

这里大概总结了几个，解决办法可以看链接

* [flex-direction: column属性不生效](https://blog.csdn.net/heyNewbie/article/details/101302169)
* [不支持min-height和flex: 1 ](https://www.cnblogs.com/SamWeb/p/9836497.html)
* [align-items: center文字溢出等问题](https://blog.csdn.net/zyq19930325/article/details/87930077?utm_medium=distribute.pc_aggpage_search_result.none-task-blog-2~aggregatepage~first_rank_ecpm_v1~rank_aggregation-1-87930077.pc_agg_rank_aggregation&utm_term=flex%E5%B8%83%E5%B1%80%E5%85%BC%E5%AE%B9ie%E6%B5%8F%E8%A7%88%E5%99%A8&spm=1000.2123.3001.4430)

#### 六、布局案例

##### 6.1 等高布局

每一列的内容不一样，但容器的高度时等高的

![](https://files.mdnice.com/user/20608/8f3b0b9b-c5c7-4559-84fc-9c3d21c618f9.png)

```
.item {    width: 400px;    height: 300px;    background: skyblue;    display: flex;    justify-content: space-between;    padding: 5px;}.item div {    width: 100px;    font-size: 20px;    background: pink;}.item div p {		text-align: center;}
```

![](https://files.mdnice.com/user/20608/a7c35a50-0cb4-40f3-a41f-fb9385eea3b4.png)

##### 6.2 左侧宽度固定，右侧自适应布局

常见的TOB系统布局方式，左侧是菜单树，右侧是内容

```
html, body {    margin: 0;    padding: 0;}.container {    display: flex;    width: 100%;    height: 100vh;    background: skyblue;}.left-tree {    width: 200px;    height: 100%;    background: pink;}.main {		flex: 1 1 auto;}
```

![](https://files.mdnice.com/user/20608/9b0f9ab4-9580-4607-b880-5b738e63eec4.png)

##### 6.3 粘性页脚

无论中间的内容有多少，页脚始终在底部展示

```
html, body {    margin: 0;    padding: 0;}.container {    display: flex;    flex-direction: column;    width: 100%;    height: 100vh;}.header {    width: 100%;    height: 60px;    background: pink;}.main {    flex: 1 1 auto;    background: skyblue;}.footer {    width: 100%;    height: 60px;    background: pink;}
```

![](https://files.mdnice.com/user/20608/adae1ba0-6333-42af-92f6-f184bab0de53.png)

#### 七、总结

`flex布局` 是目前最流行的布局方式之一，优点是浏览器兼容性较好，学习成本较低，上手简单，可以快速通过 `flex布局` 实现布局效果。缺点是相较于 `grid网格布局` 来说， `flex布局` 是 `一维布局` ，一般用于单行或者单列的布局，如果要实现多行多列的布局，推荐使用 `gird网格布局` 。

理解轴的概念对学习flex布局来说至关重要，掌握 `flex布局` 会对我们编写布局时，事半功倍！本文整体介绍了轴、容器、项目的概念，再结合了几个具体的布局案例来讲解flex布局，希望能对读者有所帮助，谢谢！

#### 八、参考

* [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex)
* [阮一峰：Flex布局教程](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html) 
* [一篇文章弄懂flex布局](https://www.cnblogs.com/echolun/p/11299460.html) 
* [caniuse.com](https://caniuse.com/?search=flex)
* [IE 11 flex布局兼容性问题 ---- 不支持min-height 和flex:1](https://www.cnblogs.com/SamWeb/p/9836497.html)
* [关于flex布局在IE浏览器上的坑](https://blog.csdn.net/heyNewbie/article/details/101302169)
* [Flexbugs](https://github.com/philipwalton/flexbugs)
* [CSS:flex布局浏览器兼容处理 ie8, ie9](https://blog.csdn.net/yangwqi/article/details/111476985)
* [W3C](https://www.w3.org/TR/2012/CR-css3-flexbox-20120918/)
