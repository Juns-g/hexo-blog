---
title: CSS与页面性能
tags:
  - 面试
  - 前端
  - CSS
cover: >-
  https://files.codelife.cc/wallhaven/full/vq/wallhaven-vq5odm.jpg?x-oss-process=image/resize,limit_0,m_fill,w_2560,h_1440/quality,Q_92/format,webp
categories: 面试
abbrlink: 27026
date: 2024-07-21 17:45:12
---

# CSS 与页面性能

> 性能优化绝对是提及频次很高的问题，这里讨论一下如何从 CSS 上做性能优化

## 渲染方面

- 重排和重绘：都是比较消耗性能的，比如在 vue 中频繁切换的场景使用 v-show 替代 v-if，或者批量修改样式
- 减少性能开销大的属性：动画、浮动、特殊定位
- 减少 CSS 计算属性：比如 calc(100% - 20px)，可以用边距来控制
- 属性值为 0 时不加单位
- 减少 CSS 规则的数量
- 减少复杂选择器的使用，深度嵌套的更耗费性能

> 选择器性能排行
>
> ```css
> 1.id选择器（#myid）
> 2.类选择器（.myclassname）
> 3.标签选择器（div,h1,p）
> 4.相邻选择器（h1+p）
> 5.子选择器（ul > li）
> 6.后代选择器（li a）
> 7.通配符选择器（*）
> 8.属性选择器（a[rel="external"]）
> 9.伪类选择器（a:hover,li:nth-child）
> ```

## 加载方面

- css 放到 head 中，减少首次渲染时间
- 按需加载
- 样式和结构分离
- css 压缩，降低体积
- 减少 @import 的使用
