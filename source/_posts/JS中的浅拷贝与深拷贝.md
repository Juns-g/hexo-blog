---
title: JS中的浅拷贝与深拷贝
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 35313
date: 2024-02-17 19:42:16
---

# JS 中的浅拷贝与深拷贝

拷贝数据这是一个很常见的操作，像基本类型我们可以直接赋值，没啥影响 🥳，但如果是引用类型的话，直接赋值是会出问题的，看个小例子：

```js
let obj = { name: '111' }

let copyObj = obj
copyObj.name = 'copy'

console.log(copyObj) // { name: 'copy' }
console.log(obj) // { name: 'copy' }
```

我们可以发现一个很明显的问题，改的是`copyObj`啊，为啥子原来的`obj`里面的东东也被改了 😭？

这就涉及到引用类型的存储方式了，引用类型的存储是存储在堆内存中的，而栈内存中存储的是指向堆内存中的地址。所以说，我们上面的赋值就是把地址给了`copyObj`，他们俩指向的是同一个地址，所以改一个另一个也会被改。

![复制引用类型](https://tsejx.github.io/javascript-guidebook/static/refered-type-copy-sample.fef48b0d.jpg)

那么我们怎么解决这个问题呢？这就需要了解浅拷贝和深拷贝了 🤗。

## 浅拷贝

浅拷贝指的是只复制对象或数组本身，而不复制它们内部引用的其他对象或数组。也就是说，浅拷贝会创建一个新的对象或数组，并将原始对象或数组中的元素复制到新的对象或数组中，但是这些元素仍然是原始对象或数组中元素的引用。

当然实现浅拷贝的方式就挺多的了

### [Object.assign()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

> MDN: Object.assign() 静态方法将一个或者多个源对象中所有可枚举的自有属性复制到目标对象，并返回修改后的目标对象。

看他的介绍我们就知道，可以复制 😁

```js
let obj = { name: '111' }

let copyObj = Object.assign({}, obj)
copyObj.name = 'copy'

console.log(copyObj) // { name: 'copy' }
console.log(obj) // { name: '111' }
```

### 展开语法

他就相当于`Object.assign()`的语法糖，也可以实现浅拷贝

```js
let obj = { name: '111' }

let copyObj = { ...obj }
copyObj.name = 'copy'

console.log(copyObj) // { name: 'copy' }
console.log(obj) // { name: '111' }
```

他也可以合并对象，后面的属性会覆盖前面的属性

```js
let obj1 = { name: '111', x: 1 }
let obj2 = { name: '222', y: 2 }

let mergeObj = { ...obj1, ...obj2 }
// { name: '222', x: 1, y: 2 }
```

### 数组

数组也和对象差不多，有几种浅拷贝的方式，具体也不展开说这几个 api 了

```js
let arr = [1, 2, 3]

let copyArr = [...arr]
let copyArr2 = [].concat(arr)
let copyArr3 = arr.slice()
```

### for ... in

我们也可以直接遍历对象，然后手动复制

```js
let obj = { name: '111' }
let copyObj = {}

for (let key in obj) {
  if (obj.hasOwnProperty(key)) {
    // 不复制原型上的属性
    copyObj[key] = obj[key]
  }
}
```

或者用迭代器

```js
let obj = { name: '111' }
let copyObj = {}

Object.entries(obj).forEach(([key, val]) => {
  copyObj[key] = val
})
```

### 写个函数

我们直接写一个函数来实现 array 和 object 的浅拷贝

```js
function shallowCopy(obj) {
  if (typeof obj !== 'object' || obj === null) {
    return obj
  }

  let newObj = obj instanceof Array ? [] : {}
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      newObj[key] = obj[key]
    }
  }

  return newObj
}
```

## 深拷贝

深拷贝就需要完全复制了，不仅复制对象或数组本身，还要复制它们内部引用的其他对象或数组。

### [structuredClone()](https://developer.mozilla.org/zh-CN/docs/Web/API/structuredClone)

这是去年才出的 api，他可以完全复制一个对象。爆杀我们手写的 🤣，支持各种类型，也解决了循环引用的问题。

```js
const original = { name: 'MDN' }
original.itself = original

const clone = structuredClone(original)

console.log(clone !== original) // true
console.log(clone.name === 'MDN') // true
console.log(clone.itself === clone) // true
```

### JSON 序列化和反序列化

这是最简单的写法了，虽然他会有一些问题，我么可以想到，对象转字符串再转回来，就是深拷贝了，也很合理 🤪

```js
const obj = {
  name: '111',
  arr: [2, 3, { name: 'arr' }],
}

const copyObj = JSON.parse(JSON.stringify(obj))
copyObj.arr[2].name = 'copy'

console.log(obj)
// { name: '111', arr: [ 2, 3, { name: 'arr' } ] }
console.log(copyObj)
// { name: '111', arr: [ 2, 3, { name: 'copy' } ] }
```

不过吗，简单也是有代价的，她还是有不少问题的：

#### 函数 、`undefined`、`symbol` 直接丢失

```js
const obj = {
  fn: function () {},
  foo: () => {},
  un: undefined,
  symbol: Symbol('11'),
  nl: null,
}

console.log(JSON.stringify(obj)) // {"nl":null}
```

#### NaN、Infinity 和-Infinity 会变成 null

```js
const obj = {
  nan: NaN,
  inf: Infinity,
  '-inf': -Infinity,
}

console.log(JSON.stringify(obj))
// {"nan":null,"inf":null,"-inf":null}
```

#### Date 会变成字符串

```js
const obj = {
  date: new Date(),
}
console.log('🚀 ~ obj:', obj)

let str = JSON.stringify(obj)
console.log(str)
// {"date":"2024-02-17T12:54:27.619Z"}
console.log(JSON.parse(str))
// { date: '2024-02-17T12:55:03.978Z' }
```

#### 变成空对象

```js
const obj = {
  map: new Map([[1, 2]]),
  reg: new RegExp(/ab+c/),
  set: new Set([2, 3]),
  err: Error('11'),
}

console.log(JSON.stringify(obj))
// {"map":{},"reg":{},"set":{},"err":{}}
```

#### 循环引用会报错

```js
const obj = {
  itself: null,
}
obj.itself = obj

console.log(JSON.stringify(obj))
// Uncaught TypeError: Converting circular structure to JSON
```

### 递归

之前遇到的问题也是内部如果有引用类型，就会直接赋值地址，那么我们直接递归一下不就可以啦 😆

```js
function deepCopy(target) {
  if (typeof target !== 'object' || target === null) return target

  let newObj = Array.isArray(target) ? [] : {}
  for (let key in target) {
    if (target.hasOwnProperty(key)) {
      newObj[key] = deepCopy(target[key])
    }
  }
  return newObj
}
```

不过其实这个深拷贝也是有缺陷的:

1. 不支持循环引用：会造成栈溢出
2. 特殊对象无法处理：如 Date、RegExp、Error、Function、undefined、Map、Set
3. 会丢失原型链

### 考虑循环引用

对于上面的函数，我们这样执行会出现死循环

```js
let obj2 = { name: '11' }
obj2.itself = obj2
deepCopy(obj2)
// Uncaught RangeError: Maximum call stack size exceeded
```

解决这个问的话，我们可以额外开辟一个存储空间，来存储当前对象和拷贝对象的对应关系，当需要拷贝当前对象时，先去存储空间中找，有没有拷贝过这个对象，如果有的话直接返回，如果没有的话继续拷贝，这样就可以循环引用的问题。

```js
function deepCopy(target, map = new Map()) {
  if (typeof target !== 'object' || target === null) return target

  if (map.has(target)) return map.get(target)
  let newObj = Array.isArray(target) ? [] : {}
  map.set(target, newObj)

  for (let key in target) {
    if (target.hasOwnProperty(key)) {
      newObj[key] = deepCopy(target[key], map)
    }
  }
  return newObj
}

let obj2 = { name: '11' }
obj2.itself = obj2
deepCopy(obj2)
```

其实我们还可以使用弱引用 WeakMap 来优化性能，因为 Map 是强引用，当对象作为键/值的时候，即使我们释放了对象的内存，他们之间的引用并不会自动删除，会造成内存消耗比较大。

> 这里涉及到了垃圾回收机制，暂时还不熟，建议不要给自己挖坑 😖

### 多种类型

上面其实我们也就只考虑了数组和对象，但是实际情况复杂的多

- 基本数据类型: string, number, boolean, symbol, undefined，他们直接返回就好
- function: 其实函数的拷贝没啥价值，直接返回内存地址就可以了
- 可继续遍历的类型: object, array, set, map
- 不可继续遍历的类型: Number, String, Boolean, Date, Error

所以我们可以这样完善：

```js
const getType = data => Object.prototype.toString.call(data).slice(8, -1)

function deepCopy(target, map = new WeakMap()) {
  if (typeof target !== 'object' || target === null) {
    return target
  }

  const type = getType(target)
  const Ctor = target.constructor
  const types = ['Number', 'String', 'Boolean', 'Date', 'Error', 'RegExp']
  if (types.includes(type)) {
    return new Ctor(target)
  }

  let cloneTarget = new Ctor()
  if (map.has(target)) return map.get(target)
  map.set(target, cloneTarget)

  if (type === 'Map' || type === 'WeakMap') {
    target.forEach((val, key) => {
      cloneTarget.set(key, deepCopy(val, map))
    })
    return cloneTarget
  }

  if (type === 'Set' || type === 'WeakSet') {
    target.forEach(val => {
      cloneTarget.add(deepCopy(val, map))
    })
  }

  // array / object
  for (let key in target) {
    if (target.hasOwnProperty(key)) {
      cloneTarget[key] = deepCopy(target[key], map)
    }
  }

  return cloneTarget
}
```

## 参考文章

- [如何写出一个惊艳面试官的深拷贝?](https://juejin.cn/post/6844903929705136141)
- [JavaScript 提升：掌握深拷贝与浅拷贝的技巧及如何手写深拷贝](https://juejin.cn/post/7305213173896773658)
- [JS 深拷贝的原生终结者 structuredClone API](https://juejin.cn/post/7080433165264748557)
