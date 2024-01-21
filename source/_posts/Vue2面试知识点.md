---
title: Vue2面试知识点
tags:
  - Vue
  - 面试
  - 前端
categories: 面试
cover: >-
  https://dogefs.s3.ladydaily.com/~/source/wallhaven/full/2y/wallhaven-2y1op9.jpg?w=2560&h=1440&fmt=webp
abbrlink: 36128
date: 2024-01-21 16:22:16
---

# Vue2 面试知识点

## 1. Vue2 的响应式

### 关键词

`Object.defineProperty()`, `getter/setter`, `发布-订阅模式`, `观察者模式`, `依赖收集`, `Watcher`, `Observer`, `Dep`, `原型链`, `__proto__`, `prototype`, `Array.prototype`, `parsePath`, `diff算法`, `虚拟DOM`, `设计模式`, `正则表达式`

这些都是有所涉及的，并且可以拓展下去问的

### 原理

简单概括就是：

`Vue`是采用数据劫持结合`观察者`(发布-订阅)模式的方式，通过`Object.defineProperty()`来劫持各个属性的`getter`和`setter`，在数据变动时发布消息给订阅者(`Watcher`)，触发相应的监听回调来更新`DOM`

### 响应式的创建、更新流程

- 当一个`Vue`实例创建时，`Vue`会遍历`data`选项的属性，用`Object.defineProperty()`为他们设置`getter/setter`，并且在内部追踪相关依赖，用户访问或修改属性时，会触发`getter/setter`的监听回调来更新`DOM`
- 每个组件实例都有相应的`watcher`实例，它会在组件渲染的过程中把属性记录为依赖，之后当依赖项的`setter`被调用时，会通知`watcher`重新计算，从而致使它关联的组件得以更新，生成新的虚拟`DOM`树
- `Vue`会遍历并比对新旧虚拟 DOM 树中每个节点的差别，并记录下来，最后将差异化的部分渲染到页面上（`diff`算法在 vue2 和 vue3 中有不同）

### Vue2 响应式的缺点

- `Object.defineProperty()`可以监听通过数组下标修改数组的操作，通过遍历每个数组元素来实现
  - 但是`Vue2`无法监听，原因是性能和体验不成正比，其次即使监听了也监听不到原生方法的操作
  - 处于性能考虑，`Vue2`放弃了对数组元素的监听，改为对数组原型上的 7 种方法进行劫持
- `Object.defineProperty()`无法检测直接通过`arr.length = 1`改变数组长度的操作
- `Object.defineProperty()`只能监听属性，所以需要深度遍历对象的每个属性，如果属性值也是对象，需要深度遍历。
- `Object.defineProperty()`只能监听属性而不是对象本身，所以对象新增的属性没有响应式；因此新增响应式属性需要使用`Vue.set()`方法
- 不支持 Map、Set、WeakMap 和 WeakSet

### 如何解决数组响应式的问题

`push`、`pop`、`shift`、`unshift`、`splice`、`sort`、`reverse`这七个数组方法，在`Vue2`内部重写了所以可以监听到，除此之外可以使用`set()方法，`Vue.set()`对于数组的处理其实就是调用了`splice`方法

## 参考文章

- [「2022」寒冬下我的面试知识点复盘【Vue3、Vue2、Vite】篇](https://juejin.cn/post/7166446028266733581#heading-34)
