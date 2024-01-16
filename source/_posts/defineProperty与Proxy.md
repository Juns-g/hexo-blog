---
title: Object.defineProperty与Proxy
tags:
  - JS
  - 学习
cover: 'https://www.doowebs.es/wp-content/uploads/2016/08/javascript-1.png'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 28864
date: 2024-01-15 17:22:52
---

# Object.defineProperty与Proxy

在Vue2中，响应式采用`Object.defineProperty`来实现，而在Vue3中，使用的是`Proxy`。同时，这也是一个十分常见的面试知识点，这里先了解一下这俩个api的作用和功能，之后下一篇文章来深入一下Vue的响应式原理。



## [Object.defineProperty](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)

直接在一个对象上定义一个新属性，或修改其现有属性，并返回此对象。

```js
Object.defineProperty(obj, prop, descriptor)
```

```ts
type PropertyKey = string | number | symbol // 第二个参数

// 第三个参数,详细含义见下文
interface PropertyDescriptor {
  configurable?: boolean
  enumerable?: boolean
  value?: any
  writable?: boolean
  get?(): any
  set?(v: any): void
}
```



### 基本使用

```js
const obj = {}

Object.defineProperty(obj, 'name', {
  value: '666',
})

console.log('🚀 ~ obj.name:', obj.name)
// 666
```

监听person上的name属性变化：

```js
const person = {}
let personName = 'John'

// 给 person 添加属性 name，值为 personName
Object.defineProperty(person, 'name', {
  // 默认不可枚举（for...in、Object.keys()）
  // 想枚举可以：enumerable: true
  // 默认不可修改，想修改可以：writable: true
  // 默认不可删除，想删除可以：configurable: true
  get: function () {
    console.log('🚀 ~ get:')
    return personName
  },
  set: function (val) {
    console.log('🚀 ~ set:', val)
    personName = val
  },
})

// 读取person.name, 触发get方法
console.log('🚀 ~ person.name:', person.name)
// 🚀 ~ get:
// 🚀 ~ person.name: John

// 修改personName,重新访问person.name, 修改成功
personName = 'Tom'
console.log('🚀 ~ person.name:', person.name)
// 🚀 ~ get:
// 🚀 ~ person.name: Tom

// 修改person.name, 触发set方法
person.name = 'Jerry'
console.log('🚀 ~ person.name:', person.name)
// 🚀 ~ set: Jerry
// 🚀 ~ get:
// 🚀 ~ person.name: Jerry
```



### 监听对象上的多个属性

咋一看感觉思路很简单，通过`Object.keys()`拿到对象的所有键，然后遍历劫持就可以，所以就写出如下的代码：

```js
const person = {
  name: 'juns',
  age: 20,
}

Object.keys(person).forEach(key => {
  Object.defineProperty(person, key, {
    enumerable: true,
    configurable: true,
    get() {
      return person[key]
    },
    set(val) {
      console.log('🚀 ~ set:')
      person[key] = val
    },
  })
})

console.log('🚀 ~ person.name:', person.name)
// 报错，堆栈溢出
```

运行一下就会发现报错了，再看下代码就可以得知在访问person身上的属性时，就会触发get方法，返回person[key]，但是访问person[key]也会触发get方法，导致递归调用，最终栈溢出。

所以需要设置一个中转站来解决这个问题：

```js
const person = {
  name: 'juns',
  age: 20,
}

function defineProperty(obj, key, val) {
  Object.defineProperty(obj, key, {
    get() {
      return val
    },
    set(newVal) {
      val = newVal
    },
  })
}

function Observer(obj) {
  Object.keys(obj).forEach(key => {
    defineProperty(obj, key, obj[key])
  })
}

console.log('🚀 ~ person.name:', person.name)
```



### 深度监听一个对象

我们也需要解决对象嵌套对象的这种情况，可以观察到，上面的Observer就是我们想要的监听函数，只需要加一层递归就可以实现了。

```js
const person = {
  name: 'juns',
  age: 20,
}

function defineProperty(obj, key, val) {
  if (typeof val === 'object') Observer(val)
  Object.defineProperty(obj, key, {
    get() {
      return val
    },
    set(newVal) {
      if (typeof newVal === 'object') Observer(key)
      val = newVal
    },
  })
}

function Observer(obj) {
  if (typeof obj !== 'object' || obj === null) {
    return
  }
  Object.keys(obj).forEach(key => {
    defineProperty(obj, key, obj[key])
  })
}

console.log('🚀 ~ person.name:', person.name)
```



### 监听数组

如果监听的对象属性是一个数组呢？如何实现监听

```js
let arr = [1, 2, 3]
const obj = {}

Object.defineProperty(obj, 'arr', {
  get() {
    console.log('get', arr)
    return arr
  },
  set(val) {
    console.log('set', val)
    arr = val
  },
})

console.log(obj.arr) // 输出get arr [1,2,3]  正常
obj.arr = [1, 2, 3, 4] // 输出set [1,2,3,4] 正常
obj.arr.push(3) // set 没有监听到
obj.arr.unshift(1) // set 没有监听到
obj.arr.pop() // set 没有监听到
obj.arr.shift() // set 没有监听到

console.log(obj.arr)
```

可以发现，通过`push`方法给数组增加元素，set方法是监听不到的。

事实上，通过索引访问或者修改数组中已经存在的元素，是可以触发get和set的。但是对于通过push、unshift增加的元素，会增加一个索引，这种情况需要手动初始化，新增加的元素才能被监听到。

在Vue2中，通过重写Array原型上的方法解决了这个问题。下一篇文章中具体研究。



## [Proxy](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

在上面还有一个问题没有解决，就是给一个对象新增属性的时候，也需要手动监听新的属性

> 正是因为这个原因，使用vue给 data 中的数组或对象新增属性时，需要使用 `vm.$set`才能保证新增的属性也是响应式的。
> 可以看到，通过`Object.definePorperty()`进行数据监听是比较麻烦的，需要大量的手动处理。这也是为什么在Vue3.0中尤雨溪转而采用Proxy。



### 基本使用

**Proxy** 对象用于创建一个对象的代理，从而实现基本操作的拦截和自定义（如属性查找、赋值、枚举、函数调用等）。

```js
const p = new Proxy(target, handler)
```

1. target: 要使用 `Proxy` 包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）
2. handler: 一个通常以函数作为属性的对象，各属性中的函数分别定义了在执行各种操作时代理 `p` 的行为。

通过Proxy，我们可以对设置代理的对象上的一些操作进行拦截，外界对这个对象的各种操作，都要先通过这层拦截。一般配合`Reflect`一起使用。

```js
const obj = {
  name: '2323',
  age: 23,
}

const handler = {
  get(target, key, receiver) {
    console.log('get', key)
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    console.log('set', key, value)
    return Reflect.set(target, key, value, receiver)
  },
}

const p = new Proxy(obj, handler)

p.name = '123'
// set name 123
console.log(p.name)
// get name
```

可以看出，`Proxy`代理的是整个对象，而不是对象的某个特定属性，不需要我们通过遍历来逐个进行数据绑定。

> 值得注意的是: 之前使用`Object.defineProperty()`给对象添加一个属性之后，对对象属性的读写操作仍然在对象本身。
>
>  但是使用Proxy，如果想要读写操作生效，我们就要对`Proxy`的实例对象`proxyObj`进行操作。



### 解决`Object.defineProperty()`中遇到的问题

在上文遇到的问题有：

1. 一次只能对一个属性进行监听，需要遍历来对所有属性监听。这个可以解决

2. 在遇到一个对象的属性还是一个对象的情况下，需要递归监听
3. 对于对象的新增属性，需要手动监听
4. 对于数组通过push、unshift方法增加的元素，无法监听

对于2来看一下

```js
const obj = {
  name: '2323',
  age: 23,
  children: {
    name: 'child',
  },
}
const handler = {
  get(target, key, receiver) {
    console.log('get', key)
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    console.log('set', key, value)
    return Reflect.set(target, key, value, receiver)
  },
}

const p = new Proxy(obj, handler)

console.log('🚀 ~ p.children.name:', p.children.name)
// get children
// 🚀 ~ p.children.name: child

p.children.name = 'new child'
// 没有触发log

console.log('🚀 ~ p.children.name:', p.children.name)
// get children
// get children
// 🚀 ~ p.children.name: new child
```

可以看到，访问proxyObj的深层属性时，并不会触发set。所以proxy如果想实现深度监听，也需要实现一个类似上文的`Observer`的递归函数，使用proxy逐个对对象中的每个属性进行拦截，具体的实现逻辑可以参考上文。



第三个问题也解决了

```js
p.children.height = 20

console.log('🚀 ~ p.children.height:', p.children.height)
// get children
// get children
// 🚀 ~ p.children.height: 20
```



对于数组进行一下测试

```js
const arr = ['111']
const handler = {
  get(target, key, receiver) {
    console.log('get', key)
    return Reflect.get(target, key, receiver)
  },
  set(target, key, value, receiver) {
    console.log('set', key, value)
    return Reflect.set(target, key, value, receiver)
  },
}

const p = new Proxy(arr, handler)

console.log('🚀 ~ p:', p)
// 🚀 ~ p: [ '111' ]

console.log('🚀 ~ p[1]:', p[1])
// get 1
// 🚀 ~ p[1]: undefined

p[0] = '000'
// set 0 000
console.log('🚀 ~ p[0]:', p[0])
// get 0
// 🚀 ~ p[0]: 000

p.push('222')
// get push
// get length
// set 1 222
// set length 2
console.log('🚀 ~ p[1]:', p[1])
// get 1
// 🚀 ~ p[1]: 222
```

其中push有俩次get和set和原理有关，也很正常，push会造成其他属性变化，长度+1这样的。

> 另外，重复触发set可能会导致重复派发更新，可以关注下vue3是如何解决这个问题的。



### Proxy支持13种拦截操作

`Objdect.defineProperty()`仅仅支持`getter`和`setter`，详情看文档吧

https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy

CV过来的一些简介：

- get(target, propKey, receiver)：拦截对象属性的读取，比如proxy.foo和proxy['foo']。
- set(target, propKey, value, receiver)：拦截对象属性的设置，比如proxy.foo = v或proxy['foo'] = v，返回一个布尔值。
- has(target, propKey)：拦截propKey in proxy的操作，返回一个布尔值。
- deleteProperty(target, propKey)：拦截delete proxy[propKey]的操作，返回一个布尔值。
- ownKeys(target)：拦截Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而Object.keys()的返回结果仅包括目标对象自身的可遍历属性。
- getOwnPropertyDescriptor(target, propKey)：拦截Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。
- defineProperty(target, propKey, propDesc)：拦截Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。
- preventExtensions(target)：拦截Object.preventExtensions(proxy)，返回一个布尔值。
- getPrototypeOf(target)：拦截Object.getPrototypeOf(proxy)，返回一个对象。
- isExtensible(target)：拦截Object.isExtensible(proxy)，返回一个布尔值。
- setPrototypeOf(target, proto)：拦截Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。
- apply(target, object, args)：拦截 Proxy 实例作为函数调用的操作，比如proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。
- construct(target, args)：拦截 Proxy 实例作为构造函数调用的操作，比如new proxy(...args)。



### Proxy中有关this的问题

虽然Proxy完成了对目标对象的代理，但是它不是透明代理,也就是说：即使handler为空对象（即不做任何代理），他所代理的对象中的this指向也不是该对象，而是proxyObj对象。让我们来看一个例子：

```js
const target = {
  logThis() {
    console.log(this)
    console.log(this === proxyObj)
  },
}

const proxyObj = new Proxy(target, {})

target.logThis() // targetObj false
proxyObj.logThis() // proxyObj true
```

从log可以看出，被代理对象的内部this指向的是proxyObj，这个可能会引起问题，例如：

```js
const _name = new WeakMap()

class Person {
  constructor(name) {
    _name.set(this, name)
  }
  get name() {
    return _name.get(this)
  }
}

const jerry = new Person('jerry')
console.log(jerry.name) // jerry

const p = new Proxy(jerry, {})
console.log(p.name) // undefined
```

在上面的例子中，由于jerry对象的name属性的获取依靠this的指向，而this又指向proxyObj，所以导致了无法正常代理。

这个问题的解决方法是代理这个类本身：

```js
const _name = new WeakMap()

class Person {
  constructor(name) {
    _name.set(this, name)
  }
  get name() {
    return _name.get(this)
  }
}

const ProxyPerson = new Proxy(Person, {})

const p = new ProxyPerson('jerry')
console.log(p.name) // jerry
```



此外有些js内置对象的正确属性的获取也需要正确的this

```js
const target = new Date()
const handler = {}
const proxyObj = new Proxy(target, handler)

proxyObj.getDate()
// TypeError: this is not a Date object.
```

解决方法：手动绑定this

```js
const target = new Date('2024-01-01')
const handler = {
  get(target, prop) {
    if (prop === 'getDate') {
      return target.getDate.bind(target)
    }
    return Reflect.get(target, prop)
  },
}

const proxy = new Proxy(target, handler)
console.log('🚀 ~ proxy.getDate():', proxy.getDate())
// 1
```



### 小结

> CV自js红宝书

代理是 ECMAScript 6 新增的令人兴奋和动态十足的新特性。尽管不支持向后兼容，但它开辟出了一片前所未有的 JavaScript 元编程及抽象的新天地。

从宏观上看，代理是真实 JavaScript 对象的透明抽象层。代理可以定义包含捕获器的处理程序对象，

而这些捕获器可以拦截绝大部分 JavaScript 的基本操作和方法。在这个捕获器处理程序中，可以修改任何基本操作的行为，当然前提是遵从捕获器不变式。

与代理如影随形的反射 API，则封装了一整套与捕获器拦截的操作相对应的方法。可以把反射 API看作一套基本操作，这些操作是绝大部分 JavaScript 对象 API 的基础。

代理的应用场景是不可限量的。开发者使用它可以创建出各种编码模式，比如（但远远不限于）跟踪属性访问、隐藏属性、阻止修改或删除属性、函数参数验证、构造函数参数验证、数据绑定，以及可观察对象。



## 参考文章

[一文搞懂Object.defineProperty和Proxy，Vue3.0为什么采用Proxy？](https://juejin.cn/post/7069397770766909476)
