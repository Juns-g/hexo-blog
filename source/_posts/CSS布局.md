---
title: CSS布局
tags:
  - CSS
  - 学习
  - 前端
categories: 前端
keywords: 前端 CSS 布局
cover: 'https://w.wallhaven.cc/full/1p/wallhaven-1pd1o9.jpg'
abbrlink: 45816
date: 2024-02-13 17:26:43
---

# CSS 布局

## 前言

在 CSS 中我们可以给元素添加样式，但是元素之间的排列方式是由布局来决定的。CSS 中也有很多种布局方式，下面了解一下常见的布局方式，不过首先了解俩个概念。

## 盒子模型

页面中的每个元素都可以看作一个盒子，也就叫做盒子模型。

### 分类

**W3C 标准盒子模型**和**IE 怪异盒子模型**

如果说指定了`width: 100px`，那么在 W3C 标准盒子模型中，`width`指的是内容(content)的宽度，而在 IE 怪异盒子模型中，`width`指的是内容 + padding + border 的宽度。

```css
box-sizing: content-box; /* W3C 标准盒子模型 */
box-sizing: border-box; /* IE 怪异盒子模型 */
```

### 获取盒子模型的宽高

- `offsetWidth`：获取盒子模型的宽度，包括`content`、`padding`、`border`。
- `offsetHeight`：获取盒子模型的高度，包括`content`、`padding`、`border`。
- `clientWidth`：获取盒子模型的宽度，包括`content`、`padding`，不包括`border`。
- `clientHeight`：获取盒子模型的高度，包括`content`、`padding`，不包括`border`。
- `element.getBoundingClientRect()`：获取盒子模型的宽高、距离视口的距离。

## BFC

BFC, Block Formatting Context, 块级格式化上下文。
他指的是一个**隔离的独立的块级渲染区域**。

https://tsejx.github.io/css-guidebook/layout/bfc

### 触发条件

满足任意一条就可以了

1. 根元素默认创建 BFC
2. 浮动元素（元素 float 不为 none）
3. 绝对定位元素（元素 position 为 absolute 或 fixed）
4. 行内块元素（元素 display 为 inline-block）
5. 表格单元格（元素 display 为 table-cell，HTML 表格单元格默认为该值）
6. 表格标题（元素 display 为 table-caption，HTML 表格标题默认该值）
7. 匿名表格单元格元素（display 值是 table、table-row、table-row-group、table-header-group、table-footer-group）
8. overflow 值不为 visible
9. 弹性元素（display 为 flex 或 inline-flex 元素的直接子元素）
10. 网格元素（display 为 grid 或 inline-grid 元素的直接子元素）

### 使用场景

#### 解决外边距合并问题

当两个元素相邻时，并且都设置了 margin，那么他们的外边距会合并，这时候可以给其中一个盒子创建 BFC 来解决。

折叠规则：

- 如果两个外边距都是**正数**，那么取其中**较大的值**。
- 如果两个外边距都是**负数**，那么取其中**绝对值较大的值**。
- 如果两个外边距一个是**正数**一个是**负数**，那么取**两者的和**。

出现这个问题的原因：俩个相邻元素处于同一个 BFC 中，所以他们的外边距会合并。

#### 清除浮动

浮动的元素会脱离文档流，如果父元素没有设置高度，那么父元素的高度会塌陷，这时候可以给父元素创建 BFC 来解决。

```html
<div class="bfc-clear-float">
  <div class="child"></div>
</div>
<style>
  .bfc-clear-float {
    width: 100px;
    border: 1px solid #000;
    overflow: hidden; /* 触发 BFC */
  }

  .child {
    float: left;
    width: 50px;
    height: 50px;
    background-color: skyblue;
  }
</style>
```

#### 多列布局设定

最后一列可能掉到下一行，可以给最后一列创建 BFC。

```html
<div class="container">
  <div class="column">Column 1</div>
  <div class="column">Column 2</div>
  <div class="column">Column 3</div>
</div>

<style>
  .column {
    width: 31.33%;
    background-color: green;
    float: left;
    margin: 0 1%;
  }
  .column:last-child {
    float: none;
    overflow: hidden;
  }
</style>
```

## 基础布局

### 弹性布局

在传统的布局方式中，block 布局是把块在垂直方向从上到下依次排列的；而 inline 布局则是在水平方向来排列。弹性盒子布局并没有这样内在的方向限制，可以自由操作。

> 注意：设为弹性布局之后，子项的 float、clear 和 vertical-align 属性将失效。

#### 弹性盒子的轴

容器默认有两根轴，主轴和交叉轴。flex 的子项默认沿主轴排列，可以通过`flex-direction`属性改变主轴的方向。

#### 容器属性

display: flex | inline-flex

flex 是设置为弹性容器，默认占父元素的 100%
inline-flex 是设置为行内弹性容器，宽高自适应

##### flex-direction

决定主轴的方向

- row（默认值）：表示子项从左向右排列。此时水平方向轴为- 主轴。
- row-reverse：表示子项从右向左排列。
- column：表示子项从上向下排列。此时垂直方向轴为主轴。
- column-reverse：表示子项从下向上排列。

##### flex-wrap

决定子项是否换行，使用时需要给容器添加固定宽度

- nowrap（默认值）：表示不换行，所有子项目单行排列，子项可能会溢出。
- wrap：表示换行，所有子项目多行排列，溢出的子项会被放到下一行，按从上向下顺序排列。
- wrap-reverse：所有子项目多行排列，按从下向上顺序排列。

##### flex-flow

是 flex-direction 属性和 flex-wrap 属性的简写形式，默认值为 row nowrap。

##### justify-content

决定子项在主轴上的对齐方式

- flex-start（默认值）：子项按主轴起点线对齐
- flex-end：子项按主轴终点线对齐
- center： 居中排列
- space-between：子项均匀分布，第一项紧贴主轴起点线，最后一项紧贴主轴终点线，子项目之间的间隔都相等。要注意特殊情况，当剩余空间为负数或者只有一个项时，效果等同于 flex-start。
- space-around：子项均匀分布，每个项目两侧的间隔相等，相邻项目之间的距离是两个项目之间留白的和。所以，项目之间的间隔比项目与边框的间隔大一倍。要注意特殊情况，当剩余空间为负数或者只有一个项时，效果等同于 center。
- space-evenly：子项均匀分布，所有项目之间及项目与边框之间的距离相等。

##### align-items

决定子项在交叉轴上的对齐方式

- stretch（默认值）：当子项未设置高度或者高度为 auto 时，子项的高度设为行高。这里需要注意，在只有一行的情况下，行的高度为容器的高度，即子项高度为容器的高度。（当子项设定了高度时无法展开）
- flex-start：表示子项与交叉轴的起点线对齐。
- flex-end：表示子项与交叉轴的终点线对齐。
- center：表示与交叉轴的中线对齐。
- baseline：表示基线对齐，当行内轴与侧轴在同一线上，即所有子项的基线在同一线上时，效果等同于 flex-start。

##### align-content

他定义的是多根交叉轴线对的对齐方式，也就是换行之后出现了很多行才有效。
属性值基本同 align-items

#### 子项属性

##### order

子项的排列顺序，数值越小越靠前，默认为 0。

##### flex-grow

决定子项的放大比例，默认为 0，即如果存在剩余空间，也不放大。

##### flex-shrink

决定子项的缩小比例，默认为 1，即如果空间不足，该项目将缩小。

##### flex-basis

决定了子项在分配多余空间之前的初始大小，默认为 auto，即由子项的宽度和高度决定。

##### flex

flex 属性是 flex-grow、flex-shrink 和 flex-basis 的简写，默认值为 0 1 auto。后两个属性可选。

##### align-self

允许单个子项与其他子项不一样的对齐方式，属性值同 align-items。

#### 总结

定义为弹性布局

```css
.container {
  display: flex;
}

/* 行内flex */
.item {
  display: inline-flex;
}
```

容器样式

```css
.container {
    flex-direction: row | row-reverse | column | column-reverse;
    /* 主轴方向: 左到右（默认）| 右到左 | 上到下 | 下到上 */

    flex-wrap: nowrap | wrap | wrap-reverse;
    /* 换行: 不换行（默认）| 换行 | 换行并第一行在下方 */

    flex-flow: <flex-direction> | <flex-wrap>
    /* 主轴方向和换行简写 */

    justify-content: flex-start | flex-end | center | space-between | space-around;
    /* 主轴对齐方式: 左对齐（默认） | 右对齐 | 居中对齐 | 两端对齐 | 平均分布 */

    align-items: flex-start | flex-end | center | baseline | stretch;
    /* 交叉轴对齐方式: 顶部对齐（默认）| 底部对齐 | 居中对齐 | 上下对齐并铺满 | 文本基线对齐 */

    align-content: flex-start | flex-end | center | space-between | space-around | stretch;
    /* 多主轴对齐: 顶部对齐（默认）| 底部对齐 | 居中对齐 | 上下对齐并铺满 | 上下平均分布*/
}
```

子项样式

```css
.item {
    order: <integer>;
    /* 排序: 数值越小,排越前,默认为0 */

    flex-grow: <number>; /* default 0*/
    /* 放大: 默认0（即如果有剩余空间也不放大,值为1则放大,2是1的双倍大小,以此类推）*/

    flex-shrink: <number>; /* default 1*/
    /* 缩小: 默认1（如果空间不足则会缩小,值为0不缩小）*/

    flex-basis: <length> | auto; /* default auto */
    /* 固定大小: 默认为0,可以设置px值,也可以设置为百分比大小 */

    flex: none | [<'flex-grow'> <'flex-shrink'>? || <'flex-basis'>]
    /* flex-grow, flex-shrink 和 flex-basis 的简写,默认值为0 1 auto */

    align-self: auto | flex-start | flex-end | center | baseline | stretch;
    /* 单独对齐方式: 自动（默认）| 顶部对齐 | 底部对齐 | 居中对齐 | 上下对齐并铺满 | 文本基线对齐 */
}
```

## 居中布局

### 水平居中

#### 内联元素

直接`text-align: center`

适用对象：

- 内联元素 line
- 内联块 inline-block
- 内联表 inline-table
- inline-flex 元素

优点: 简单快捷，兼容性好

缺点:

- 只对行内内容有效
- 属性会继承影响到后代行内内容
- 如果子元素宽度大于父元素宽度则无效，但是后代行内内容中宽度小于设置 text-align 属性的元素宽度的时候，也会继承水平居中

#### 块级元素

可以水平方向的 margin 为 auto

```css
.child {
  margin: 0 auto;
}
```

#### 多级块元素

##### 内联块实现

```html
<div class="parent">
  <div class="child">child1</div>
  <div class="child">child2</div>
</div>
<style>
  .parent {
    text-align: center;
  }
  .child {
    display: inline-block;
  }
</style>
```

优点: 简单易理解，兼容性好

缺点:

- 只对行内内容有效
- 属性会继承影响到后代行内内容
- 块级改为 inline-block 换行、空格会产生元素间隔

##### 弹性布局

```css
.parent {
  display: flex;
  justify-content: center;
}
```

##### margin 偏移 / transform

```css
.parent {
  position: relative;
}
.child{
  position: absolute:
  left: 50%;
  transform: translateX(-50%);
  /* 或者 */
  margin-left: -50%;
}
```

##### 外边距适配

```css
.parent {
  position: relative;
}
.child {
  position: absolute;
  left: 0;
  right: 0;
  margin: 0 auto;
}
```

### 垂直居中

#### 单行内联元素

设置`line-height`等于父元素的`height`

```css
.parent {
  height: 100px;
  line-height: 100px;
}
```

#### 多行元素

##### 表格布局

```css
.parent {
  display: table;
  height: 100px;
}

.child {
  display: table-cell;
  vertical-align: middle;
}
```

##### 弹性布局

```css
.parent {
  display: flex;
  flex-direction: column;
  justify-content: center;
}
```

#### 块级元素

- margin 偏移 / transform
- top 和 bottom 为 0，垂直 margin: auto

#### 图片

```html
<div class="parent">
  <img class="child" />
</div>
<style>
  .parent {
    height: 100px;
    line-height: 100px;
    font-size: 0;
  }

  .child {
    width: 50px;
    height: 50px;
    vertical-align: middle;
  }
</style>
```

在 font-size 不为 0 时，图片设置垂直居中时的中线位置并不是它的父元素的绝对中线位置，它们会有一个和字符 x 高度相关的高度差。设置 font-size 为 0，会消除这个高度差，使图片的中线位置和父元素中线重合。

### 水平垂直居中

实现方案：

1. 文本元素：`text-align: center` + `line-height: height` + `vertical-align: middle`
2. 固定宽高的元素：相对于父元素容器 margin left 和 right 偏移 50%，通过计算垂直居中元素宽/高，利用负 margin 偏移自身宽/高的 50%。或者通过设置上下左右坐标为 0，外边距自适应 auto 实现垂直居中。两种处理手法都需要设置垂直居中元素为标准盒模型。
3. 不确定垂直居中元素的宽高：相对于父元素容器左边距和上边距坐标偏移 50%，使用 transform + translate 将垂直居中元素自身偏移负 50%，使用标准盒模型
4. 弹性布局：父元素设置为弹性布局容器，并将 justify-content 和 align-items 设置为 center 居中
5. 网格布局：父元素设置为网格布局容器，垂直居中元素设置外边距 margin 为自适应 auto 即可

#### 垂直居中文本

```html
<div class="base parent">
  <div class="child">我是文本</div>
</div>

<style>
  .base {
    height: 100px;
    width: 100px;
    background-color: gray;
  }
  .parent {
    line-height: 100px;
    text-align: center;
  }
  .child {
    display: inline-block;
    vertical-align: middle;
  }
</style>
```

#### 固定宽高

负 margin 或者外边距自适应

```html
<div class="base parent">
  <div class="child">我是文本</div>
</div>

<style>
  .base {
    height: 100px;
    width: 100px;
    background-color: gray;
  }
  .parent {
    position: relative;
  }
  .child {
    background-color: red;
    position: absolute;
    width: 50px;
    height: 50px;
    left: 50%;
    top: 50%;
    margin-top: calc(-0.5 * 50px);
    margin-left: calc(-0.5 * 50px);
  }
</style>
```

```html
<div class="base parent">
  <div class="child">我是文本</div>
</div>

<style>
  .base {
    height: 100px;
    width: 100px;
    background-color: gray;
  }
  .parent {
    position: relative;
  }
  .child {
    background-color: red;
    position: absolute;
    width: 50px;
    height: 50px;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    margin: auto;
  }
</style>
```

#### 未知宽高

绝对定位+transform

```css
.parent {
  position: relative;
}

.child {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```

#### 弹性布局

```css
.parent {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

#### 网格布局

```css
.parent {
  display: grid;
}
.child {
  margin: auto;
}
```

#### 可视窗口的水平垂直居中

需要保证一下兼容性 ☹️

```css
.outer {
  display: table;
  position: absolute;
  height: 100%;
  width: 100%;
}

.middle {
  display: table-cell;
  vertical-align: middle;
}

.inner {
  margin: 0 auto;
  width: 400px;
}
```

## 多栏布局

### 双栏布局

左固定右自适应

#### float + margin

原理：定宽元素浮动脱离文档流，自适应元素左侧边距>=定宽元素的宽度

```css
.left {
  float: left;
  width: 25%; /* 左度固定 */
}
.right {
  margin-left: 30%;
}
```

#### float + calc

```css
.left {
  float: left;
  width: 200px;
}
.right {
  float: right;
  width: calc(100% - 200px);
}
```

#### float + overflow

```css
.left {
  float: left;
  width: 200px;
}
.right {
  overflow: hidden;
}
```

#### table

缺点多

```css
.container {
  display: table;
  width: 100%;
}

.left {
  width: 25%; /* 左宽度固定 */
}

.left,
.right {
  display: table-cell;
  text-align: center;
}
```

#### position

```css
.container {
  position: relative;
}
.left {
  position: absolute;
  left: 0;
  top: 0;
  width: 200px;
}
.right {
  position: absolute;
  right: 0;
  top: 0;
  margin-left: 200px;
}
```

#### flex

```css
.container {
  display: flex;
}
.left {
  width: 200px;
}
.right {
  flex: 1;
}
```

#### grid

```css
.container {
  display: grid;
  grid-template-columns: 200px auto;
  width: 100%;
}
```

### 三栏布局

左右固定，中间自适应

#### float

需要把中间的放到末尾

```html
<div class="container">
  <div class="left">left</div>
  <div class="right">right</div>
  <div class="center">center</div>
</div>

<style>
  .container {
    overflow: hidden;
  }
  .left {
    float: left;
    width: 100px;
  }
  .right {
    float: right;
    width: 200px;
  }
  .center {
    margin-left: 100px;
    margin-right: 200px;
  }
</style>
```

#### 绝对定位

```html
<div class="container">
  <div class="left">left</div>
  <div class="center">center</div>
  <div class="right">right</div>
</div>

<style>
  .left {
    width: 100px;
    position: absolute;
    left: 0;
  }
  .right {
    width: 200px;
    position: absolute;
    right: 0;
  }
  .center {
    position: absolute;
    left: 100px;
    right: 200px;
  }
</style>
```

#### 表格布局

```css
.container {
  display: table;
  width: 100%;
}
.left {
  width: 100px;
  display: table-cell;
}
.right {
  width: 200px;
  display: table-cell;
}
.center {
  display: table-cell;
}
```

#### flex

```css
.container {
  display: flex;
}
.left {
  width: 100px;
}
.right {
  width: 200px;
}
.center {
  flex: 1;
}
```

#### grid

```css
.container{
  display: grid;
  width: 100%;
  grid-template-columns: 100px auto 200px;k
}
```
