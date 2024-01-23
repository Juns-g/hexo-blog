---
title: Vue3面试知识点
tags:
  - Vue
  - 面试
  - 前端
categories: 面试
cover: >-
  https://dogefs.s3.ladydaily.com/~/source/wallhaven/full/2y/wallhaven-2y1op9.jpg?w=2560&h=1440&fmt=webp
abbrlink: 7649
date: 2024-01-23 21:52:54
---

# Vue3 面试知识点

## 1. Vue3 的响应式

### 关键词

`Proxy`, `Refelct`

### 特点

- 众所周知`Vue2`的响应式是通过`Object.defineProperty()`来劫持各个属性的 get 和 set，当数据变化时发布消息给`订阅者`，触发监听回调，但是其实这个API存在很多问题
- `Vue3`中为了解决这些问题，使用了`Proxy`和`Reflect`来代替他
  - 支持监听对象和数组的变化
  - 对象嵌套属性时只代理第一层，运行时才递归代理深层，性能更好
  - 能拦截对象13种方法，新增数据结构都支持
- 提供了俩个API：`ref`, `reactive`

### 什么是Proxy

`Proxy`用与创建一个目标对象的代理，在对目标对象的操作之前提供了拦截，可以对外界的操作进行过滤和改写。这样我们可以不直接操作对象本身，而是通过操作代理对象来间接操作对象

### defineProperty 和 Proxy 的区别

- `defineProperty`是ES5的方法，`Proxy`是ES6的
- `defineProperty`是劫持对象的某个属性，`Proxy`是代理整个对象
- `defineProperty`监听对象/数组时，需要迭代对象的每个属性
- `defineProperty`不能监听道对象的新增属性，`Proxy`可以
- `defineProperty`不支持`Map`, `Set`等数据结构
- `defineProperty`只支持get和st，而`Proxy`支持13种方法
- `Proxy`兼容性差，且无法通过`pollyfill`解决，所以`Vue3`不支持IE

### 为什么需要Reflect

> [不会Reflect，怎么玩转Proxy --Reflect篇](https://juejin.cn/post/7286787235361243148)

大多数情况我们很少使用到 Reflect，他使用最多的就是配合 Proxy 使用。他的方法和Proxy的方法数量相同，并且方法名一模一样，而且他返回的值也正是Proxy需要的值。

其实就是相当于不用我们自己手写时间get/set等，直接return Reflect.get(...arguments)就可以了

- 使用`Reflect`可以修正`Proxy`的this问题(这点有待研究)
- `Proxy`的一些方法要求返回boolean来表示操作是否成功，这也和`Reflect`对应
- 以前的许多借口都定义在`Object`上，历史原因越来越杂，所以直接干脆都挪到`Reflect`上，目前是13种，之后可能新增

### 对数组的处理

`Vue2`对数组的监听做了特殊处理，`Vue3`也需要做处理，原因是：

- `push`, `pop`等方法操作数组时会隐式访问和修改数组的`length`，所以我们需要这时停止依赖追踪
- `includes`, `indexOf`, `lastIndexOf`这些查找时，可能是使用代理对象进行查找，也有可能使用原始值去查找，使用需要重写，先去响应式对象中找，没找到再去原始值内找

源码地址: [packages/reactivity/src/baseHandlers.ts#L48](https://github.com/vuejs/core/blob/f1068fc60ca511f68ff0aaedcc18b39124791d29/packages/reactivity/src/baseHandlers.ts#L48)

### 惰性响应式

- `Vue2`对一个深层前端套的对象做响应式，需要递归遍历他，每一层都变成了响应式
- 但是在`Vue3`中为了减少性能消耗，只有在getter中用到的内部属性才去递归响应式

源码地址: [packages/reactivity/src/baseHandlers.ts#L155](https://github.com/vuejs/core/blob/f1068fc60ca511f68ff0aaedcc18b39124791d29/packages/reactivity/src/baseHandlers.ts#L155)

也就是因为这个，所以对象其实只代理了第一层，只有当get时拿到的返回值是Object的时候，才会用reactive进行深度代理。

但是也带来了解构丢失响应式的问题

- 对`Vue3`响应式数据使用`ES6解构`出来的是一个`引用对象类型`时，它还是响应式的(因为返回的是响应式数据)。但是解构出的是`基本数据类型`时，响应式会丢失。

### Set、Map做的处理

因为Proxy无法直接拦截题目，他们的方法必须在自身调用，所以Vue3封装了toRaw()返回原对象

## 参考文章

- [「2022」寒冬下我的面试知识点复盘【Vue3、Vue2、Vite】篇](https://juejin.cn/post/7166446028266733581#heading-34)https://juejin.cn/post/7166446028266733581#heading-34
