---
title: JS中对象的key的排列顺序
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 48949
date: 2024-02-21 00:31:56
---

# JS 中对象的 key 的排列顺序

面试官：来你看一下这个顺序应该是什么？

```js
const obj = {}
obj['b'] = 100
obj['a'] = 33
obj[2] = 0
obj[1] = 10

console.log(Object.keys(obj))
```

我：🤡 我只知道 obj 的 key 会做自动转换，数字会转换成字符串，然后输出的顺序还是不一定和声明时一样，我猜测是`1 2 a b`。

结果我还是错的，答案是`1 2 b a`，面试官也建议我去了解看看这个到底是啥机制 🥴。

## 实验看看

直接去代码试试看，我们知道对象的键可以为字符串或者 symbol，当使用数字作为 key 时其实会被转换成字符串

### 字符串

```js
const obj = {
  b: 1,
  a: 1,
  d: 1,
  c: 1,
}

console.log(Object.keys(obj))
// b, a, d, c
```

可以看出来，和我们声明的顺序是一致的，所以我们可以猜测，只有字符串的时候，遍历的顺序和我们的声明顺序是一样的，下面看看数字呢？

### 数字

```js
const obj = {
  4: 1,
  1: 1,
  3: 1,
  2: 1,
}

console.log(Object.keys(obj))
// '1', '2', '3', '4'
```

也可以看出，数字被转换成了字符串，而且顺序改变了，变成了升序。在 JS 中，sort 方法有时候也有坑，因为默认情况他并不是按照数字的大小排序的，而是一个什么字典序（待深究），所以我们看看这里是不是也是这样的：

```js
const obj = {
  10: 1,
  11: 1,
  1: 1,
  3: 1,
  21: 2,
  2: 1,
}

console.log(Object.keys(obj))
// '1', '2', '3', '10', '11', '21'
```

好的 🤪，他看上去是直接的升序，和 sort 不一样，还挺好的，下面我们看看 symbol

### symbol

```js
const obj = {
  [Symbol()]: 1,
  [Symbol(2)]: 1,
  [Symbol(1)]: 1,
  [Symbol('b')]: 1,
  [Symbol('a')]: 1,
}

console.log(Object.keys(obj))
// 🈚️输出
```

😡 咋回事，想想 🤔 看？

其实这是 symbol 的特性，他不能被常规迭代，来个特殊方法看看：

```js
const obj = {
  [Symbol()]: 1,
  [Symbol(2)]: 1,
  [Symbol('b')]: 1,
  [Symbol(1)]: 1,
  [Symbol('a')]: 1,
}

console.log(Object.getOwnPropertySymbols(obj))
// [ Symbol(), Symbol(2), Symbol(b), Symbol(1), Symbol(a) ]
```

我们也能看到，这就和我们声明时的顺序一样，那么其实我们还得看看他们 3️⃣ 混合是啥样了：

### 综合

```js
const obj = {
  c: 1,
  a: 1,
  2: 1,
  [Symbol(1)]: 1,
  1: 1,
  [Symbol()]: 1,
  b: 1,
  3: 1,
}

console.log(Object.keys(obj))
// [ '1', '2', '3', 'c', 'a', 'b' ]
console.log(Object.getOwnPropertySymbols(obj))
// [ Symbol(1), Symbol() ]
```

那么也就是说，数字在最前面，并且顺序改变成升序。字符串的 key 的顺序没有改变，但是会整体移动到数字 key 后面，symbol 另算。

### 补漏

其实我们可能还漏了一些情况，比如小数？还有一些其他样式的数字呢。

```js
const obj = {
  c: 1,
  2: 1,
  0.9: 4,
  1e2: 1,
  NaN: 1,
  1: 1,
  a: '1',
}

console.log(Object.keys(obj))
// [('1', '2', '100', 'c', '0.9', 'NaN', 'a')]
```

其中可以发现，0.9 还有 NaN 不对啊，这顺序不太对，所以下面看看规范吧。

## 查文档

先看[Object.keys()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/keys)，发现如下描述：

> Object.keys() 返回一个数组，其元素是字符串，对应于直接在对象上找到的可枚举的字符串键属性名。这与使用 for...in 循环迭代相同，只是 for...in 循环还会枚举原型链中的属性。Object.keys() 返回的数组顺序和与 for...in 循环提供的顺序相同。

也就是说他俩的顺序是一样的，再往下找找，找到[规范](https://262.ecma-international.org/6.0/#sec-ordinary-object-internal-methods-and-internal-slots-ownpropertykeys)：

> 9.1.12 `[[OwnPropertyKeys]]()`
> When the `[[OwnPropertyKeys]]` internal method of O is called the following steps are taken:
>
> - Let keys be a new empty List.
> - For each own property key P of O that is an integer index, in ascending numeric index order
>   - Add P as the last element of keys.
> - For each own property key P of O that is a String but is not an integer index, in property creation order
>   - Add P as the last element of keys.
> - For each own property key P of O that is a Symbol, in property creation order
>   - Add P as the last element of keys.
> - Return keys.

翻译一下就是：

- 新建一个空数组 keys
- 遍历 O 中的所有`整数`类型的 key，`按数字索引升序排列`添加到数组中
- 遍历 O 中的所有`字符串类型(包括负数、浮点数)`的 key，按照`创建顺序`添加到数组中
- 遍历 O 中的所有`Symbol`类型的 key，按照`创建顺序`添加到数组中
- 返回 keys

所以我们就可以知道，数字 key 会被转换成字符串，然后按照升序排列，字符串 key 会按照创建顺序排列，symbol 也是按照创建顺序排列。

## 来个题

来个题实践看看，我们有没有理解：

```js
const obj = {}

obj[-1] = ''
obj[1] = ''
obj[1.1] = ''
obj['2'] = ''
obj['c'] = ''
obj['b'] = ''
obj['a'] = ''
obj[Symbol(1)] = ''
obj[Symbol('a')] = ''
obj[Symbol('b')] = ''
obj['d'] = ''

console.log(Object.keys(obj))
```

<details>
  <summary>答案👈🏻</summary>
  <pre>['1', '2', '-1', '1.1', 'c', 'b', 'a', 'd']
</pre>
</details>

😼 大功告成，对的

## 参考文章

- [一行 Object.keys() 引发的血案](https://juejin.cn/post/7041049741458669576)
