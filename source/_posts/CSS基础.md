---
title: CSS基础
tags:
  - CSS
  - 学习
  - 前端
categories: 前端
keywords: 前端 CSS 样式
cover: 'https://w.wallhaven.cc/full/1p/wallhaven-1pd1o9.jpg'
abbrlink: 10604
date: 2024-02-07 19:11:31
---

# CSS 基础

## 前言

从头开始温习一遍 CSS，然后记录一下，避免以后忘了。

## 概念

### 选择器

选择器，顾名思义就是用来选择 HTML 元素的，不然我们在 CSS 中如何知道应该给哪一个元素添加样式呢？🤗

他分为五类：

- **元素选择器**: 选择一个或多个 dom 元素
- **关系选择器**: 选择元素之间的关系，也叫做运算符
- **属性选择器**: 根据元素的属性来选择对应的元素
- **伪类选择器**: 选择元素的特殊状态
- **伪元素选择器**: 选择元素的特殊部分

#### 元素选择器

```css
* {
  /* 选择所有的元素 */
}

div {
  /* 选择所有的 div 元素 */
}

div#id111 {
  /* 选择 id 为 id111 的 div 元素 */
}
span.class1 {
  /* 选择 class 为 class1 的 span 元素 */
}
```

#### 关系选择器 / 运算符

```css
div span {
  /* 包含 */
  /* 选择 div 元素下的所有 span 元素 */
}

.box > span {
  /* 只能选中子，不包括孙 */
  /* 选择 class 为 box 的元素下的所有子 span 元素 */
}

.box + span {
  /* 选择紧接在 class 为 box 的元素后的 span 元素 */
}

#id1 ~ span {
  /* 选择 id 为 id1 的元素后面的所有兄弟 span 元素 */
}
```

#### 属性选择器

```css
a[target] {
  /* 选择带有 target 属性的 a 元素 */
}

a[target='_blank'] {
  /* 选择 target 属性值为 _blank 的 a 元素 */
}

a[target^='http'] {
  /* 选择 target 属性值以 http 开头的 a 元素 */
}

a[target$='.pdf'] {
  /* 选择 target 属性值以 .pdf 结尾的 a 元素 */
}

a[target*='pdf'] {
  /* 选择 target 属性值包含 pdf 的 a 元素 */
}

a[target~='pdf'] {
  /* 选择 target 属性值包含 pdf 的 a 元素 */
  /* 比如: 'pdf word' */
}
```

#### 伪类选择器

伪类是选择一个处于特定状态的元素，可以用伪类选择器来选择他，比如 `:hover`、`:active`、`:focus` 等。

```css
a:link {
  /* 链接访问前的样式 */
}

a:visited {
  /* 链接访问后的样式 */
}

a:hover {
  /* 鼠标悬停时的样式 */
}

a:active {
  /* 链接激活时的样式, 指的是鼠标点击与释放之间 */
}

a:focus {
  /* 链接获取焦点时的样式 */
}

a:not(.ca) {
  /* 选择不包含 class 为 ca 的 a 元素 */
  /* (这里放选择器) */
}

span:first-child {
  /* 选择父元素下的第一个 span 元素 */
}

span:last-child {
  /* 选择父元素下的最后一个 span 元素 */
}

span:nth-child(2) {
  /* 选择父元素下的第二个 span 元素 */
  /* (odd) 选择奇数元素  */
  /* (even) 选择偶数元素 */
}

div:only-child {
  /* 选择父元素下只有一个子元素的 div 元素 */
}

div:empty {
  /* 选择没有子元素的 div 元素 */
}

input:checked {
  /* 选择被选中的 input 元素 */
}

input:enabled {
  /* 选择可用的 input 元素 */
}

input:disabled {
  /* 选择不可用的 input 元素 */
}
```

举例一些用途：ul 的最后一个元素不添加边框、表格的奇数行添加背景色。

```html
<ul>
  <li>1</li>
  <li>2</li>
  <li>3</li>
  <li>3</li>
</ul>

<style>
  li:not(:last-child) {
    border: red 1px solid;
    /* 除了最后一个li，其他的都添加红色边框 */
  }
</style>
```

```html
<table>
  <tr>
    <td>11</td>
  </tr>
  <tr>
    <td>11</td>
  </tr>
  <tr>
    <td>11</td>
  </tr>
</table>

<style>
  /* 表格奇数行添加背景色 */
  table tr:nth-child(odd) {
    background-color: #f2f2f2;
  }
</style>
```

#### 伪元素选择器

伪元素和伪类很像，但是伪元素类似于添加了一个新的 DOM 节点，而不是改变元素的状态。但也不是真的添加了一个新的 DOM 节点，只是在 CSS 中模拟了一个。

> CSS3 将伪元素选择器原本的 `:` 改成了 `::`，用来区分伪类，但是俩个写法都有效。

```css
p::first-line {
  /* 选择 p 元素的第一行 */
}

p::first-letter {
  /* 选择 p 元素的第一个字母 */
}

p::before {
  /* 在 p 元素之前插入内容 */
  content: 'Hello';
}

p::after {
  /* 在 p 元素之后插入内容 */
  content: 'World';
}

input::placeholder {
  /* 选择 input 元素的 placeholder */
  color: red;
}

p::selection {
  /* 选择 p 元素的选中部分 */
  color: red;
}
```

### 优先级

既然有选择器了，我们就可以给元素添加样式了，但是当一个元素被多个选择器选中时，该如何确定最终的样式呢？

这里就涉及到优先级的概念了

#### 优先级的顺序

1. !important
2. 内联样式
3. ID 选择器
4. 类选择器
5. 属性选择器 / 伪类选择器 / 标签选择器
6. 伪元素选择器
7. 通配符选择器 / 否定伪类 / 关系选择器

权重值：

| 选择器                             | 权重值     |
| ---------------------------------- | ---------- |
| !important, 内联样式               | [1,0,0,0]  |
| ID 选择器                          | [0,1,0,0]  |
| 类选择器                           | [0,0,1,0]  |
| 属性选择器、伪类选择器、标签选择器 | [0,0,0,1]  |
| 通配符、关系选择器、否定选择器     | [0,0,0,0]  |
| 继承样式                           | 没有权重值 |

#### 权重值比较规则

- [1,0,0,0] > [0,99,99,99], 从左往右逐个等级比较，前面的相等，才会往后比较
- 如果权重值相等，后面的样式会覆盖前面的样式(这里指的是在 CSS 文件中的顺序)

#### !important

当我们使用他的时候，会覆盖其他的任何声明。尽量避免使用他，因为他会破坏 CSS 的优先级规则，导致代码难以维护。

## 属性

### 定位

一个元素在页面中的位置，可以通过定位来控制。

#### position

用来指定元素在文档中的定位方式，可以设置的值如下：

- static: 默认值
- absolute: 绝对定位，相对于最近的非 static 定位的父元素进行定位
- fixed: 固定定位，相对于**浏览器窗口**进行定位
- relative: 相对定位，相对于元素自身的位置进行定位
- sticky: 粘性定位，相对定位和固定定位的混合，当元素在屏幕中可见时，表现为相对定位，当元素在屏幕中不可见时，表现为固定定位
- unset: 重置为父元素的值

##### 脱离文档流

设置了 `position` 属性为`absolute`、`fixed`、`sticky` 时，元素会脱离文档流，不再占据文档流中的位置，其他元素会以他脱离文档流的位置进行排列。

- 相当于他不占据文档流的位置，其他元素不会受到他的影响
- 这个元素会变成块级元素，相当于设置了 `display: block;`
- 他的宽度会自适应内容的宽度，不再占据整行，相当于`width: 100%`变成了`width: auto;`

##### sticky

生效条件：

1. 父元素的 `overflow` 属性不为 `visible`
2. 四个方向的 `top`、`right`、`bottom`、`left` 至少指定一个值
3. 同时设置时，top 优先级大于 bottom，left 优先级大于 right

```html
<div class="outer">
  文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容
  <div class="sticky"></div>
  文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容
</div>

<style>
  .outer {
    width: 300px;
    height: 150px;
    padding: 10px;
    border: red 1px solid;
    overflow: auto;
  }
  .sticky {
    position: sticky;
    top: 0;
    left: 0;
    background-color: yellow;
    height: 50px;
  }
</style>
```

#### float

总感觉不咋见到他 😤，运行代码看效果就行

```html
<div class="btns">
  <button value="none">none</button>
  <button value="left">left</button>
  <button value="right">right</button>
</div>
<div class="outer">
  文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容
  <div class="box"></div>
  文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容文本内容
</div>

<style>
  .btns {
    margin-bottom: 10px;
  }
  .outer {
    width: 300px;
    height: 150px;
    padding: 10px;
    border: red 1px solid;
    overflow: auto;
  }
  .box {
    float: right;
    background-color: yellow;
    height: 50px;
    width: 50px;
  }
</style>

<script>
  let btns = document.querySelectorAll('.btns button')
  let box = document.querySelector('.box')
  btns.forEach(function (btn) {
    btn.onclick = function () {
      box.style.float = this.value
    }
  })
</script>
```

#### z-index

表示元素在 z 轴上的位置，值越大，元素越靠前。

使用相对性： 他只决定在同一个父元素中的同级子元素的层级关系，不会影响到其他父元素中的元素。

注意事项 ⚠️：

- 只有设置了 `position` 属性的元素才能使用 `z-index` 属性
- 父节点的 `z-index` 值会影响子节点的 `z-index` 值
- 如果没有值，或者值相同，那么就看在 html 中的顺序，后面的就在上面

失效场景：

- 父元素`position: relative`，子元素`position: absolute`，子元素的 `z-index` 大于父元素的 `z-index`，子元素的 `z-index` 会失效
- 元素的 `position` 属性为 `static`

### 盒子模型

#### display

- none: 隐藏元素
- block: 块级元素，前后带有换行符
- inline:（默认值），行内元素，不会独占一行
- inline-block: 行内块级元素，不会独占一行
- flex: 弹性盒子
- grid: 网格布局
- table: 表格
- list-item: 列表项

##### 行内元素与块级元素

- inline: 元素**不独占一行**，宽度和高度由内容决定，设置宽高无效，不支持 margin、padding 的上下方向，但是支持左右方向
  - 常见的 inline 元素包括 <span>、<a>、<strong> 等。
- block: 元素**独占一行**，width, height, margin, padding 都可以设置
  - 常见的 block 元素包括 <div>、<p>、<h1> 等。
- inline-block: 元素**不独占一行**，width, height, margin, padding 都可以设置
  - 常见的 inline-block 元素包括 <img>、<button>、<input> 等。

##### 行内元素的空隙问题

行内元素之间有空隙，这是因为行内元素的排版规则，他会把换行符、空格、tab 等当做一个空格处理。

可以给父元素设置 `font-size: 0;`，然后给子元素设置 `font-size`，这样就可以解决行内元素之间的空隙问题。

```html
<div class="box">
  <div>111</div>
  <div>111</div>
  <div>111</div>
</div>
<style>
  .box {
    width: 100px;
    height: 100px;
    border: red 1px solid;
    margin-left: 100px;
    /* font-size: 0; */
  }
  .box div {
    display: inline;
    background-color: blue;
    /* font-size: 16px; */
  }
</style>
```

取消注释即可，不取消的话俩个 div 之间有空隙

#### width / height

可选值：auto、百分比、固定值

##### 宽高自适应

`width`的默认值就是 auto，他有这些含义：

- 充分利用可用空间：比如 div, p 的的宽度默认是 100%于容器的宽度
- 适应内容：比如 span, a 的宽度默认是适应内容的宽度，收缩到合适

#### margin / padding

外边距和内边距

```css
/* 应用于所有边 */
margin: 1em;
margin: -3px;

/* 上边下边 | 左边右边 */
margin: 5% auto;

/* 上边 | 左边右边 | 下边 */
margin: 1em auto 2em;

/* 上边 | 右边 | 下边 | 左边 */
margin: 2px 1em 0 auto;

/* 全局值 */
margin: inherit;
margin: initial;
margin: unset;
```

#### box-sizing

用于定义文档如何计算一个元素的总宽度和高度。

- content-box: 标准盒子模型，默认值，宽度和高度只包括内容，不包括边框和内边距。实际宽度 = width + padding \* 2 + border \* 2
- border-box: 怪异盒子模型，宽度和高度包括内容、内边距和边框，即使设置了 padding 和 border，元素的宽度和高度也不会改变。实际宽度 = width

他们都不包括外边距 margin

#### box-shadow

定义元素的阴影，具体看[文档](https://developer.mozilla.org/zh-CN/docs/Web/CSS/box-shadow)吧

### 边框

[border](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border) 就是 border-width、border-style、border-color 三个属性的简写，但是使用 border 定义的时候就只能定义四个边的宽度一样。单独使用 border-width 可以分别定义四条边宽度。

#### [border-radius](https://developer.mozilla.org/zh-CN/docs/Web/CSS/border-radius)

设置元素的圆角，可以设置四个值，分别对应左上、右上、右下、左下的圆角。

### 字体

font 属性可以用来作为 font-style, font-variant, font-weight, font-size, line-height 和 font-family 的简写，害必须按这个顺序，而且不能省略 font-size 和 font-family🤬

- font-family: 字体系列，可以设置多个，用逗号隔开，如果第一个字体不存在，就会使用第二个，以此类推。
- font-size: 字体大小
- font-style: 字体样式，可以设置为 italic 斜体，normal 正常，oblique 倾斜
- font-weight: 字体粗细，可以设置为 bold 粗体 700，normal 正常 400，lighter 更细，bolder 更粗，100-900 数字

### 文本

#### text-align

用于定义字体的水平对齐方式，可选值：

- left/right/center: 左/右/居中 对齐
- justify: 俩端对齐，但是最后一行不会对齐
- justify-all: 俩端对齐，包括最后一行
- start/end: 由文本方向决定，比如阿拉伯语，start 就是右对齐，end 就是左对齐

#### vertical-align

定义行内元素在行框内的垂直对齐方式

- baseline: 默认值，元素的基线与父元素的基线对齐
- sub/supr: 把当前盒的基线放在父元素的下标/上标位置
- top/bottom: 元素的顶端与行框的 top/bottom 对齐
- text-top/text-bottom: 元素的顶端/底端与父元素的顶端/底端对齐
- middle: 元素的中点与行框的中点对齐

#### text-decoration

他也是简写属性，用来定义元素的文本装饰

text-decoration: <text-decoration-line> | <text-decoration-style> | <text-decoration-color>

text-decoration-line:

- none: 默认值，无装饰
- underline: 下划线
- overline: 上划线
- line-through: 删除线
- blink: 闪烁

text-decoration-style:

- solid: 实线
- double: 双实线
- dotted: 点线
- dashed: 虚线
- wavy: 波浪线

#### text-transform

定义文本如何转换大小写

- none: 默认值，不转换
- capitalize: 首字母大写
- uppercase: 全部大写
- lowercase: 全部小写
- full-width: 全角

#### text-justify

俩端对齐的方式

#### text-indent

块内文本内容的缩进

#### text-overflow

> 可以用来实现文本溢出时的省略号
> 要让他生效，需要定义 `white-space: nowrap;` 和 `overflow: hidden;`

- clip: 默认值，不显示省略号，直接裁剪
- ellipsis: 显示省略号

#### word-wrap

当内容超过指定容器的边界时是否断行

- normal: 默认值，不断行
- break-word: 强制断行

#### word-break

定义文本的字间和字符间的换行

- normal: 默认值，依据各自语言的规则，允许在字间发生换行
- keep-all: 对于 CJK（中文，韩文，日文）文本不允许在字符内发生换行，Non-CJK 文本表现同 normal
- break-all: 对于 Non-CJK 文本允许在任意字符内发生换行。该值适合包含一些非亚洲文本的亚洲文本，比如使连续的英文字符断行
- break-word: 与 break-all 类似，但是不允许在连续的英文字符内换行

#### white-space

指定元素是否保留文本之间的空格、换行

- normal: 默认值，合并空白符，换行符
- pre: 原封不动的保留你输入时的状态，空格、换行都会保留，并且当文字超出边界时不换行。等同 <pre> 元素效果
- nowrap: 合并空白符，换行符，不换行

#### line-height

可以使用它来做垂直居中，默认是按照 base-line 对齐，可以用`vertical-align: middle`

### 背景

https://tsejx.github.io/css-guidebook/concept/properties/background

### 变换

#### transform

可以旋转、缩放、倾斜、平移元素

部分取值：

- translateX(x): 沿着 x 轴平移
- translate(x, y): 沿着 x、y 轴平移
- scale(x, y): 缩放
- rotateX(angle): 沿着 x 轴旋转
- rotate(angle): 旋转，需要先定义变形的原点(transform-origin)，然后再旋转

做个例题：用 css3 实现秒针的转动

```html
<div class="clock">
  <dv class="second"></dv>
</div>

<style>
  .clock {
    width: 200px;
    height: 200px;
    border: blue 1px solid;
    border-radius: 50%;
    position: relative;
  }
  .second {
    height: 96px;
    width: 2px;
    background-color: black;
    position: absolute;
    top: 4px;
    left: 50%;
    transform-origin: 50% 100%;
    animation: rotateSeconds 60s linear infinite;
  }
  @keyframes rotateSeconds {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(360deg);
    }
  }
</style>
```

### 动画

#### animation

就是所有属性的简写，除了 animation-play-state

animation: name duration function delay count direction;

animation: 动画名称 动画时间 运动曲线 何时开始 播放次数 是否反方向;

#### animation-name

就是关键帧的名称，配合@keyframes 一起使用，需要设置运动时间，不然不会播放

```html
<div class="box"></div>
<style>
  .box {
    height: 100px;
    width: 100px;
    background-color: blue;
    animation-name: move;
    animation-duration: 4s;
    position: absolute;
  }
  @keyframes move {
    from {
      left: 0px;
    }
    to {
      left: 200px;
    }
  }
</style>
```

#### animation-duration

动画持续时间，6s、0.5s、3ms

#### animation-timing-function

速度曲线，默认 ease

- linear: 匀速
- ease: 慢-快-慢
- ease-in: 慢-快
- ease-out: 快-慢
- ease-in-out: 很慢-慢-快-慢-很慢
- cubic-bezier(n,n,n,n): 自定义速度曲线

#### animation-delay

延迟时间，默认 0。可以为负数，-2s 让动画立刻开始，但是跳过 2s。

#### animation-iteration-count

动画播放次数，默认 1，可以设置为 infinite 无限循环

#### animation-direction

是否反方向播放，默认 normal

#### animation-play-state

可以用来控制播放动画的状态，暂停 paused，继续 running

### 过渡

可以把元素的样式从一种变换为另一种

## 规则

### @font-face 自定义字体

### @import 样式导入

一定要先于除了`@charset`之外的所有规则

```css
@import './style.css';
@import url('style.css');
```

### @keyframes 关键帧

指定动画名称和效果

```css
@keyframes move {
  from {
    left: 0px;
  }
  to {
    left: 200px;
  }
}

@keyframes move2 {
  0% {
    left: 0px;
  }
  50% {
    left: 100px;
  }
  100% {
    left: 200px;
  }
}
```

### @media 媒体查询