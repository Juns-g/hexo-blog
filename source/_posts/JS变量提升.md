---
title: JS变量提升
tags:
  - JS
  - 面试
  - 前端
categories: 面试
cover: 'https://www.doowebs.es/wp-content/uploads/2016/08/javascript-1.png'
abbrlink: 27856
date: 2024-01-23 15:50:17
---

# JS变量提升

> 参考链接：
>
> - [彻底解决 JS 变量提升| 一题一图，超详细包教包会😉](https://juejin.cn/post/6933377315573497864)



## 什么是变量提升？

举个例子

```js
console.log(a) // undefined
var a = 10
```

为什么打印的是undefined呢？就是由于js的变量提升机制，代码执行前，浏览器会将带有`var`, `function`关键词的变量提前进行声明（默认值就是undefined）。在变量提升解读啊，带`var`的只是声明还没有被定义，带`function`的已经声明和定义。

示例：

```js
var a = 12
var b = a
b = 1
function sum(x, y) {
  var total = x + y
  return total
}
sum(1, 2)
```

![图片](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/579be5ba5dd644f09e7c85ce9b366de1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

变量提升只发生在当前的作用域



## 带var和不带var的区别

- 全局作用域中可以不带`var`声明变量，但是建议带上，不带相当于给`window`设置一个属性
- 私有作用域，带`var`的是私有变量，不带的会向上级作用域查找
- 全局中使用`var`声明的变量会映射到`window`下称为属性

```js
a = 12 // == window.a
console.log(a) // 12
console.log(window.a) // 12

var a = b = 12 // 这里的 b 也是不带 var 的
// ! 相当于
var a = 12
b = 12
```

### 题目

下面来个思考题

```js
console.log(a, b)
var a = 12, b = 'bbb'
function foo() {
  console.log(a, b)
  var a = b = 13
  console.log(a, b)
}
foo()
console.log(a, b)
```

4处我的回答是：

```js
// undefined undefined
// 12 'bbb'
// 13 13
// 12 13
```

正确的应该是

```js
/* 输出：
    undefined undefined
    undefined "bbb"
    13 13
    12 13
*/
```

其中第二个就是局部作用域的变量提升引起的，可以看成这样

```js
function foo(){
  var a = 13
  b =13
}
```

所以a是变量提升，log的是undefined，b是向上找到window.b

再看一题：

```js
a = 2
function foo() {
  var a = 12
  b = 'bbb'
  console.log('b' in window)
  console.log(a, b)
}

foo()
console.log(b)
console.log(a)
```

我的回答是：

```js
// true
// 12 bbb
// bbb
// 2
```

因为b没有带`var`，所以就直接设置的window.b属性

下一题：

```js
function foo() {
  console.log(a)
  a = 12
  b = '林一一'
  console.log('b' in window)
  console.log(a, b)
}
foo()
```

可能先入为主以为可以正常log，实际上是直接报错`ReferenceError: a is not defined`

因为第一个log那里并没有找到window.a，下面的a也没有变量提升，所以直接报错

下一道是一个经典面试题：

```js
fn()
console.log(v1)
console.log(v2)
console.log(v3)
function fn() {
  var v1 = v2 = v3 = 111
  console.log(v1)
  console.log(v2)
  console.log(v3)
}
```

我的答案是：

```js
// 111
// 111
// 111
// 报错，因为没有v1
```

## 等号左边下的变量提升

函数左边的变量提升

- 普通函数

```js
print()
function print() {
  console.log('111')
}
print()
```

都输出111，很明显function有变量提升

- 匿名函数

```js
print()
var print = function () {
  console.log('111')
}
print()
```

直接报错`print is not a function`，因为一开始是`var print = undefined`，不是函数

## 条件判断下的变量提升

- if else 条件判断下的变量提升

```js
console.log(a)
if (false) {
  var a = '111'
}
console.log(a)
```

输出是2个undefined，因为：**在当前作用域中不管条件是否成立都会进行变量提升**

- 在if的`( )`中的表达式不会变量提升

```js
var y = 1
if (function f() {}) {
  console.log(typeof f) // undefined
  y = y + typeof f
}
console.log(y) // 1undefined
```

很好理解，直接是未定义

```js
console.log(a)
console.log(p())
if (true) {
  var a = 12
  function p() {
    console.log('111')
  }
}
// undefined
// Uncaught TypeError: p is not a function
```

### 题目

```js
if (!('val' in window)) {
  var val = 2019
}
console.log(val)
console.log('val' in window)

// undefined
// true
```

提升之后 window.val = undefined

## 重名问题下的变量提升

- 带 var 和 带 function 重名条件下，函数先执行变量提升

```js
console.log(a)// 函数
var a = 1
function a() {
  console.log(1)
}
```

或

```js
console.log(a) // 函数
function a() {
  console.log(1)
}
var a = 1
```

输出都是函数，也就是说函数声明的变量会覆盖var的变量提升

```js
var a = undefined
a = 函数
// log
```

- 函数名 和 var 声明的变量重名

```js
var a = 1
function a() {
  console.log(1)
}
console.log(a) // 1
```

因为：

```js
var a = undefined
a = '函数'
a = 1
// log
```

- 变量重名在提升阶段会重新定义/赋值

```js
console.log('1', fn())
function fn() {
  console.log(1)
}

console.log('2', fn())
function fn() {
  console.log(2)
}

console.log('3', fn())
var fn = '111'

console.log('4', fn())
function fn() {
  console.log(3)
}
```

则理解为

```js
var fn = undefined
fn = '函数 1'
fn = '函数 2'
fn = '函数 3'
// log 1
// log 2
// log 3
fn = '111'
// log 4
```

所以输出是：

```js
//  3
//  1 undefined
//  3
//  2 undefined
//  3
//  3 undefined
//  Uncaught TypeError: fn is not a function
```

### 思考题

```js
var a = 2
function a() {
  console.log(3)
}
console.log(typeof a) // number
```

```js
console.log(fn) // 函数fn
var fn = 2019
console.log(fn) // 2019
function fn() {}
```

```js
let a = 0,
  b = 0
function fn(a) {
  fn = function fn2(b) {
    console.log(a, b)
    console.log(++a + b)
  }
  console.log('a', a++)
}
fn(1) // a, 1 这里是入参
fn(2) // 2, 2   5 a拿的是fn作用域中的a=2
console.log(a, b) // 0 0
```

## 函数形参的变量提升

```js
function a(b) {
  console.log(b)
}
a(45)

// 等价于
// function a(b) {
//     var b = undefined;
//     b = 45;
// }
```

```js
var a = 1
function foo(a) {
  // var a = undefined
  // a = 1
  console.log(a) // 1
  var a
  console.log(a) // 1
}
foo(a)
```

### 题目

```js
var foo = '111'
;(function (f) {
  // var foo = undefined
  console.log(foo) // undefined
  var foo = f || 'hello'
  // foo = 111
  console.log(foo) // 111
})(foo)
console.log(foo) // 111
```

类似的

```js
var foo = '111'
;(function (foo) {
  // var foo = undefined
  // foo = '111
  console.log(foo) // 111
  var foo = foo || 'world'
  console.log(foo) // 111
})(foo)
console.log(foo) // 111
```

面试题

```js
var a = 10
;(function () {
  // var a = undefined
  console.log(a) // undefined
  a = 5
  console.log(a) // 5
  console.log(window.a) // 10
  // a = 20
  var a = 20
  console.log(a) // 20
})()

// var b = undefined
var b = {
  a,
  c: b, // b is undefined
}
console.log(b.c) // undefined
```

## 非匿名自执行函数的变量提升

- 非匿名自执行函数

```js
var a = 10
;(function c() {})()
console.log(c)
// ReferenceError: c is not defined
```

- 匿名自执行函数在自己的作用域内存在正常的变量提升

```js
var a = 10
;(function () {
  console.log(a) // 10
  a = 20
  console.log(a) // 20
})()
console.log(a) // 20
```

```js
var a = 10
;(function () {
  console.log(a) // undefined
  var a = 20
  console.log(a) // 20
})()
console.log(a) // 10
```

- 在自己的作用域内变量提升，修改函数名的值无效

```js
var a = 10
;(function a() {
  console.log(a) // 函数
  a = 20
  console.log(a) // 函数
})()
```

## 总结

感觉还是有点乱，这个有点难理解，面试前建议再看几遍

