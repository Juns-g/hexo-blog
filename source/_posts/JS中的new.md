---
title: JS中的new
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 56167
date: 2024-02-17 14:36:29
---

# JS 中的 new

前面我们已经学过了 this，原型链了，现在可以来理解 new 做了啥了 🤪。

## 有啥用

先来看例子

```js
function Test(name) {
  this.name = name
}
Test.prototype.sayName = function () {
  console.log(this.name)
}
const t = new Test('111')
console.log(t.name) // '111'
t.sayName() // '111'
```

这里我们可以看到，通过 new 生成的实例，可以访问到构造函数中的属性和原型链上的方法。也就是说，new 做了几件事，创建了一个新的对象，将这个对象的原型链指向构造函数的原型对象，并且把构造函数的 this 指向这个新的对象。

## 返回值问题

一般来说，我们用做构造函数时，不会有 return，默认也是 return undefined，但是如果我们手动 return 值会发生什么？

已经有人测试过了 🤗，如果返回的是一个对象，那么这个对象就会被返回，如果返回的是基本类型，那么这个基本类型就会被忽略。

```js
function Test(name) {
  this.name = name
  return { age: 1 }
}

let t = new Test('我是名字')
console.log(t) // { age: 1 }
```

## 手写

所以总结看呢，我们需要做这些事情：

1. 创建一个新对象
2. 原型链接
3. 绑定 this
4. 判断返回值

```js
function myNew(ctor, ...args) {
  if (typeof ctor !== 'function') {
    throw new Error('构造函数必须是function')
  }
  const obj = {}
  // 原型链链接有很多方案
  obj.__proto__ = ctor.prototype
  const res = ctor.apply(obj, args)
  // 如果构造函数返回对象，就return出去
  return res instanceof Object ? res : obj
}
```
