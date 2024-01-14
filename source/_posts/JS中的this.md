---
title: JS中的this
date: 2024-01-14 15:15:14
tags:
  - TS
  - 学习
cover: 'https://www.doowebs.es/wp-content/uploads/2016/08/javascript-1.png'
keywords: 学习 前端 JS
categories: 前端
---



# JS中的this

在js里面，`this`是一个关键字，他的值实际上取决于函数的调用方式。`this`的作用是指向当前执行代码的对象，在不同的场景下会有不同的值，所以`this`的指向是动态的。



## 为什么要用this？

从概念上理解可能有点费劲，下面举例说明

```js
function identify() {
  return this.name.toUpperCase()
}

var me = {
  name: 'Kyle',
}

var you = {
  name: 'Reader',
}

identify.call(me) // KYLE
identify.call(you) // READER
```

在没有学习之前我们可能不明白这是为什么，那么可以换一个思路，将`context`环境对象传给函数：

```js
function identify(context) {
  return context.name.toUpperCase()
}
identify(you) // READER
```

结果是一样的，我们其实可以直接把环境对象传入，这样看着就会比较直观。但是，当模式越发复杂的时候，将执行环境作为一个明确的参数传递给函数，就会比较混乱了。

所以`this`机制可以以一种更优雅的方式来隐含传递



## this的原理

为什么`this`指向的就是函数运行的执行环境呢？

这里简单描述一下，下面是一个简单的例子

```js
const obj = { foo: 5 }
```

上面这个代码将一个对象赋值给`obj`，js引擎会先在内存里面生成一个对象`{ foo: 5 }`，然后把这个对象的内存地址赋值给变量`obj`

也就是说，`obj	`其实就只是一个地址。如果读取`obj.foo`，js引擎会先从`obj`拿到内存地址，然后再从地址读出原始对象，然后返回他的`foo`属性。

这样的结构是很清晰的，但是如果属性的值是一个函数呢？

```js
const obj = { foo: function () {} };
```

这时，js引擎会把函数单独保存在内存中，然后把函数的地址赋值给`foo`属性的`value`属性，所以函数可以在不同的环境中执行

那么就需要一种机制，能够在函数体内部获得当时的运行环境，所以就出现了`this`，他的设计目的就是**在函数体内部，指代函数当前的运行环境**



## 几个规则



### 默认绑定规则

函数在哪个词法作用域中生效，`this`就指向哪里

比如：

```js
var a = 1
function fn() {
  console.log('🚀 ~ this.a:', this.a)
  console.log('🚀 ~ this:', this)
}
fn()
// 1
// window
```

解释为：在浏览器的控制台运行，`this`指向`window`。这里需要明白一个概念，`this`所在的函数在哪里生效，那么`this`就指向这个地方的词法作用域。由于函数没有词法作用域，只有作用域，所以这里指向`window`这个全局词法作用域。



再来一个例子，把`fn`放到`bar()`函数中调用，`this`指向哪里？

```js
var a = 1

function fn() {
  console.log('🚀 ~ this.a:', this.a)
  console.log('🚀 ~ this:', this)
}

function bar() {
  var a = 2
  fn()
}

bar()
// 1
// window
```

结果同样还是`window`，其实就如同上面说的一样，`this`指向函数生效的词法作用域，`this`在`bar()`中被调用，所以指向`window`



再看一个：

```js
function fn() {
  function bar() {
    console.log('🚀 ~ this:', this)
  }
  bar()
}
fn()
// window
```

原理同上，因为函数没有词法作用域，所以向上找到`window`这个全局作用域



总结：独立调用的函数`this`指向`window`，因为`this`指向词法作用域而函数没有词法作用域，只能指向全局词法作用域`window`



### 隐式绑定规则

当一个函数被对象拥有，且调用时，函数的`this`指向对象

```js
function fn() {
  console.log('🚀 ~ this.a:', this.a)
  console.log('🚀 ~ this:', this)
}
var obj = {
  a: 1,
  fn,
}
obj.fn()
// 1
// obj
```

这并不是独立调用，`this`指向`obj`



### 隐式丢失

当函数被多个对象链式调用时，`this`就指向引用函数的那个对象

```js
var obj = {
  a: 1,
  fn,
}
var obj2 = {
  a: 2,
  obj,
}
function fn() {
  console.log('🚀 ~ this.a:', this.a)
  console.log('🚀 ~ this:', this)
}
obj2.obj.fn()
// 1
// obj
```



### 显式绑定

`call`,`bind`,`apply`，那就是直接指向绑定过去的了

```js
function fn() {
  console.log('🚀 ~ this.a:', this.a)
  console.log('🚀 ~ this:', this)
}
var obj = {
  a: 1,
}
// fn.call(obj)
// fn.apply(obj)
let bar = fn.bind(obj)
bar()
// 1
// obj
```



### new绑定

指向实例对象

```js
function Person(name, age) {
  this.name = name
  this.age = age
}

var person1 = new Person('Alice', 25)
console.log(person1.name) // Alice
console.log(person1.age) // 25
```



### 箭头函数

箭头函数没有自己的`this`



## 练习

```js
var myObj = {
  name : " 北宸 ",
  showThis: function(){
    this.name = " 南蓁 "
    console.log(this)
  }
}
var foo = myObj.showThis
foo() // window, 在全局环境中调用一个函数就指向window
myObj.showThis() // myObj
```



```js
var myObj = {
  name: ' 北宸南蓁 ',
  showThis: function () {
    console.log('showThis:', this) // myObj
    function inner() {
      console.log('inner', this) // window
    }
    inner()
  },
}
myObj.showThis()
// 嵌套函数中的内部函数的this不会从外层函数中继承
```



```js
const myObj = {
  name: '111',
  showThis: function () {
    console.log('showThis:', this) // myObj 111
    const self = this
    function inner() {
      self.name = '222'
      console.log('inner:', self) // myObj 222
    }
    inner()
  },
}
myObj.showThis()
console.log(myObj.name) // 111
// 使用一个变量来保存外部this
```



```js
var myObj = {
  name: '111',
  showThis: function () {
    console.log('showThis:', this) // myObj 111
    var inner = () => {
      console.log('inner:', this) // myObj 111
    }
    inner()
  },
}
myObj.showThis()
// 箭头函数不会创建自身的执行上下文，所以取决于外部函数
```



```js
var a = 5
console.log('🚀 ~ this.a:', this.a) // 5

function test() {
  this.a = 1
  console.log('🚀 ~ this.a:', this.a) // 1
}

let b = test()
console.log('🚀 ~ this.a:', this.a) // 1
```



```js
window.name = 'ByteDance'
function A() {
  this.name = 123
}
A.prototype.getA = function () {
  console.log(this)
  return this.name + 1
}
let a = new A()
let funcA = a.getA
funcA() 
// ByteDance, 单独调用函数，指向window
```



```js
window.name = 'ByteDance'
function A() {
  this.name = 123
}
A.prototype.getA = function () {
  console.log(this)
  return this.name + 1
}
let a = new A()
a.getA()
// 123 因为this指向a，a.name = 123
```



```js
window.name = 'ByteDance'
class A {
  constructor() {
    this.name = 123
  }
  getA() {
    console.log(this)
    return this.name + 1
  }
}
let a = new A()
let funcA = a.getA
funcA()
// undefined 然后 报错，因为调用funcA的时候this是undefined
// 可以修改成 let funcA = a.getA.bind(a)
```



```js
window.name = 'ByteDance'
class A {
  constructor() {
    this.name = 123
  }
  getA() {
    console.log(this)
    return this.name + 1
  }
}
let a = new A()
a.getA()
// this指向A
```



```js
var length = 10

function fn() {
  console.log(this)
  console.log(this.length)
}

var obj = {
  length: 5,
  method: function (fn) {
    // 大家可以试着，把 function 里的参数 fn 去掉，
    // 并不影响 fn 的调用者，还是 window
    fn()
    // arguments 有 2 个参数，fn 和 1，
    // 当执行 arguments[0]() 等价于调用 fn()，
    // arguments是一种特殊的对象，在这里作为 fn 的调用者
    arguments[0]()
  },
}

obj.method(fn, 1)
// 10
// 2
```

对于上一题，如何让`fn()`输出`obj`的`length`呢？

```js
var length = 10

function fn() {
  console.log(this.length)
}

var obj = {
  length: 5,
  method: function (fn) {
    fn.call(obj)
    // 或者 fn.call(this)
  },
}

obj.method(fn, 1)
```



```js
window.val = 1

var obj = {
  val: 2,
  dbl: function () {
    this.val *= 2
    val *= 2
    console.log('val:', val)
    console.log('this.val:', this.val)
  },
}

// 说出下面的输出结果
obj.dbl()
var func = obj.dbl
func()
// 2, window的val
// 4, obj的val

// 8, 2 * 2 * 2, window
// 8, window
```



## 参考文章

- [「前端面试题系列4」this的原理以及用法](https://juejin.cn/user/3333374985382749/posts)
