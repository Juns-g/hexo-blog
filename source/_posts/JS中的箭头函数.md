---
title: JS中的箭头函数
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 20769
date: 2024-02-28 15:26:00
---

# JS 中的箭头函数

众所周知，箭头函数是 ES6 引入的新特性之一，我们也知道他相当于是简写的函数，还知道没有自己的 this，以及不能作为构造函数，但是具体深究一下看看，还有哪些是我们不知道的吧 🤷‍♂️

## 基本用法

函数声明：

```js
let foo = name => {
  console.log(name)
}
```

他也就等同于

```js
function foo(name) {
  console.log(name)
}
```

没啥区别，其中`(name)`外面的`()`是可以省略的。

然后呢他还有一些简写，比如`{ }`中直接就返回值了，可以这么简写：

```js
let foo = name => `name: ${name}`

let foo2 = name => {
  return `name: ${name}`
}
```

他们俩是一样的，不过需要注意，返回对象也是`{ }`😫，这个时候需要包一层`( )`

```js
let foo = name => ({ name })

let foo2 = name => {
  return {
    name,
  }
}
```

所以说箭头函数看上去更加简洁好用，行数也短一些，看着就舒服 😆

## 和普通函数的区别

直接 log 看看先 👀

```js
let fn = name => {
  console.log(name)
}

let fn2 = function (name) {
  console.log(name)
}

console.dir(fn)
console.dir(fn2)
```

![区别](https://pic.imgdb.cn/item/65df3a219f345e8d03bcd00a.jpg)

可以看出，箭头函数缺少了`arguments`、`caller`、`prototype`

### arguments

箭头函数没有自己的 arguments，但是他可以继承父级的 arguments

```js
let fn = name => {
  console.log(arguments)
}
let fn2 = function (name) {
  console.log(arguments)
}
fn2() // Arguments [callee: ƒ, Symbol(Symbol.iterator): ƒ]
fn() // 报错 Uncaught ReferenceError: arguments is not defined
```

可以看出，在全局作用域下，箭头函数是没有 arguments 的，但是在函数内部，他是可以继承父级的 arguments。

当他处于普通函数的作用域中的时候，arguments 就是上层普通函数的 arguments

```js
let fn2 = function (name) {
  console.log('fn2:', arguments)
  let fn = name => {
    console.log('fn:', arguments)
  }
  fn()
}
fn2('111')
// Arguments ['111', callee: ƒ, Symbol(Symbol.iterator): ƒ]
// Arguments ['111', callee: ƒ, Symbol(Symbol.iterator): ƒ]
```

但是我们也知道，ES6 还有一个 rest 参数，可以用它来起到 arguments 的作用：

```js
let fn = (a, ...rest) => {
  console.log(a, rest)
}

fn(1, 2, 3, 4, 5, 6)
// 1 [ 2, 3, 4, 5, 6 ]
```

### this 指向

和 arguments 一样，箭头函数没有自己的 this

```js
var testName = 'windowName'
const fn = () => {
  console.log(this.testName)
}

const fn2 = function () {
  console.log(this.testName)
}

fn() // windowName
fn2() // windowName
```

这里都是指向了 window。箭头函数是没有自己的 this 的，所以他的 this 是继承父级的 this，也就是 window

```js
var testName = 'windowName'
let obj = {
  testName: 'objName',
  fn: () => {
    console.log(this.testName)
  },
  fn2: function () {
    console.log(this.testName)
  },
}

obj.fn() // windowName
obj.fn2() // objName
```

这里就不一样了，fn2 的 this 就是指向当前的这个 obj，而 fn 的 this 是继承父级的 this，也就是 window。这个函数转换成 ES5 的语法就是这样：

```js
var _this = this // window
var testName = 'windowName'
var obj = {
  testName: 'objName',
  fn: function fn() {
    console.log(_this.testName)
  },
  fn2: function fn2() {
    console.log(this.testName)
  },
}
obj.fn() // windowName
obj.fn2() // objName
```

正是因为他没有自己的 this，所以 call、apply、bind 无法改变他的 this，他的 this 在声明的时候就已经确定了。

```js
var testName = 'windowName'
let obj = {
  testName: 'objName',
  fn: () => {
    console.log(this.testName)
  },
  fn2: function () {
    console.log(this.testName)
  },
}

obj.fn.call({ testName: '222' }) // windowName
obj.fn2.call({ testName: '222' }) // 222
```

### prototype

我们在之前学习了原型链相关的知识，也就知道，声明一个函数的时候，会自动创建他的原型对象 prototype，但是箭头函数可不会。

```js
let fn = name => {
  console.log(name)
}

let fn2 = function (name) {
  console.log(name)
}

console.log(fn.prototype) // undefined
console.log(fn2.prototype) // { constructor: ƒ }
```

### 不能作为构造函数

> [99.9%的人都不知道的箭头函数不能当做构造函数的秘密](https://juejin.cn/post/7050492355056664612)

在前面我们也学习了 new 的原理：

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

可以看到，使用 new 创建一个实例的时候，需要用到构造函数的 prototype，还需要改变构造函数的 this 指向，但是箭头函数没有自己的 prototype，也没有自己的 this，所以他不能作为构造函数。

### yield

箭头函数不可以使用 yield 命令，所以不能用作 Generator 函数，这方面我还不太了解，仅仅是知道。

## 使用场景

### 回调函数

主要还是由于普通函数的 this 指向问题，导致作为回调函数时，需要使用 bind、apply、call 来改变 this 指向，而箭头函数没有自己的 this，所以就不会有这个问题。

比如 forEach、map，就经常使用箭头函数作为回调函数，主要是看着简洁，没有 this 问题。

```js
const words = ['hello', 'WORLD', 'Whatever']
const downcasedWords = words.map(word => word.toLowerCase())
```

## 不建议使用的情况

### 对象上的方法

我们知道，箭头函数没有自己的 this，但是对象方法我们可能需要使用 this，所以说最好不用箭头函数就行，可以直接用方法的简写

```js
let obj = {
  name: '111',
  fn: () => {
    console.log(this.name) // 不推荐，不是111
  },
  fn2() {
    console.log(this.name) // 推荐，是111
  },
  fn3: function () {
    // 和fn2一样，fn2是简写
    console.log(this.name)
  },
}
```

### 定义原型方法

我们也知道，在原型上定义方法，一般需要访问 this，箭头函数没有自己的 this 又会导致问题咯

```js
function Cat(name) {
  this.name = name
}

Cat.prototype.sayCatName = () => {
  console.log(this === window) // => true
  return this.name
}

const cat = new Cat('Mew')
cat.sayCatName() // => undefined
```

### 定义事件回调函数

这个是需要分情况的，有的时候回调函数的 this 指代当前发生事件的 DOM 节点，这个时候用 this 就很舒服，比如：

```js
const button = document.getElementById('myButton')
button.addEventListener('click', function () {
  console.log(this === button) // => true
  this.innerHTML = 'Clicked button'
})
```

这里使用箭头函数就会造成 this 指代 window

### 作为类中的方法

这里推荐看这个文章：[曾经我以为我很懂箭头函数](https://juejin.cn/post/6844904152640782343)

## 题目

```js
window.name = 'window_name'

let obj1 = {
  name: 'obj1_name',
  print: () => console.log(this.name),
}

let obj2 = { name: 'obj2_name' }

obj1.print() // 问 输出结果
obj1.print.call(obj2) // 问 输出结果
```

<details>
  <summary>答案👈🏻</summary>
  <pre>都是'window_name'</pre>
</details>

```js
function A() {
  this.foo = 1
}

A.prototype.bar = () => console.log(this.foo)

let a = new A()
a.bar() // 问 输出结果
```

<details>
  <summary>答案👈🏻</summary>
  <pre>'undefined'，因为window没有foo属性</pre>
</details>

```js
let obj1 = {
  name: '111',
  print: function () {
    return () => console.log(this.name)
  },
}

let obj2 = { name: '222' }

obj1.print()() // 问 输出结果
obj1.print().call(obj2) // 问 输出结果
obj1.print.call(obj2)() // 问 输出结果
```

<details>
  <summary>答案👈🏻</summary>
  <pre>'111', '111', '222'，自己分析吧🧐</pre>
</details>

```js
var flag = 'window flag'
function Person(fg) {
  let o = {
    flag: fg,
    fn: () => {
      console.log(this)
    },
    fn2() {
      console.log(this)
    },
  }
  return o
}

let p = new Person('111')
p.fn()
p.fn2()
```

<details>
  <summary>答案👈🏻</summary>
  <pre>
    Person {}
    { flag: 'window flag', fn: [Function: fn], fn2: [Function: fn2] }
    箭头函数里面的this，就是Person函数的this
  </pre>
</details>

改编一下上一题：

```js
var flag = 'window flag'
function Person(fg) {
  let flag = 'Person flag'
  this.flag = 'Person this flag'
  let _this = this
  let o = {
    flag: fg,
    fn: () => {
      console.log(this.flag)
      console.log(this === _this)
    },
    fn2() {
      console.log(this.flag)
    },
  }
  return o
}

let p = new Person('111')
p.fn()
p.fn2()
```

<details>
  <summary>答案👈🏻</summary>
  <pre>
    Person this flag
    true
    111
  </pre>
</details>

上面两题推荐看这个文章：[对阮一峰《ES6 入门》中箭头函数 this 描述的探究](https://juejin.cn/post/6844904133409914894)，这个还是比较综合的。

下面这一题慢慢分析吧，原文分析在这里：[2022 年了你还不了解箭头函数与普通函数的区别吗？](https://juejin.cn/post/7069943937577779214#heading-18)

```js
var name = '南玖'
function Person(name) {
  this.name = name
  ;(this.foo1 = function () {
    console.log(this.name)
  }),
    (this.foo2 = () => console.log(this.name)),
    (this.foo3 = function () {
      return function () {
        console.log(this.name)
      }
    }),
    (this.foo4 = function () {
      return () => {
        console.log(this.name)
      }
    })
}
var person1 = new Person('nan')
var person2 = new Person('jiu')

person1.foo1() // 'nan'
person1.foo1.call(person2) // 'jiu'

person1.foo2() // 'nan'
person1.foo2.call(person2) // 'nan'

person1.foo3()() // '南玖'
person1.foo3.call(person2)() // '南玖'
person1.foo3().call(person2) // 'jiu'

person1.foo4()() // 'nan'
person1.foo4.call(person2)() // 'jiu'
person1.foo4().call(person2) // 'nan'
```

## 参考文章

- [「ES6 系列」彻底弄懂箭头函数(内附习题)](https://juejin.cn/post/6844903862365585416)
- [什么时候你不能使用箭头函数？](https://juejin.cn/post/6844903475554287629)
- [[译]JS 箭头函数三连问：为何用、怎么用、何时用](https://juejin.cn/post/6844903711098028046)
