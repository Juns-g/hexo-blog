---
title: CSS文本溢出隐藏
tags:
  - 面试
  - 前端
  - CSS
cover: >-
  https://files.codelife.cc/wallhaven/full/vq/wallhaven-vq5odm.jpg?x-oss-process=image/resize,limit_0,m_fill,w_2560,h_1440/quality,Q_92/format,webp
categories: 面试
abbrlink: 12866
date: 2024-07-28 19:24:19
---

# CSS 文本溢出隐藏

面试常问，开发也常用，组件库应该都有文本组件支持这样的功能，这里一次性了解下所有的实现方式

## 单行

其实只要知道要做什么，答案就有了。

- 让文字只显示在一行：white-space: nowrap
- 长度超出时，隐藏溢出内容：overflow: hidden
- 溢出的内容用省略号展示：text-overflow: ellipsis

```css
white-space: nowrap;
overflow: hidden;
text-overflow: ellipsis;
```

## 多行

可以分俩种情况，一种是限制 xx 行后隐藏，另一种是超出 xx 高度后隐藏

### 高度截断

使用伪元素定位到行尾，遮住文字，但是看上去其实还挺奇怪的

```css
.box2 {
  overflow: hidden;
  position: relative;
  height: 45px;
  overflow: hidden;
}
.box2::after {
  content: '...';
  position: absolute;
  right: 0;
  bottom: 0;
}
```

### 行数截断

使用 webkit 的 css 拓展功能即可

```css
overflow: hidden;
text-overflow: ellipsis;
-webkit-line-clamp: 2;
display: -webkit-box;
-webkit-box-orient: vertical;
```
