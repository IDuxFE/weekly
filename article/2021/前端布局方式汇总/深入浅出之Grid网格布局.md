 

#### 一、简介

Grid网格布局 (下面都简称为 Grid布局)，是一个基于栅格的二维布局系统，旨在彻底改变基于网格用户界面的设计。CSS 一直以来并没有把布局做的足够好。刚开始，我们使用 `table` ，后来是 `float` ， `position` 和 ` inline-block` ，这些本质上是一些 `hacks` 而且许多重要功能尚未解决（例如垂直居中）。虽然[flex弹性布局](https://developer.mozilla.org/zh-CN/docs/Web/CSS/flex)可以做到这些，但是flex布局实际上是一维布局，而Grid布局是二维的，它远比flex布局要强大，以下是Grid布局的浏览器兼容性，支持 `Chrome57+` 、 `Edge16+` 、 `Firefox52+` 、 `Safari10.1+` 等 

![](https://files.mdnice.com/user/20608/cc2f06db-52be-4257-8917-b4dc84c2795c.png)

#### 二、容器和项目

采用网格布局的区域，称为"容器"（container）。容器内部采用网格定位的子元素，称为"项目"（item）。

![](https://files.mdnice.com/user/20608/b5041e19-4621-4db3-9a8a-f65f4c1014ca.png)

![](https://files.mdnice.com/user/20608/a249f3ac-2ba9-4732-b4fb-f6d7574f12ff.png)

#### 三、行、列、单元格、 区域、网格线

容器里面的水平区域称为"行"（row），垂直区域称为"列"（column）。行和列的交叉区域，称为"单元格"（cell）。正常情况下， `n` 行和 `m` 列会产生 `n x m` 个单元格。比如，例如：3行3列会产生9个单元格。一定数量的单元格可以组成"区域"（area）, 区域必须是正方形/长方形的，不可以是其他形状。

![](https://files.mdnice.com/user/20608/617ddd46-8091-440c-b445-1d03fb5a192c.png)

划分网格的线，称为"网格线"（grid line）。水平网格线划分出行，垂直网格线划分出列。正常情况下， `n` 行有 `n + 1` 根水平网格线， `m` 列有 `m + 1` 根垂直网格线，比如三行就有四根水平网格线。

![](https://files.mdnice.com/user/20608/5c7c5b1b-86b3-40ce-ae18-0531e4079322.png)

#### 四、 容器属性

##### 4.1 display:  grid， display: grid: inline-grid

`display: grid` 指定一个容器采用网格布局

![](https://files.mdnice.com/user/20608/6b05e5ae-2d2d-4f4e-be1f-4df152151a57.png)

默认情况下，网格都是块级元素，使用 `display: inline-grid` 可以将网格设置为行内元素

![](https://files.mdnice.com/user/20608/99686e21-e6cd-4d3d-a153-942a42e73183.png)

##### 4.2 grid-template-columns，grid-template-rows 

`grid-template-columns` 属性定义每一列的列宽， `grid-template-rows` 属性定义每一行的行高

```css
grid-template-columns<rows>: length | percent | auto | fr
```

定义一个三行三列的网格，列宽和行高都是 `100px` ，使用了网格布局，基本无须单独控制子项的宽高了

```css
.mian {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
}

.main div {
    background: pink;
}
```

![](https://files.mdnice.com/user/20608/6b05e5ae-2d2d-4f4e-be1f-4df152151a57.png)

除了使用绝对单位，也可以使用百分比

```css
.main {
    display: grid;
    grid-template-columns: 33.33% 33.33% 33.33%;
    grid-template-rows: 33.33% 33.33% 33.33%;
}
```

![](https://files.mdnice.com/user/20608/6b05e5ae-2d2d-4f4e-be1f-4df152151a57.png)

也可以使用 `auto` ，子项会自适应父容器剩余大小去撑开

```css
.main {
    display: grid;
    grid-template-columns: 100px 50px auto;
    grid-template-rows: 100px auto 50px;
}
```

![](https://files.mdnice.com/user/20608/63171e88-8516-43ec-8aba-c2e2f9350623.png)

最后可以使用推荐的关键字 `fr` ，可以理解为比例值，将行与列都分成3份

```css
.main {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr;
    grid-template-rows: 1fr 1fr 1fr;
}
```

![](https://files.mdnice.com/user/20608/6b05e5ae-2d2d-4f4e-be1f-4df152151a57.png)

将第二行与第二列都划分成2份

```css
.main {
    display: grid;
    grid-template-columns: 1fr 2fr 1fr;
    grid-template-rows: 1fr 2fr 1fr;
}
```

![](https://files.mdnice.com/user/20608/2130effe-7a1d-46b5-80b5-f4d81f53a6ca.png)

##### 4.3 grid-template-areas

网格布局允许指定"区域"（area），一个区域由单个或多个单元格组成。 `grid-template-areas` 属性用于定义区域

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-template-areas: 'a b c'
        'd e f'
        'g h i';
}
```

![](https://files.mdnice.com/user/20608/05380980-6eef-4629-bc81-56f4952b5d9c.png)

可以将多个单元格合并成一个区域，其中 `grid-area` 后面会介绍，这里先不作深究

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-template-areas: 'a a b'
        'a a b'
        'c c c';
}

.main div:nth-child(1) {
    grid-area: a;
    background: skyblue;
}
```

![](https://files.mdnice.com/user/20608/a37935c1-f16f-49b5-bb5c-9144fed70d06.png)

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-template-areas: 'a a b'
        'a a b'
        'c c c';
}

.main div:nth-child(1) {
    grid-area: a;
}

.main div:nth-child(2) {
    grid-area: b;
}

.main div:nth-child(3) {
    grid-area: c;
}
```

![](https://files.mdnice.com/user/20608/91474f32-d01b-4b61-b88c-21936bfc5ce2.png)

##### 4.4  grid-template

`grid-template` 属性是 `grid-template-columns` 、 `grid-template-rows` 和 `grid-template-areas` 这三个属性的合并简写形式。但是不建议合并在一起写，所以这里就不作案例展示，毕竟这些属性本身比较难记。可以看MDN上关于 [grid-template](https://developer.mozilla.org/en-US/docs/Web/CSS/grid-template) 相关的介绍

##### 4.5 grid-row-gap，grid-column-gap

`grid-row-gap` 属性设置行与行的间隔（行间距）， `grid-column-gap` 属性设置列与列的间隔（列间距）。需要注意的是，[CSS Grid Layout](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Grid_Layout) 起初是用 [ `grid-row-gap` ](https://developer.mozilla.org/zh-CN/docs/Web/CSS/grid-gap) 属性来定义的，目前逐渐被 `row-gap` 替代。因为其他布局方式也可以使用gap属性，例如 `flex弹性布局` 。但是，为了兼容那些不支持 `row-gap` 属性的浏览器，你需要像上面的例子一样使用带有前缀的属性。详情看MDN关于[row-gap](https://developer.mozilla.org/zh-CN/docs/Web/CSS/row-gap)的介绍

```css
grid-row<column>-gap: length
```

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    row-gap: 20px;
    column-gap: 20px;
}
```

![](https://files.mdnice.com/user/20608/d3a6b2c8-cc5d-4809-8dfa-486656312000.png)

可以看到行与列之间多了20px的间距

![](https://files.mdnice.com/user/20608/714cf1f4-03f6-4e7a-9614-2ef52251f343.png)

##### 4.6 grid-gap

`grid-gap` 属性是 `grid-column-gap` 和 `grid-row-gap` 的合并简写形式，现在已经逐渐被gap替代

```css
grid-gap: <grid-row-gap><grid-column-gap>;
```

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    gap: 50px;
}
```

![](https://files.mdnice.com/user/20608/29b37992-d8db-4018-a623-fca48cde7ef6.png)

##### 4.7 justify-items，align-items

`justify-items` 属性设置单元格内容的水平位置， `align-items` 属性设置单元格内容的垂直位置，默认值都是 `stretch`

```css
justify-items<align-items>: start | end | center | stretch;
```

首先我们先定义一个3行3列的网格

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
}

.main div {
    background: pink;
}
```

![](https://files.mdnice.com/user/20608/586416f6-78d1-46c6-a2df-7b889c6dbb26.png)

为什么在没有给定子项宽高的情况下，仍然可以铺满整个父容器？原因是因为水平与垂直都是默认拉升的（ `stretch` ），也就是：

```
.main div {		justify-items: stretch;		align-items: stretch;}
```

如果我们给子项设置宽高：

```css
.main div {
    width: 50px;
    height: 50px;
}
```

![](https://files.mdnice.com/user/20608/e1092645-5ade-4f30-a08c-ddf62434b3e0.png)

虽然子项的宽高发生了变化，但是实际上网格的大小仍然没有变，用调试工具就可以看出来：

![](https://files.mdnice.com/user/20608/f0321239-9b00-45c1-bdcf-2404c10e6ed4.png)

这个情况下，可以使用 `justify-items` ， `align-items` 来对齐子项的位置

行与列设置居中对齐

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    justify-items: center;
    align-items: center;
}

.main div {
    width: 50px;
    height: 50px;
    background: pink;
}
```

![](https://files.mdnice.com/user/20608/86cba19e-4a5f-4b75-aa63-d7dd99947289.png)

行与列设置尾部对齐

```css
.main {
    justify-items: end;
    align-items: end;
}
```

![](https://files.mdnice.com/user/20608/f6e18c4b-86e8-4eae-a4d7-5eb7a9fc079b.png)

##### 4.8 place-items

`place-items` 属性是 `align-items` 属性和 `justify-items` 属性的合并简写形式，注意的是，垂直对齐是在前，水平对齐是在后。如果只写一个，则默认垂直与水平是一致。

```css
place-items: <align-items><justify-items>;
```

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    place-items: center;
}

.main div {
    width: 50px;
    height: 50px;
    background: pink;
}
```

![](https://files.mdnice.com/user/20608/e6b1ef77-8a48-47ee-8c54-394f6fd97e56.png)

##### 4.9 justify-content ， align-content

`justify-content` 属性是整个内容区域在容器里面的水平位置， `align-content` 属性是整个内容区域的垂直位置。默认都是 `start`

```css
justify-content<align-content>: start | end | center | stretch | space-around | space-between | space-evenly;
```

首先定义一个3行3列的网格布局

```css
.main {
    width: 500px;
    height: 500px;
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
}

.main div {
    background: pink;
}
```

![](https://files.mdnice.com/user/20608/0ee0daa8-59ca-4ebf-b7b5-4d1a83aae9a5.png)

向结束位置对齐

```css
.main {
    justify-content: end;
    align-content: end;
}
```

![](https://files.mdnice.com/user/20608/57f61592-a351-40a3-8072-b23359bb0721.png)

居中对齐

```css
.main {
    justify-content: center;
    align-content: center;
}
```

![](https://files.mdnice.com/user/20608/c4957359-cc9b-484a-8075-4a08d129117c.png)

两端对齐

```
.main{  	justify-content: space-between;  	align-content: space-between;}
```

![](https://files.mdnice.com/user/20608/e2b09ebd-95c9-4130-ae1c-b0bb997e853e.png)

子项与子项之间间隔相等

```css
.main {
    justify-content: space-around;
    align-content: space-around;
}
```

![](https://files.mdnice.com/user/20608/f1ed6ccb-6702-453e-86e3-6b2cb089a27f.png)

子项与子项之间间隔相等，子项与容器边框之间间隔相等

```css
.main {
    justify-content: space-evenly;
    align-content: space-evenly;
}
```

![](https://files.mdnice.com/user/20608/378c5fc7-6f25-44c7-8fc5-406a94578624.png)

其他情况不在此继续展示，可自行尝试

##### 4.10 place-content

`place-content` 属性是 `align-content` 属性和 `justify-content` 属性的合并简写形式。和 `place-items` 一样，垂直在前，水平在后。如果只写一个，则默认垂直与水平是一致。

```css
place-content: <align-content><justify-content>
```

```css
.main {
    place-content: center;
}
```

![](https://files.mdnice.com/user/20608/437bda4c-2947-42ea-9a66-a905277ecc93.png)

##### 4.11  justify-items，align-items 与  justify-content，align-content 的注意点

这两对属性和 `flex弹性布局` 很相似，所以经常搞混，这里特别要说明的是，与 `flex布局` 不一样的地方是，这两对属性都是作用在父容器上，而 `flex布局` 的 `justify-content` ， `align-items` 属性是作用在父容器上的，但是没有 `justify-items` ， `align-content` 这两个属性。一般在 `Grid布局` 中， `justify-items` ， `align-items` 是一起使用， `justify-content` ， `align-content` 是一起使用

那么什么时候使用 `justify-items` ， `align-items` ，什么时候使用 `justify-content` ， `align-content` 呢？

可以记住一个技巧：

###### （1）当子项的大小低于单元格的大小的时候，使用 `justify-items` ， `align-items`

还记得前面说的吗，子项的大小是可能会比单元格小的，例如单元格大小是 `100*100` ，而子项大小只有 `50*50` 。这种情况使用 `justify-items` ， `align-items` 。

![](https://files.mdnice.com/user/20608/f0321239-9b00-45c1-bdcf-2404c10e6ed4.png)

###### （2）当容器大小大于单元格大小时，使用 `justify-content` ， `align-content`

![](https://files.mdnice.com/user/20608/0ee0daa8-59ca-4ebf-b7b5-4d1a83aae9a5.png)

##### 4.12 显示网格与隐式网格

正常情况下，我们定义了 例如三行三列的网格，总共有 `9` 个元素，那这 `9` 个元素正好布满整个容器，这 `9` 个子项都是 `显示网格`

```css
.main {
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
}

.main div {
    background: pink;
}
```

![](https://files.mdnice.com/user/20608/e4d4caa7-8d5c-4992-a918-8e6ce09e0f2f.png)

如果有超过 `9` 个子项呢？比如，数字 `10` ，数字 `11` ，数字 `12` .....？？这些多出来的网格就是 `隐式网格` ，这些隐式网格默认都是按照 `rows` 的方向进行排列，高度会自动 `拉伸` 铺满容器

![](https://files.mdnice.com/user/20608/be209e4a-1487-40cc-9cfb-252c8702734f.png)

##### 4.13 grid-auto-flow

明白了什么是 `显示网格` 和 `隐式网格` ，就可以看一下 ` grid-auto-flow` 。划分网格以后，容器的子元素会按照顺序，自动放置在每一个网格。默认的放置顺序是"先行后列"，即先填满第一行，再开始放入第二行。这就解释了，上面的 `隐式网格` 为什么会在 `789` 的下面，明明父容器右侧还有空白位置，为什么无法铺满，这是因为默认设置了"先行后列"，即 `grid-auto-flow: rows` ，这个属性会决定网格的排列顺序

```css
grid-auto-flow: rows | column | ros dense | column dense;
```

将顺序设置为先列后行，即 `grid-auto-flow: column`

```css
.main {
    width: 500px;
    height: 500px;
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-auto-flow: column;
}

.main div {
    background: pink;
}
```

![](https://files.mdnice.com/user/20608/a6a665e2-397a-4b37-b7c2-815785ef71ca.png)

如果将网格 `1` 和网格 `2` 各占据两个单元格，然后在默认 `grid-auto-flow: row` 情况下，会产生下面这样的布局

![](https://files.mdnice.com/user/20608/44ac82e1-06d1-4ac7-9005-685bd6e60e02.png)

上图中，网格 `1` 和网格 `2` 后面的位置都空了出来，因为网格 `3` 的排序是默认跟着网格 `2` 的，设置 `grid-auto-flow: row dense ` ，可以将空出的位置铺满

```css
.main {
    width: 300px;
    height: 400px;
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-auto-flow: row dense;
}
```

![](https://files.mdnice.com/user/20608/5aece1e3-0e56-4cd5-a325-75bbc0cf290a.png)

如果是"先列后行"的情况

```css
.main {
    width: 500px;
    height: 300px;
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-auto-flow: column
}
```

##### 4.14 grid-auto-rows，grid-auto-columns

如果是先行后列的顺序， `隐式网格` 的列宽默认和 `显示网格` 一样，行高则是拉伸铺满，如果是先列后行的顺序， `隐式网格` 的行高默认和 `显示网格` 一样。难道 `隐式网格` 的高度或者宽度只能拉升吗？当然不是，可以使用 `grid-auto-rows` ， `grid-auto-columns` 控制 `隐式网格` 的行高和列宽

先行后列的顺序下，设置 `隐式网格` 行高为 `50px`

```css
.main {
    width: 500px;
    height: 300px;
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-auto-flow: rows;
    grid-auto-rows: 50px;
}
```

![](https://files.mdnice.com/user/20608/3b775126-ebd8-4004-a54c-cc27be8de151.png)

先列后行的顺序下，设置 `隐式网格` 列高为 `50px`

```css
.main {
    width: 500px;
    height: 300px;
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-auto-flow: column;
    grid-auto-columns: 50px;
}
```

![](https://files.mdnice.com/user/20608/ab4db9c7-a99d-442a-be88-cb2762452cea.png)

将 `隐式网格` 的行高也设置成 `100px` ，这样就和 `显示网格` 展示的大小一样了

```css
.main {
    width: 500px;
    height: 400px;
    background: skyblue;
    display: grid;
    grid-template-columns: 100px 100px 100px;
    grid-template-rows: 100px 100px 100px;
    grid-auto-rows: 100px;
}
```

![](https://files.mdnice.com/user/20608/10c95c95-8ee6-4b14-97d9-c959fbbf181e.png)

##### 4.15 repeat()方法与auto-fill关键字

我们在定义网格的时候，写 `grid-template-columns` 和 `grid-template-rows` 都是写多个重复值，比如 `grid-template-columns: 100px 100px 100px` ，简单的网格可以这样定义，但是如果网格很多的时候，难道要这样写吗？ `grid-template-columns: 100px 100px 100px 100px 100px ......` ，这样很麻烦，可以使用 `repeat()` 去简化重复的值

```css
repeat(number, length | percent | fr)
```

定义一个5行4列的网格

```css
.main {
    display: grid;
    grid-template-columns: repeat(5, 100px);
    grid-template-rows: repeat(4, 100px);
}
```

![](https://files.mdnice.com/user/20608/f4a5ecaf-0229-44b7-aeae-db3809d81812.png)

也可以具体的数值和 `repeat()函数` 一起使用

```css
.main {
    display: grid;
    grid-template-columns: 200px repeat(2, 100px);
    grid-template-rows: 200px repeat(2, 100px);
}
```

![](https://files.mdnice.com/user/20608/0d2bc7fa-7bcc-423d-895f-f70127b25f41.png)

还有一种情况，容器的宽度有时候可以容纳更多的网格，但是由于定义了列数量或者行的数量比较少时

```css
.main {
    width: 500px;
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
}
```

![](https://files.mdnice.com/user/20608/b588c8de-3e35-4291-9d62-4f2d668f868a.png)

明明 `网格4` 是可以继续在 `网格3` 的后面继续排列的，为什么需要换行呢，仅仅是因为使用了 `repeat(3, 100px)` 定义三列，有什么办法，在单元格的大小是固定的，但是容器的大小不确定的情况下，使得每一行（或每一列）容纳尽可能多的单元格，这时可以使用 `auto-fill` 关键字表示自动填充

```css
.main {
    width: 500px;
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(auto-fill, 100px);
    grid-template-rows: repeat(3, 100px);
}
```

![](https://files.mdnice.com/user/20608/14eb815b-a782-493e-a9dd-06fe253f5eec.png)

##### 4.16 minmax()方法

`minmax()` 函数产生一个长度范围，表示长度就在这个范围之中。它接受两个参数，分别为最小值和最大值。

```css
minmax(min, max)
```

定义一个列宽和行高不小于 `100px` ，不大于 `200px` 的网格

```css
.main {
    display: grid;
    grid-template-columns: minmax(100px, 200px) minmax(100px, 200px) minmax(100px, 200px);
    grid-template-rows: minmax(100px, 200px) minmax(100px, 200px) minmax(100px, 200px);
}
```

![](https://files.mdnice.com/user/20608/bf4f0ea4-98bf-4fa9-835b-f4267bfff342.png)

 `minmax()方法可以和repeat()方法一起使用`

```css
.main {
    display: grid;
    grid-template-columns: repeat(3, minmax(100px, 150px));
    grid-template-rows: 100px 100px 100px;
}
```

![](https://files.mdnice.com/user/20608/f3495034-f450-4d4a-9f39-de4423ee553c.png)

#### 五、项目属性

##### 5.1 grid-area

首先定义一个三行三列的网格

![](https://files.mdnice.com/user/20608/6b05e5ae-2d2d-4f4e-be1f-4df152151a57.png)

默认情况下，网格数字都是按顺序排列的，有什么办法可以指定位置摆放呢？ `grid-area` 属性就可以指定项目放在哪一个区域，但前提是需要先用 `grid-template-areas` 属性指定了区域

将 `网格1` 放到 `区域e`

```css
.main {
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
    grid-template-areas: 'a b c'
        'd e f'
        'g h i';
}

.main div:nth-child(1) {
    grid-area: e;
    background: chartreuse;
}
```

![](https://files.mdnice.com/user/20608/e691f6a9-12bd-4994-b3cb-bde0c4dfa09a.png)

##### 5.2 grid-column-start ， grid-column-end  和 grid-row-start ， grid-row-end 与span关键字

这两对属性表示grid子项占据的区域的起始和终止位置，具体方法就是指定项目的四个边框，分别定位在哪根网格线

* `grid-column-start`属性：左边框所在的垂直网格线
* `grid-column-end`属性：右边框所在的垂直网格线
* `grid-row-start`属性：上边框所在的水平网格线
* `grid-row-end`属性：下边框所在的水平网格线

如何查看所在的网格线？可以利用浏览器的调试工具，这样就可以展示网格线的序号了

![](https://files.mdnice.com/user/20608/2e31d5ce-b885-4cbc-961d-d60500c7bf43.png)

`grid-column-start` 和 `grid-column-end`

```css
.main {
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
}

.main div {
    background: pink;
    grid-column-start: 1;
    grid-column-end: 3;
}
```

![](https://files.mdnice.com/user/20608/7a92d1cd-b45a-4ebe-b88b-331d19cb94f2.png)

`grid-row-start` 和 `grid-row-end`

```css
.main {
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
}

.main div {
    background: pink;
    grid-row-start: 2;
    grid-row-end: 3;
}
```

![](https://files.mdnice.com/user/20608/0759812c-ff49-4aab-b18b-158f3d31a81d.png)

两对属性一起使用

```css
.main div {
    grid-column-start: 2;
    grid-column-end: 3;
    grid-row-start: 2;
    grid-row-end: 3;
}
```

![](https://files.mdnice.com/user/20608/aa5f3989-9e80-4b71-9e8e-330724396ebb.png)

除了使用这种方式外，还可以直接命名网格线，效果也是一样的，使用调试工具可以查看网格线的命名

```css
.main {
    background: skyblue;
    display: grid;
    grid-template-columns: [col1] 100px [col2] 100px [col3] 100px [col4];
    grid-template-rows: [row1] 100px [row2] 100px [row3] 100px [row4];
}

.main div {
    grid-column-start: col2;
    grid-column-end: col3;
    grid-row-start: row2;
    grid-row-end: row3;
}
```

![](https://files.mdnice.com/user/20608/c63d5abc-c5fc-47ed-bcd8-e4d5be87a6a0.png)

这两对属性还可以使用 `span` 关键字，表示"跨越"，即左右边框（上下边框）之间跨越多少个网格

```css
grid-column<row>-start: span number;
```

```css
.main div {
    grid-column-start: span 2;
}
```

![](https://files.mdnice.com/user/20608/adfce37c-36da-43f7-85a6-21fc801eba7b.png)

```css
.main div {
    grid-column-start: 2;
    grid-column-end: span 2;
}
```

![](https://files.mdnice.com/user/20608/c91695d9-d7a2-4a17-a4f7-2b8eb42eb015.png)

##### 5.3 grid-row，gird-column

`grid-column` 属性是 `grid-column-start` 和 `grid-column-end` 的合并简写形式， `grid-row` 属性是 `grid-row-start` 属性和 `grid-row-end` 的合并简写形式

```css
grid-column: <start-line>/ <end-line>;
grid-row: <start-line>/ <end-line>;
```

```css
.item-1 {
    grid-column: 1 / 3;
    grid-row: 1 / 2;
}

/* 等同于 */
.item-1 {
    grid-column-start: 1;
    grid-column-end: 3;
    grid-row-start: 1;
    grid-row-end: 2;
}
```

这两个属性之中，也可以使用 `span` 关键字，表示跨越多少个网格

```css
.item-1 {
    grid-column: 1 / span 2;
    grid-row: 1 / span 2;
}

/* 等同于 */
.item-1 {
    grid-column: 1 / 3;
    grid-row: 1 / 3;
}
```

斜杠以及后面的部分可以省略，默认跨越一个网格

```css
.item-1 {
    grid-column: 1;
    grid-row: 1;
}
```

用 `grid-area` 也可以实现同样的效果

```css
grid-area: grid-row-start / grid-column-start / grid-row-end / grid-column-end;
```

```css
.main {
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
}

.main div {
    background: pink;
    grid-area: 2 / 2 / 3 / 3;
}
```

![](https://files.mdnice.com/user/20608/9ec062e8-0c50-41f8-8be7-0fc66c2bcf58.png)

##### 5.4 justify-self，align-self

`justify-self` 属性设置单元格内容的水平位置（左中右），跟 `justify-items` 属性的用法完全一致，但只作用于单个项目

`align-self` 属性设置单元格内容的垂直位置（上中下），跟 `align-items` 属性的用法完全一致，也是只作用于单个项目

```css
justify-self: start | end | center | stretch;
align-self: start | end | center | stretch;
```

* start：对齐单元格的起始边缘
* end：对齐单元格的结束边缘
* center：单元格内部居中
* stretch：拉伸，占满单元格的整个宽度（默认值）

```css
.main {
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
}

.main div {
    width: 50px;
    height: 50px;
    background: pink;
    grid-area: 2 / 2 / 3 / 3;
    justify-self: end;
    align-self: end;
}
```

![](https://files.mdnice.com/user/20608/2e12e477-33fa-431c-b0cc-a7d1118ed529.png)

##### 5.5 place-self

`place-self` 属性是 `align-self` 属性和 `justify-self` 属性的合并简写形式。如果省略第二个值， `place-self` 属性会认为这两个值相等

```css
place-self: <align-self><justify-self>;
```

```css
.main {
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(3, 100px);
    grid-template-rows: repeat(3, 100px);
}

.main div {
    width: 50px;
    height: 50px;
    background: pink;
    grid-area: 2 / 2 / 3 / 3;
    place-self: center;
}
```

![](https://files.mdnice.com/user/20608/6e161d78-fcb5-4128-918b-93344400bff0.png)

#### 六、布局案例

##### 6.1 栅格布局

利用 `grid布局` 可以轻松实现 `栅格布局`

```css
.row {
    background: skyblue;
    display: grid;
    grid-template-columns: repeat(12, 1fr);
    grid-template-rows: 50px;
    grid-auto-rows: 50px;
}

.row div {
    background: pink;
    border: 1px black solid;
}

.row .col-1 {
    grid-area: auto/auto/auto/span 1;
}

.row .col-2 {
    grid-area: auto/auto/auto/span 2;
}

.row .col-3 {
    grid-area: auto/auto/auto/span 3;
}

.row .col-4 {
    grid-area: auto/auto/auto/span 4;
}

.row .col-5 {
    grid-area: auto/auto/auto/span 5;
}

.row .col-6 {
    grid-area: auto/auto/auto/span 6;
}

.row .col-7 {
    grid-area: auto/auto/auto/span 7;
}

.row .col-8 {
    grid-area: auto/auto/auto/span 8;
}

.row .col-9 {
    grid-area: auto/auto/auto/span 9;
}

.row .col-10 {
    grid-area: auto/auto/auto/span 10;
}

.row .col-11 {
    grid-area: auto/auto/auto/span 11;
}

.row .col-12 {
    grid-area: auto/auto/auto/span 12;
}
```

```html
<div class="row">
    <div class="col-6">1</div>
    <div class="col-3">2</div>
    <div class="col-4">3</div>
    <div class="col-5">4</div>
</div>
```

![](https://files.mdnice.com/user/20608/1ba04241-bfe2-4d7b-960d-69b4743d2ccd.png)

##### 6.2 不规则子项排列

如果要实现这样的效果，使用 `grid布局` 也是轻而易举实现的

![](https://files.mdnice.com/user/20608/10f11816-b80e-4c93-bb7a-e8054c2871dc.png)

```css
.main {
    width: 308px;
    margin: 20px auto;
}

.main-list {
    height: 352px;
    margin: 0 14px;
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(4, 1fr);
    grid-template-areas:
        "a1 a3 a3"
        "a2 a3 a3"
        "a4 a4 a5"
        "a6 a7 a7";
    gap: 8px;
}

.main-list div:nth-of-type(1) {
    grid-area: a1;
}

.main-list div:nth-of-type(2) {
    grid-area: a2;
}

.main-list div:nth-of-type(3) {
    grid-area: a3;
}

.main-list div:nth-of-type(4) {
    grid-area: a4;
}

.main-list div:nth-of-type(5) {
    grid-area: a5;
}

.main-list div:nth-of-type(6) {
    grid-area: a6;
}

.main-list div:nth-of-type(7) {
    grid-area: a7;
}

.main-list a {
    width: 100%;
    height: 100%;
    display: block;
    line-height: 30px;
}

.main-list h3 {
    text-align: right;
    margin-right: 4px;
}

.main-list p {
    text-align: center;
}

.theme1 {
    background: pink;
}

.theme2 {
    background: skyblue;
}

.theme3 {
    background: orange;
}
```

```html
<div class="main">
    <div class="main-list">
        <div class="theme1"></div>
        <div class="theme2"></div>
        <div class="theme1"></div>
        <div class="theme1"></div>
        <div class="theme1"></div>
        <div class="theme3"></div>
        <div class="theme3"></div>
    </div>
</div>
```

![](https://files.mdnice.com/user/20608/6bb84931-4ee0-4299-92ea-09cfcb6908f9.png)

##### 6.3 更多布局案例

[Grid by Example](https://gridbyexample.com/)

#### 七、总结

`grid布局` 和 `flex弹性布局` 一样，都是当下最流行的CSS布局方案之一。它的优点是可以实现多行多列的布局，属于 `二维布局` ，基本可以满足任何的布局页面。

优点：

* 固定和灵活的轨道尺寸
* 可以使用行号、名称或通过定位网格区域将项目放置在网格上的精确位置
* 可以将多个项目放入网格单元格或区域中，它们可以彼此部分重叠

缺点：

* 浏览器兼容性较差

* 学习成本较高

  

`grid布局` 可以说是目前最强大的CSS布局方案，在实际开发过程中，往往 `grid布局` 和 `flex布局` 一起结合使用。

#### 八、参考

* [MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS/grid)
* [阮一峰：Grid网格布局教程](https://www.ruanyifeng.com/blog/2019/03/grid-layout-tutorial.html) 
* [A Complete Guide to Grid](https://css-tricks.com/snippets/css/complete-guide-grid/) 
* [caniuse.com](https://caniuse.com/?search=grid)
* [Grid by Example](https://gridbyexample.com/)
