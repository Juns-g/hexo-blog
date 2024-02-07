---
title: call、apply、bind
tags:
  - JS
  - 学习
cover: 'https://www.doowebs.es/wp-content/uploads/2016/08/javascript-1.png'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 3446
date: 2024-02-07 13:33:30
---

# call、apply、bind

## 前言

在前面已经了解了`this`，这篇文章讨论如何改变`this`的指向。

## 何为 this？

this 是用来指代这个函数当前的运行环境，可以这么理解：

```js
const me = { name: '111' }
const you = { name: '222' }

function sayName(context) {
  console.log(context.name)
}

sayName(me) // 111
sayName(you) // 222
```

我们可以把传入的执行环境简化，这时候就使用到了`this`

```js
const me = { name: '111' }
const you = { name: '222' }

function sayName() {
  console.log(this.name)
}

sayName.call(me) // 111
sayName.call(you) // 222
```

这里的`call`的参数就相当于传入上面的`context`

再看一个：

```js
const me = { name: '111', sayName }
const you = { name: '222', sayName }

function sayName() {
  console.log(this.name)
}

me.sayName() // 111
you.sayName() // 222
sayName() // undefined
// 相当于是 window.sayName，但是 window 没有 name 属性，所以是 undefined
```

这样就可以理解了，直接调用函数就是在`window`下调用，`this`指代的就是`window`对象

那么理解了`this`之后，我们如何改变`this`指代的运行环境呢？这里就需要`call`、`apply`、`bind`了

## 他们有什么用

我们已经知道了`call`等可以改变`this`指向，那么改变这个有啥用啊？还是举个例子来看：

```js
const me = {
  name: '111',
  say: function () {
    console.log(this.name)
  },
}

const you = { name: '222' }

me.say() // 111
```

在上面这个代码里，我想输出`you`的`name`，也就是借用`me`的`say`方法，怎么做？可以直接把这个函数提取成公共函数，然后再去给`you`添加这个属性，但是这也有问题，我不希望`you`多出来一个属性 ☝️，那用完了再删掉不就行，比如：

```js
const me = {
  name: '111',
  say,
}

function say() {
  console.log(this.name)
}

const you = {
  name: '222',
}

me.say() // 111

you.say = say
you.say()
delete you.say
```

这样当然可以，而且实际上这就是 call 做的 😆

同理，如果这个方法只在 me 上，也可以像这样去做，但是每次都这样写有点繁琐的，而且实际的 call 也有一些其他的处理，这时候我们就可以直接用 call 来处理这种情况

```js
const me = {
  name: '111',
  say: function () {
    console.log(this.name)
  },
}

const you = { name: '222' }

me.say() // 111
me.say.call(you) // 222
```

这样不就能够理解了，call 传入的`you`就是`say`的`this`

如果有参数呢？那就直接传参数呗 🤣👉🏻

```js
const me = {
  name: '111',
  say: function (state, text) {
    console.log(`${this.name} ${state}地说: ${text}`)
  },
}

const you = { name: '222' }

me.say('生气😡', '不许🐶叫') // 111 生气😡地说: 不许🐶叫
me.say.call(you, '开心😆', '我就叫') // 222 开心😆地说: 我就叫
```

这样的话就可以简单理解了 call 了，实际上 apply 和 bind 就是 call 的变种，apply 是传入数组，bind 是返回一个新函数

## 基本使用

### 语法

```js
fun.call(thisArg, param1, param2, ...)
fun.apply(thisArg, [param1, param2, ...])
fun.bind(thisArg, param1, param2, ...)
```

### 参数

`thisArg?`: 可选，调用函数时的`this`值

- fun 的 `this`指向他
- 非严格模式下，如果`thisArg`是`null`或`undefined`，`this`指向全局对象
- 严格模式下，`thisArg`传入`null`或`undefined`，`this`就是`null`或`undefined`
- 如果`thisArg`是原始值，`this`就是这个原始值的包装对象

`param1, param2, ...`: 可选，传入的参数

- call 和 bind: 除了第一个参数是`thisArg`，后面的参数都是传入的参数
- apply: 除了第一个参数是`thisArg`，第二个参数是数组，数组里面是传入的参数

## 应用场景

### 数据类型判断

看过数据类型判断八股的肯定都知道有一个方法最精准：`Object.prototype.toString.call`😈，他就用到了 call🤓。为什么使用 call 呢？因为其他的数据类型可能改写了原型上的`toString`方法。

```js
let oToString = Object.prototype.toString

const arr = [
  Object.prototype.toString,
  Array.prototype.toString,
  Function.prototype.toString,
  String.prototype.toString,
  Number.prototype.toString,
  Boolean.prototype.toString,
]

arr.forEach(item => {
  console.log(item === oToString) // 只有第一个是true😈
})
```

```js
function isType(data, type) {
  const typeMap = {
    '[object String]': 'string',
    '[object Number]': 'number',
    '[object Boolean]': 'boolean',
    '[object Null]': 'null',
    '[object Undefined]': 'undefined',
    '[object Object]': 'object',
    '[object Array]': 'array',
    '[object Function]': 'function',
    '[object Date]': 'date', // Object.prototype.toString.call(new Date())
    '[object RegExp]': 'regExp',
    '[object Map]': 'map',
    '[object Set]': 'set',
    '[object HTMLDivElement]': 'dom', // document.querySelector('#app')
    '[object WeakMap]': 'weakMap',
    '[object Window]': 'window', // Object.prototype.toString.call(window)
    '[object Error]': 'error', // new Error('1')
    '[object Arguments]': 'arguments',
  }
  const dataType = typeMap[Object.prototype.toString.call(data)] || '未知类型'
  return dataType === type
}

console.log(isType([], 'array'))
console.log(isType({}, 'object'))
```

## 手写

### call

先来想想思路，目的是借用方法，那么我可以把这个方法作为这个对象的一个属性，然后调用，之后再删除这个属性。

那么可以这么写：

```js
Function.prototype.myCall = function (context, ...args) {
  // 在 context 上添加属性
  context.fn = this
  // 以 context 为上下文执行函数
  const res = context.fn(...args)
  // 删除这个属性
  delete context.fn
  return res
}
```

当然这个肯定是不完善的，想想上面的基本使用，比如`thisArg`为原始值、null、undefined，以及 fn 这个名字可能会被占用，可以写出如下的修正版本

```js
Function.prototype.myCall = function (context, ...args) {
  if (context === null || context === undefined) {
    context = window
  } else {
    context = Object(context)
  }
  // 使用 Symbol 来避免重复
  const fn = Symbol('fn')
  context[fn] = this
  const res = context[fn](...args)
  delete context[fn]
  return res
}
```

### 手写 apply

实际的 apply 的第二个参数是可以接受类数组的，这里就不考虑这个情况了

```js
Function.prototype.myApply = function (context) {
  if (context === null || context === undefined) {
    context = window
  } else {
    context = Object(context)
  }

  const args = arguments[1]
  const fn = Symbol('fn')
  context[fn] = this
  let res = null
  if (args) {
    if (!Array.isArray(args)) {
      throw new Error('你传入的第二个参数不是数组')
    } else {
      // 如果是类数组就转化成数组
      const arr = Array.from(args)
      res = context[fn](...arr)
    }
  } else {
    res = context[fn]()
  }
  delete context[fn]
  return res
}
```

### 手写 bind

bind 需要返回一个函数

> TODO，涉及到了原型还有 new，这里先不写了

## 参考链接

- [js 基础-面试官想知道你有多理解 call,apply,bind？[不看后悔系列]](https://juejin.cn/post/6844903906279964686)
