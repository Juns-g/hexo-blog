---
title: JS中的作用域链与闭包
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 9545
date: 2024-02-15 17:16:29
---

# JS 中的作用域链与闭包

## 前言

年前字节一面就被这个坑了，还有闭包也说不出来，当时只能从动态作用域、this 的方面来解释，👉🏻😔

## 作用域链

首先我们先来了解一下作用域链，先看这个代码

```js
var a = '111'
function A() {
  console.log(a)
  a = '222'
  function B() {
    console.log(a)
  }
  B()
}
A()
```

我想他的输出应该是： 111 222，原因也很简单，因为作用域链，在打印 a 的时候，函数 A 里面没有这个变量，就沿着作用域链网上找到了全局的 a。

那再看这个：

```js
function bar() {
  console.log(myName)
}
function foo() {
  var myName = '111'
  bar()
}
var myName = '222'
foo()
```

按照我当时掉入坑的想法，我会认为他输出的是 111，但是实际答案是 222，因为作用域链是在函数定义的时候就已经确定了，而不是在函数调用的时候，所以在 bar 函数定义的时候，他的作用域链就已经确定了，所以他会沿着作用域链网上找到全局的 myName。

其实在每个执行上下文的变量环境中，都有一个外部引用，指向外部的执行上下文，可以称之为 outer。

在这个例子里面，bar 函数和 foo 函数的 outer 都指向全局执行上下文。至于原因，这就涉及到了词法作用域。

## 词法作用域

他指的是作用域由代码中函数声明的位置来决定，也就是说，函数的作用域是在函数定义的时候就已经确定了，而不是在函数调用的时候，所以也被称为静态作用域。js 中的 this 机制是动态作用域，和他是不一样的，由调用时的上下文来决定。

下面是偷的图：
![图](https://pic.imgdb.cn/item/65cddce89f345e8d03411fc9.jpg)

可以看出来，词法作用域就是根据函数定义的位置来决定的。
所以说整个作用域链的顺序是：foo -> bar -> main -> 全局作用域

那么这也就解释了，当时的 bar 函数的 outer 为什么指向全局，因为他定义在全局中。也就是说，**词法作用域是代码编译阶段就决定好的，和函数是怎么调用的没有关系**。

## 闭包

先看代码吧

```js
function foo() {
  var myName = '111'
  var innerBar = {
    getName: function () {
      console.log(1)
      return myName
    },
    setName: function (newName) {
      myName = newName
    },
  }
  return innerBar
}
var bar = foo()
bar.setName('222')
bar.getName()
console.log(bar.getName())
```

直接看 🤔 感觉没啥问题，但是结合调用栈的知识想想，在 foo 执行完，把返回值 return 给 bar 的时候，foo 会从调用栈中弹出，应该是回收变量，但是之后的方法为什么还是生效呢？

这就涉及到了闭包了，虽然 foo 的上下文弹出了，但是由于返回的东西使用了他的内部变量 myName，所以这个变量还是依然保存在内存中的，就像是一个背包，把 foo 的东西都带走了。

但是他也是专属的背包，除了 bar 之外，其他人是拿不到的，所以这个东西也是私有的。

### 是什么

那么现在我们可以来**定义**一下闭包：

在 JS 中，根据词法作用域的规则，内部函数可以访问外部函数声明的变量，当通过调用外部函数返回一个内部函数后，即使外部函数已经执行完毕，但是内部函数引用的外部函数的变量依然保存在内存中，可以把这些变量的集合称之为闭包。

也就是说，闭包可以简单定义为：**可以访问其他函数内部变量的函数**

### 形成原因

内部函数存在外部作用域的引用

### 变量存储位置

闭包中的变量存储的位置是堆内存

假如闭包中的变量存储在栈内存中，那么栈的回收会把处于栈顶的变量自动回收。所以闭包中的变量如果处于栈中那么变量被销毁后，闭包中的变量就没有了。

### 作用

保护函数的私有变量不受外界干扰，实现方法和属性的私有化。

### 使用场景

- 函数作为参数

```js
var a = '111'
function foo() {
  var a = 'foo'
  function fo() {
    console.log(a)
  }
  return fo
}

function f(p) {
  var a = 'f'
  p()
}
f(foo()) // foo
```

- IIFE 自执行函数

```js
var n = '111'
;(function p() {
  console.log(n)
})()
// 111
```

- 循环赋值

```js
for (var i = 0; i < 10; i++) {
  ;(function (j) {
    setTimeout(function () {
      console.log(j)
    }, 1000)
  })(i)
}
// 不这样写的话直接都是 10
// 为什么会连续输出10，因为 JS 是单线程的遇到异步的代码不会先执行(会入栈)，等到同步的代码执行完 i++ 到 10时，异步代码才开始执行此时的 i=10 输出的都是 10。
```

- 使用回调函数其实就是闭包

```js
window.name = '11'
setTimeout(function timeHandler() {
  console.log(window.name)
}, 100)
```

- 节流防抖

```js
// 引用了外部函数的变量timer
function throttle(fn, timeout) {
  let timer = null
  return function (...arg) {
    if (timer) return
    timer = setTimeout(() => {
      fn.apply(this, arg)
      timer = null
    }, timeout)
  }
}

function debounce(fn, timeout) {
  let timer = null
  return function (...arg) {
    clearTimer(timer)
    timer = setTimeout(() => {
      fn.apply(this, arg)
    }, timeout)
  }
}
```

- 柯里化

```js
function curry(fn, len = fn.length) {
  return _curry(fn, len)
}

function _curry(fn, len, ...arg) {
  return function (...params) {
    let _arg = [...arg, ...params]
    if (_arg.length >= len) {
      return fn.apply(this, _arg)
    } else {
      return _curry.call(this, fn, len, ..._arg)
    }
  }
}

let fn = curry(function (a, b, c, d, e) {
  console.log(a + b + c + d + e)
})

fn(1, 2, 3, 4, 5) // 15
fn(1, 2)(3, 4, 5)
fn(1, 2)(3)(4)(5)
fn(1)(2)(3)(4)(5)
```

### 常见面试题

for 循环中使用闭包解决问题

```js
var data = []

for (var i = 0; i < 3; i++) {
  data[i] = function () {
    console.log(i)
  }
}

data[0]() // 3
data[1]() // 3
data[2]() // 3
```

这就是因为 i 是全局变量 i，共用一个作用域，所以最后 i 的值是 3，所以输出都是 3

使用闭包改善

```js
var data = []
for (var i = 0; i < 3; i++) {
  ;(function (j) {
    data[j] = function () {
      console.log(j)
    }
  })(i)
}
data[0]() // 0
data[1]() // 1
data[2]() // 2
```

不过也可以用 let 解决，因为 let 是块级作用域
