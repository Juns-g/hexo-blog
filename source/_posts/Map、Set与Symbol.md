---
title: Map、Set与Symbol
tags:
  - JS
  - 学习
keywords: 学习 前端 TS
categories: 前端
cover: >-
  https://static001.geekbang.org/resource/image/bd/8a/bd1d00a9c1b4141d3965cd0b9849e78a.jpg
abbrlink: 30706
date: 2024-01-30 14:21:34
---

# Map、Set与Symbol



## 前言

这几个数据结构是ES6新增的，还记得当时第一次面试被问到“你对这几个了解多少？有使用过吗？”时，啥也不会特尴尬🫢，所以现在详细了解并整理一下这几个数据结构。



## [Map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map)

Map是什么？让我们来看一下MDN文档的介绍：

> **`Map`** 对象保存键值对，并且能够记住键的原始插入顺序。任何值（对象或者[原始值](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)）都可以作为键或值。

可以看到其中几个关键点：保存键值对、记住键的原始插入顺序、任何值都能作为键或值

这些就是他的特点，相较于object不同的地方，不过其实他和object也有很多共同之处，而且很多时候用哪个都可以。

### 基本使用

```js
// 创建一个空映射
const map1 = new Map()

// 使用嵌套数组初始化映射
const map2 = new Map([
  ['key1', 'val1'],
  ['key2', 'val2'],
])
```

初始化之后可以通过`set()`来添加键值对，此外还可以通过`get()`和`has()`来查询，可以用`size`属性来获取数量，还可以用`delete()`和`clear()`来删除值

```js
const m = new Map()

m.set('first', 1)
console.log(m.get('first')) // 1

m.set('second', 2)
console.log(m.has('second')) // true
console.log(m.size) // 2

m.delete('first')
console.log(m.size) // 1

m.clear()
console.log(m.size) // 0
```

由于`set()`方法返回的是映射实例，所以也可以初始化的时候连起来操作

```js
const m = new Map().set('key1', 'val1')
m.set('key2', 2)
```

### 键的唯一性

在`Map`中，一个键只能出现一次，其中键的比较基于[SameValueZero](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness#%E9%9B%B6%E5%80%BC%E7%9B%B8%E7%AD%89) (零值相等)算法，基本上是相当于使用严格对象相等的标准来检测。

> SameValueZero 是ES规范内部定义，语言中不能使用，JS提供的判断只有`==`、`===`、`Object.is()`。

```js
const m = new Map();

const functionKey = function () {};
const fKey2 = function () {};

m.set(functionKey, 'functionVal');

console.log(m.get(functionKey)); // functionVal
console.log(m.get(fKey2)); // undefined
```

与严格相等一样，在映射中用作键和值的对象，在自己的内容或属性被修改是仍然保持不变

```js
const m = new Map();

const objKey = {};
const objVal = {};

m.set(objKey, objVal);
console.log(m.get(objKey)); // {}
objKey.key = 'key';
objVal.val = 'val';
console.log(m.get(objKey)); // { val: 'val' }
```

不过这个相等算法，也可能会导致意想不到的冲突：

```js
const m = new Map();

const a = NaN,
	b = NaN,
	c = +0,
	d = -0;

console.log(a === b); // false
console.log(c === d); // true

m.set(a, 'NaN value');
m.set(c, '+0 value');

console.log(m.get(b)); // NaN value
console.log(m.get(d)); // +0 value
```

### 顺序与迭代

`Map`与`Object`的一个主要差异就是，`Map`会维护插入顺序，因此可以根据插入顺序来执行迭代操作。它提供了一个迭代器，可以通过`entries()`或者`Symbol.iterator`属性来获取

```js
const m = new Map([
	['key1', 1],
	['key2', 2],
]);

console.log(m.entries === m[Symbol.iterator]); // true
const iterator = m.entries();

for (const [key, val] of iterator) {
	console.log(key, val);
	// key1 1
	// key2 2
}
```

因为`entries()`是默认迭代器，所以可以直接使用拓展操作把映射转成数组

```js
const m = new Map([
	['key1', 1],
	['key2', 2],
]);

console.log([...m]); // [ [ 'key1', 1 ], [ 'key2', 2 ] ]
```

也可以使用映射的[`forEach`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Map/forEach)来迭代

```js
const m = new Map([
	['key1', 1],
	['key2', 2],
]);

m.forEach((value, key) => {
	console.log(key, value);
	// key1 1
	// key2 2
});
```

通过`keys()`可以获取映射的键迭代器，`values()`可以获得值的迭代器。

### 对比Object

他们俩比较相似，而且用法也差不多，日常开发中用哪个基本相差不大，不过还是有一些优劣的。

> 下面的对比摘录自《JavaScript高级程序设计(第4版)》

- 内存占用：Map占用更少
- 插入性能：Map性能更佳
- 查找速度：Object更好
- 删除性能：Map更好

> 关键词：迭代器、SameValueZero算法、性能、Object

## [WeakMap](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)

看名字就知道他和`Map`关系很大，看看MDN介绍先：

> **`WeakMap`** 是一种**键值对**的集合，其中的键必须是对象或[非全局注册的符号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol#全局共享的_symbol)，且值可以是任意的 [JavaScript 类型](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Data_structures)，并且不会创建对它的键的强引用。换句话说，一个对象作为 `WeakMap` 的键存在，不会阻止该对象被垃圾回收。一旦一个对象作为键被回收，那么在 `WeakMap` 中相应的值便成为了进行垃圾回收的候选对象，只要它们没有其他的引用存在。唯一可以作为 `WeakMap` 的键的类型是[非全局注册的符号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol#全局共享的_symbol)，因为非全局注册的符号是保证唯一的，并且不能被重新创建。

简单看就是，基本功能和`Map`一样，但是他的键必须是对象，原始数据类型作为键会报错

```js
const sy = Symbol();
const wm = new WeakMap([
	[sy, 1],
	[2, 2],
]);
// TypeError: Invalid value used as weak map key
console.log('🚀 ~ wm.get(sy):', wm.get(sy));
```

### 弱键

`WeakMap`中的`weak`表是键是弱弱的拿着，意思就是这些键不是正式引用，不会阻止垃圾回收。

看一个例子：

```js
const wm = new WeakMap()
wm.set({}, 'val')
```

`set()`初始化了一个新的空对象并作为一个字符串的键。因为没有指向这个对象的其他引用，所以这行代码执行之后，这个对象键会被当作垃圾回收掉。然后这个键值对就从弱映射中消失了，变成了一个空映射。

另一个例子：

```js
const wm = new WeakMap();
const container = {
	key: {},
};

wm.set(container.key, 'value');
console.log(wm.get(container.key)); // value

container.key = null;
console.log(wm.get(container.key)); // undefined
```

在这个例子里面，`container`维护了一个对弱映射键的引用，因此这个对象键不会成为垃圾回收的目标，但是当`container.key = null`调用时，会清理掉。

同时也因为弱引用，所以他不可迭代，同时也没有`clear()`方法

### 用处

因为他不会妨碍垃圾回收，所以说他非常适合保存关联元数据

```js
const m = new Map()
const btn = document.querySelector('#login')
m.set(btn, 'val')
```

假设这个代码执行后，那个按钮从DOM树中删除掉了。但是Map还保存着引用，所以对应的DOM节点还会逗留在内存中。

如果是使用的弱映射，当节点被删除后，垃圾回收程序就可以立即释放内存。

```js
const wm = new WeakMap()
const btn = document.querySelector('#login')
wm.set(btn, 'val')
```

> 关键词：不可迭代、垃圾回收

## [Set](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)

我一直理解的set就挺像数组的，但是里面都是唯一值，看看MDN咋说：

> **`Set`** 对象允许你存储任何类型（无论是[原始值](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)还是对象引用）的唯一值。
>
> `Set` 对象是值的合集（collection）。集合（set）中的元素**只会出现一次**，即集合中的元素是唯一的。你可以按照插入顺序迭代集合中的元素。*插入顺序*对应于 [`add()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/add) 方法成功将每一个元素插入到集合中（即，调用 `add()` 方法时集合中不存在相同的元素）的顺序。

他的api也和Map比较类似，而且他的值相等算法也是[SameValueZero(零值相等)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness#零值相等)算法

### 基本使用

```js
const s = new Set(['val1', 'val2'])
console.log(s.size) // 2
```

初始化后可以使用`add()`添加值，使用`has()`查询，使用`size`获取数量，以及可以用`delete()`和`clear()`删除元素。

### 顺序和迭代

与`Map`类似，`Set`也提供了迭代器，可以通过`values()`或`keys()`来获取，他俩是一样的，`Symbol.iterator`属性引用的是`values()`

```js
const s = new Set([1, '2']);

console.log(s.values === s[Symbol.iterator]); // true
console.log(s.keys === s[Symbol.iterator]); // true
console.log(s.values === s.keys); // true

for (const val of s.values()) {
	console.log(val); // 1 2
}
```

如果是用`entries()`或者`forEach`则是俩个重复的值：

```js
const s = new Set([1, '2']);

for (const pairs of s.entries()) {
	console.log(pairs);
	// [ 1, 1 ]
	// [ '2', '2' ]
}

s.forEach((val, dupVal) => {
	console.log(val, dupVal);
	// 1 1
	// 2 2
});
```

## 用处

一般用的多的就是数组去重吧

```js
const unique = arr => [...new Set(arr)]
```

## WeakSet

> **`WeakSet`** 是可被垃圾回收的值的集合，包括对象和[非全局注册的符号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol#全局共享的_symbol)。`WeakSet` 中的值只能出现一次。它在 `WeakSet` 的集合中是唯一的。
>
> 它和 [`Set`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set) 对象的主要区别有：
>
> - `WeakSet` **只能是对象和符号**的集合，它不能像 [`Set`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set) 那样包含任何类型的任意值。
> - `WeakSet` 持*弱引用*：`WeakSet` 中对象的引用为*弱*引用。如果没有其他的对 `WeakSet` 中对象的引用存在，那么这些对象会被垃圾回收。

`WeakSet`就和`WeakMap`类似

### 作用

递归调用自身的函数需要一种通过跟踪哪些对象已被处理，来应对循环数据结构的方法。

```js	
// 对传入的 subject 对象内部存储的所有内容执行回调
function execRecursively(fn, subject, _refs = new WeakSet()) {
	// 避免无限递归
	if (_refs.has(subject)) {
		return;
	}
	fn(subject);
	if (typeof subject === 'object') {
		_refs.add(subject);
		for (const key in subject) {
			execRecursively(fn, subject[key], _refs);
		}
	}
}

const foo = {
	foo: 'Foo',
	bar: {
		bar: 'Bar',
	},
};

foo.bar.baz = foo; // 循环引用！
function fn(obj) {
	return console.log(obj);
}
execRecursively(fn, foo);
```

## [Symbol](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol)

Symbol是一种基本数据类型，每一个从`Symbol()`获取的symbol值都是唯一的，一个symbol值可以作为对象属性的标识符，这是该数据类型仅有的目的。

> **symbol** 是一种基本数据类型（[primitive data type](https://developer.mozilla.org/zh-CN/docs/Glossary/Primitive)）。`Symbol()` 函数会返回 **symbol** 类型的值，该类型具有静态属性和静态方法。它的静态属性会暴露几个内建的成员对象；它的静态方法会暴露全局的 symbol 注册，且类似于内建对象类，但作为构造函数来说它并不完整，因为它不支持语法："`new Symbol()`"。

```js
Symbol('11') === Symbol('11') // false
```

他不会被常规的枚举方法包含，如`for...in`、`Object.keys()`

```js
const age = Symbol('age');
const obj = {
	name: 'aaa',
	[age]: 18,
};

console.log(obj[age]); // 18
console.log(obj.age); // undefined

for (const key in obj) {
	console.log(key); // name
}

console.log(Object.keys(obj)); // [ 'name' ]
console.log(Object.getOwnPropertyNames(obj)); // [ 'name' ]
```









