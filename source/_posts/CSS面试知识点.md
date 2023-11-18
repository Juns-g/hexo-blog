---
title: CSS面试知识点
tags:
  - CSS
  - 面试
  - 前端
categories: 面试
keywords: 面试 CSS 样式
cover: 'https://w.wallhaven.cc/full/1p/wallhaven-1pd1o9.jpg'
abbrlink: 39304
date: 2023-05-09 16:49:25
---

# CSS

## CSS3 特性

## 选择器

[03-CSS 样式表和选择器](https://web.qianguyihao.com/02-CSS%E5%9F%BA%E7%A1%80/03-CSS%E6%A0%B7%E5%BC%8F%E8%A1%A8%E5%92%8C%E9%80%89%E6%8B%A9%E5%99%A8.html#css-%E7%9A%84%E5%9B%9B%E7%A7%8D%E5%9F%BA%E6%9C%AC%E9%80%89%E6%8B%A9%E5%99%A8)

### 常用

- id 选择器（#box），选择 id 为 box 的元素
- 类选择器（.one），选择类名为 one 的所有元素
- 标签选择器（div），选择标签为 div 的所有元素
- **后代选择器（#box div），选择 id 为 box 元素内部所有的 div 元素**
- **子选择器（.one>one_1），选择父元素为.one 的所有.one_1 的元素**
- **相邻同胞选择器（.one+.two），选择紧接在.one 之后的所有.two 元素**
- **群组选择器（div,p），选择 div、p 的所有元素**

## 继承性和层叠性

- 关于文字样式的属性都可以继承

- 关于盒子，定位，布局的不可以继承

## 盒模型

有俩种，标准盒子模型(content-box)，IE 盒子模型(border-box)

切换的话直接有一个属性`box-sizing: content-box`

区别也不大

- content-box 就是宽和高只包括内容 content，不包括 border 和 padding
- border-box 就是宽高包括了 border 和 padding

## 文本溢出处理

### **单行文本溢出省略**

- `overflow:hidden;`（文字长度超出限定宽度，则隐藏超出的内容）
- `white-space:nowrap;`（设置文字在一行显示，不能换行）
- `text-overflow:ellipsis;`（规定当文本溢出时，显示省略符号来代表被修剪的文本）

优点：

- 没有兼容问题
- 响应式截断
- 文本溢出范围才显示...，否则不显示
- 省略号位置刚好

缺点：只适用于单行文本溢出

### 多行

**纯 CSS**

- `-webkit-line-clamp: 2;`（用来限制在一个块元素显示的文本的行数, 2 表示最多显示 2 行。 ）
- `display: -webkit-box;`（将对象作为弹性伸缩盒子模型显示 ）
- `-webkit-box-orient: vertical;`（设置或检索伸缩盒对象的子元素的排列方式 ）
- `overflow: hidden;`（文本溢出限定的宽度就隐藏内容）
- `text-overflow: ellipsis;`（多行文本的情况下，用省略号“…”隐藏溢出范围的文本)

**优点**

- 响应式截断
- 文本溢出范围才显示省略号，否则不显示省略号
- 省略号显示位置刚好

**短板**

- 兼容性一般： -webkit-line-clamp 属性只有 WebKit 内核的浏览器才支持，适用移动端

## 两栏布局

思路：float，flex，grid，个人感觉 flex 最合适，float 在现在不是很常用了

float：

```css
.container {
  overflow: hidden; /* 清除浮动 */
}

.left {
  float: left;
  width: 200px;
}

.right {
  margin-left: 200px; /* 左侧固定宽度 */
}
```

flex:

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

grid:

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr;
}
```

但是 flex 有一个问题是默认属性值`align-items: stretch;`，就导致列等高，让他们高度自动的话就需要设置`align-items: start;`

## 三栏布局

[css 三栏布局 - 掘金 (juejin.cn)](https://juejin.cn/post/7084911270603784205)

## BFC

块级格式化上下文

BFC是一个独立的布局环境，可以理解为一个容器，在这个容器中按照一定规则进行物品摆放，并且
不会影响其它环境中的物品。如果一个元素符合触发BFC的条件，则BFC中的元素布局不受外部影响。

创建条件：

- 根元素：body
- 元素设置浮动：float 除 none 以外的值；
- 元素设置绝对定位：position (absolute、fixed)；
- display 值为：inline-block、table-cell、table-caption、flex等；
- overflow 值为：hidden、auto、scroll；

特点：

- BFC 垂直方向边距重叠
- BFC 的区域不会与浮动元素的 box 重叠
- BFC 是一个独立的容器，外面的元素不会影响里面的元素
- 计算 BFC 高度的时候浮动元素也会参与计算

作用：

- 解决 margin 重叠：两个元素都变成 BFC，他们在同一个 BFC 内 margin 会塌陷
- 解决高度塌陷：子元素设置浮动，父元素会高度塌陷，将父元素清除浮动`overflow:hidden`设置为 BFC
- 自适应两栏布局：左边宽度固定，右边自适应

## 浮动

## display

- `block`：元素呈块级显示，前后都带有换行符。可以设置宽度、高度和边距等属性。
- `inline`：元素呈行内显示，只占据内容的空间。不能设置宽度、高度和边距等属性。
- `inline-block`：元素呈行内块显示，占据内容的空间，可以设置宽度、高度和边距等属性。
- `flex`：弹性布局
- `grid`：网格布局
- `none`：元素不显示
- `table`：表格显示

此外，`display` 还有一些特殊用法：

- `display: none` 可以隐藏元素，但不会占据空间。
- `display: block` 可以让元素成为块级元素，可以通过设置 `margin` 来实现居中布局。
- `display: inline-block` 可以让元素成为行内块元素，可以通过设置 `vertical-align` 来实现对齐布局。

## position

- `static`：不咋用，一般默认就是吧
- `relative`：占用空间和正常相同，相对于原本位置偏移
- `absolute`：从标准流中移除，不占据空间
  - 指定元素相对于最近的非 static 定位祖先元素的偏移，来确定元素位置
- `fixed`：从标准流中移除，不占据空间
  - 指定元素相对于屏幕视口（viewport）的位置来指定元素位置
  - 元素的位置在屏幕滚动时不会改变

## 水平垂直居中

水平：`text-align:center;`

垂直：行高=高度

水平垂直：

- `margin:auto;`

- 绝对定位+translate，`top:50%` `left:50%` `transform:translate(-50%,-50%)`

- 绝对定位+上下左右 0+`margin:auto`

- flex 布局`display:flex; align-items:center; justify-content:center;`

- flex+margin，父容器`display:flex`，子元素`margin:auto;`

## flex 布局

默认有俩条轴，主轴和交叉轴，垂直，默认沿主轴排列，`flex-direction`决定主轴方向，默认值是横向

**常用属性**：

- `flex-direction`，row,column,row-reverse,column-reverse，简单
- `flex-wrap`
  - nowrap，所有元素在一行，容易溢出
  - wrap，多行，wrap-reverse，反过来的
- `flex-flow`，就是上面俩个合起来的简写
- `justify-content`沿着主轴的对齐方式
  - center，在主轴上居中
  - start，end，紧靠首尾，前面加上 flex-是大致相同
  - space-between，首尾紧贴父元素，中间有间隙
  - space-around，首尾与父元素的距离是子元素之间间隙的一半
  - space-evenly，首尾与父元素距离和子元素之间的间隙一样
- `align-items`，同上，但是设置的是交叉轴的
- `align-content`，定义多根轴线

**`flex: 1;`**

- 展开是`flex: 1 1 0%;`
- flext-grow:0 ,flex-shirk:1,flex-basic:0%
- flex-grow 属性用于设置或检索弹性盒子的扩展比率,默认值0；
- flex-shrink 属性指定了 flex 元素的收缩规则，默认值1；
- flex-basis 属性用于设置或检索弹性盒伸缩基准值，默认值auto;

## grid 布局

- `grid-template-rows: 20px 20px 20px`，就是三行
- `grid-template-columns: repeat(2,50px)`，就是两列
- `gird-template-rows`的一些值
  - `repeat(auto-fill, 200px)`，列宽 200px，数量不固定，浏览器放得下就放
  - `200px 1fr 2fr`，表示比例的，第一个是 200px，后面的分别占剩下的 1 : 2
  - `minmax(100px, 1fr)`，列宽最小和最大值
  - `100px auto 100px`，中间自由，第一第三个是 100px
- `grid-gap: 5px`就是间隙 5px，是`grid-row-gap:5px`和`grid-column-gap:5px`的缩写
- `justify-items`和`align-items`和 flex 相同

## sass，less

## px rem em vh vw

**rem，em**

默认情况下都是 16px，rem 就是 root:em

rem 是相对 html 标签里面的`font-size`的，例如：

```css
html {
  font-size: 32px; // 1rem = 32px
}
```

而 em 则可以再 em 在的元素上或者父元素上设置`font-size`来确定依赖的值

```html
<div class="father">
  父元素
  <div class="child">子元素</div>
</div>

<style>
  .father {
    font-size: 32px;
  }
  .child {
    font-size: 1em; // 32px
  }
</style>
```

**vw,vh 和百分比**

百分比基于的是父元素，而 vw,vh 基于的是窗口

**vmin**是 vw 和 vh 中小的那个，**vmax**同理

## CSS变量

定义用--开头，`--main-bg-color: red`

使用时需要用var，`background-color: var(--main-bg-color)`