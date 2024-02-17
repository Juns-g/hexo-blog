---
title: JS中的原型与原型链
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 27736
date: 2024-02-16 19:03:16
---

# JS 中的原型与原型链

这也是 JS 中的一个重要的概念，我个人一直理解他们的作用是为了更好的复用函数。他包含了`__proto__` 和 `prototype`，这两个东西经常让人搞混，这里来看看。

## 为什么需要他们

看个例子:

```js
function Person(name, age) {
  this.name = name
  this.age = age
  this.eat = function () {
    console.log(age + '岁的' + name + '在吃饭。')
  }
}

let p1 = new Person('111', 24)
let p2 = new Person('222', 24)

console.log(p1.eat === p2.eat) // false
```

这里我们可以看到，通过`new`生成的实例，都开了一块新的内存，所以他们的`eat`函数是不一样的，这就导致了内存的浪费，所以说我们可以用一个共享的地方，让实例可以去这里找到这个函数，这就是原型链的作用。

> 原型链的基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

```js
function Person(name, age) {
  this.name = name
}

Person.prototype.eat = function () {
  console.log('eat')
}

let p1 = new Person('111')
let p2 = new Person('222')

console.log(p1.eat === p2.eat) // true
```

这里他们的 eat 函数就相同了。

## 三者 🤓

理解原型对象还有原型链，就需要搞懂这三者之间的关系：`prototype`、`__proto__` 和 `constructor`

这三者依附在不同的引用对象类型上

- 对象：`__proto__`、`constructor`是对象独有的
- 函数：
  - `prototype`是函数独有的，用来存放实例共享的属性和方法
  - 因为函数也是对象，所以函数也有`__proto__`、`constructor`

### prototype

![prototype](https://tsejx.github.io/javascript-guidebook/static/explicit-prototype.83d1754b.jpg)

**他是函数独有的，从一个函数指向一个对象**，这个对象就是用来存放实例共享的属性和方法的。他的含义就是**函数的原型对象**，也就是函数创建的实例的原型对象，也叫显示原型对象。

因此可知：

```js
function Fn() {}
let f = new Fn()
```

其中 `f.__proto__ === Foo.prototype`

既然`prototype`是存放公用东西的地方，那么其实他就是可以让实例对象来访问公用东西的，通过`__proto__`来访问。

**任何函数在创建的时候，其实会默认同时创建该函数的 `prototype` 对象**

### `__proto__`

![__proto__](https://tsejx.github.io/javascript-guidebook/static/hidden-prototype.bee5023d.jpg)

对象都有这个属性，**从一个对象指向另一个对象**，这个被指向的对象就是他的原型对象，也称为隐式原型对象。

他的作用就是，当我访问一个对象的属性/方法时，如果这个对象里面没有，就可以从他的`__proto__`指向的原型对象中寻找。如果还是找不到，就沿着原型对象的`__proto__`继续找，知道找到要的东西，或者找到顶层原型对象`null`，返回 undefined。

这个查找的过程，从当前对象出发，沿着原型对象构成的链接寻找，就是原型链。

### constructor

![constructor](https://tsejx.github.io/javascript-guidebook/static/constructor.f2783bdc.jpg)

他也叫构造函数，是对象所独有的，**从一个对象指向一个函数**，含义是**对象的构造函数**。

每个对象都有构造函数（本身拥有/继承），这个构造函数就是用来创建这个对象的函数。

其中可以看到`Function`对象比较特殊，他的构造函数是自己，因为他可以看成函数/对象，所有的函数/对象最终都是由`Function`构造函数得来，所以说`constructor`属性的重点就是`Function`。

## 原型对象

在上面我们其实有俩种原型对象，一种是函数的原型对象，一种是对象的原型对象，分别叫做显式原型对象和隐式原型对象。

| 显式原型对象                   | 隐式原型对象                                          |
| ------------------------------ | ----------------------------------------------------- |
| 属性 `prototype`               | 属性 `__proto__`                                      |
| 函数独有                       | 对象独有(函数也是对象，所以函数也有)                  |
| 定义函数时自动复制，默认为`{}` | 创建实例对象时自动添加，赋值为构造函数的`prototype`值 |
| 实现原型的继承和属性的共享     | 构成原型链，实现原型的继承                            |

## 原型对象的指向

- 字面量方式
  这时候原型就是`Object.prototype`

```js
const { log } = console
const obj = {}

log(obj.__proto__ === Object.prototype) // true
log(Object.getPrototypeOf(obj) === Object.prototype) //true
```

- 构造器方式
  这时候原型就是构造函数的`prototype`

```js
const { log } = console

function Foo() {}
const f = new Foo()

log(f.__proto__ === Foo.prototype) // true
log(Object.getPrototypeOf(f) === Foo.prototype) //true
```

- `Object.create()`
  会把传入的对象作为原型

```js
const { log } = console

const foo = {}
const bar = Object.create(foo)

log(bar.__proto__ === foo) // true
log(Object.getPrototypeOf(bar) === foo) //true
```

## 看点题目

### 1

```js
const A = function () {}
A.prototype.n = 1
const b = new A()

A.prototype = {
  n: 2,
  m: 3,
}
const c = new A()

console.log(b.n) // 1
console.log(b.m) // undefined

console.log(c.n) // 2
console.log(c.m) // 3
```

当创建 A 时，已经创建了`A.prototype`对象为`{ }`，创建 b 的时候，把`A.prototype`对象的引用赋值给了`b.__proto__`，所以`b`的`__proto__`指向`A.prototype`，所以`b`的`n`属性就是`1`。

然后修改了`A.prototype`引用的地址，所以`c`的`__proto__`指向了新的`A.prototype`对象，所以`c`的`n`属性就是`2`，`m`属性就是`3`。

### 2

```js
const F = function () {}

Object.prototype.a = function () {
  console.log('a')
}

Function.prototype.b = function () {
  console.log('b')
}

var f = new F()

f.a() // a
f.b() // f.b is not a function

F.a() // a
F.b() // b
```

首先我们看`f`，他是通过构造函数`F`创建的实例，所以他的`__proto__`指向`F.prototype`，而`F.prototype`的`__proto__`指向`Object.prototype`，所以可以写出这样的原型链`f -> F.prototype -> Object.prototype`。他并没有涉及到 `Function`，所以第二个 log 报错。

再看`F`，他是函数，由`Function`构造，所以`F.__proto__`指向`Function.prototype`，而`Function.prototype`的`__proto__`指向`Object.prototype`。所以可以写出这样的原型链`F -> Function.prototype -> Object.prototype`。所以第一个 log 输出`a`，第二个 log 输出`b`。

### 3

```js
function Person(name) {
  this.name = name
}
let p = new Person('Tom')

console.log(p.__proto__ === Person.prototype) // true
console.log(Person.__proto__ === Function.prototype) // true
```

1. `p.__proto__`等于什么？
2. `Person.__proto__`等于什么？

1: `p` 是 `Person` 构造出来的，所以`p.__proto__`等于`Person.prototype`
2: Person 是一个函数，所以`Person.__proto__`等于`Function.prototype`

### 4

```js
const foo = {}
const F = function () {}

Object.prototype.a = 'a'
Function.prototype.b = 'b'

console.log(foo.a) // 1
console.log(foo.b) // undefined

console.log(F.a) // a
console.log(F.b) // b
```

直接看上文吧，这个简单

## 参考文章

- [2019 面试准备 - JS 原型与原型链](https://juejin.cn/post/6844903782229213197)
- [JavaScript Guidebook 原型链](https://tsejx.github.io/javascript-guidebook/object-oriented-programming/inheritance/prototype-chain)
