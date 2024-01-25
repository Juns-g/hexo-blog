---
title: JS数据类型判断
tags:
  - JS
  - 面试
  - 前端
categories: 面试
cover: 'https://w.wallhaven.cc/full/3l/wallhaven-3lw5g9.jpg'
abbrlink: 1583
date: 2024-01-25 15:46:49
---

# JS数据类型判断

> 参考文章：[每天搞透一道JS手写题💪「Day1数据类型判断+手写instanceof」](https://juejin.cn/post/7275551289965084724)



## typeof

可以区分基础类型，但是对于引用数据类型都会返回`object`，对类会返回`function`，对`null`返回`object`

```js
log(typeof 'Ken'); // string
log(typeof 3.14); // number
log(typeof false); // boolean
log(typeof function () {}); // function
log(typeof undefined); // undefined
log(typeof [1, 2, 3, 4]); // object
log(typeof { name: 'Ken', age: 18 }); // object
log(typeof new Date()); // object
log(typeof null); // object
log(typeof Object); // function
log(typeof Number); // function
```



## instanceof

直接检测构造函数的`prototype`属性是否出现在某个实例对象的原型链上。由于基础数据类型、undefined、null没有构造函数，所以返回false

```js
log(123 instanceof Number); // false
log(new Number(123) instanceof Number); // true

log('123' instanceof String); // false
log(new String('123') instanceof String); // true

log([] instanceof Array); // true
log({} instanceof Object); // true
log(function () {} instanceof Function); // true
```



## constructor

`Object`的`constructor`属性返回的是一个引用，指向创建这个实例对象的构造函数。

能判判断基础类型和引用类型，但是不能判断`undefined`和`null`



```js
const o1 = {};
log(o1.constructor === Object); // true

const o2 = new Object();
log(o2.constructor === Object); // true

const a1 = [];
log(a1.constructor === Array); // true

const a2 = new Array();
log(a2.constructor === Array); // true

const n = 3;
log(n.constructor === Number); // true

log(undefined.constructor);
// TypeError: Cannot read properties of undefined (reading 'constructor')
log(null.constructor);
// TypeError: Cannot read properties of null (reading 'constructor')
```



## [Object.prototype.toString.call()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)

> 以这种方式使用 `toString()` 是不可靠的；对象可以通过定义 [`Symbol.toStringTag`](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FSymbol%2FtoStringTag) 属性来更改 `Object.prototype.toString()` 的行为，从而导致意想不到的结果。

不过实际开发我们大多不会去修改他的值，所以也够用了。

每个数据类都重写了这个方法，所以必须用`Object.prototype`上的方法

**为什么最后要加 `call()` 呢？**

因为 `Object.prototype.toString()` 返回的是调用者的类型。不论你 `toString()` 本身的入参写的是什么，`在Object.prototype.toString()` 中，他的调用者永远都是 `Object.prototype`；所以，在不加 `call()` 情况下，我们的出来的结果永远都是 `[object Object]`

这里的 `call()` 方法, 是为了改变 `Object.prototype.toString` 这个函数中的 `this` 指向。让 `Object.prototype.toString` 这个方法的 `this` 指向我们所传入的数据。



## 手写



### [instanceof](https://github.com/Juns-g/js-handwrite/blob/master/src/instanceof.js)

```js
export function myInstanceof(left, right) {
	// 左侧必须是object
	if (Object(left) !== left) return false;
	// 对于右侧参数可以认为只能为函数, 且不能没有Prototype属性
	if (typeof right !== 'function' || !right.prototype) {
		throw new Error('Right-hand side of "instanceof" is not an object');
	}
	let proto = left.__proto__;
	while (proto !== null) {
		if (proto === right.prototype) return true;
		proto = proto.__proto__;
	}
	return false;
}
```



### 精确判断类型

[js-handwrite/blob/master/src/typeOf.js](https://github.com/Juns-g/js-handwrite/blob/master/src/typeOf.js)

```js
// 精准的typeof，使用Object.prototype.toString.call()
export const myTypeOf = (data) => Object.prototype.toString.call(data).slice(8, -1).toLowerCase();
```

