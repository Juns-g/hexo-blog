---
title: 了解一下Array.flatMap()
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 10843
date: 2024-02-29 15:38:13
---

# 了解一下 Array.flatMap()

> MDN 地址：[Array.prototype.flatMap()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/flatMap)

这个 API 我之前都没听过，所以看到他还挺好奇的，简单了解一下，然后看看他能做啥啥 🤓。

## 介绍

MDN 是这么介绍的：

> flatMap() 方法对数组中的每个元素应用给定的回调函数，然后将结果展开一级，返回一个新数组。它等价于在调用 map() 方法后再调用深度为 1 的 flat() 方法（arr.map(...args).flat()），但比分别调用这两个方法稍微更高效一些。

😯 我再看一下例子，感觉理解的差不多了。

```js
const arr1 = [1, 2, 1]

const result = arr1.flatMap(num => (num === 2 ? [2, 2] : 1))

console.log(result) // [ 1, 2, 2, 1 ]
```

他就是先 map() 然后 flat()，这样就可以实现改变长度的 map🤪。

## 基本语法

```js
flatMap(cb)
flatMap(cb, thisArg)
```

第一个就是回调函数，参数和 map 一样，不多介绍了。第二个是回调函数的 this 值，可选，也不用介绍了。

### 对比 map

他就是相当于 map 后再 flat(1)，扁平一层

```js
const arr = [1, 2, 3]

arr.map(x => [x * 2])
// [[2], [4], [6]]

arr.flatMap(x => [x * 2])
// [2, 4, 6]

// 只扁平一层
arr.flatMap(x => [[x * 2]])
// [[2], [4], [6]]
```

这个特性也就让下面这个情况更简洁：

```js
const arr1 = ["it's Sunny in", '', 'California']

arr1.map(x => x.split(' '))
// [["it's","Sunny","in"],[""],["California"]]

arr1.flatMap(x => x.split(' '))
// ["it's","Sunny","in", "", "California"]
```

所以说，flatMap 可以实现一个一对多的映射，比如：

```js
// 假设我们想要删除所有负数，并将奇数拆分成偶数和 1
const a = [5, 4, -3, 20, 17, -33, -4, 18]
//         |\  \  x   |  | \   x   x   |
//        [4,1, 4,   20, 16, 1,       18]

const result = a.flatMap(n => {
  if (n < 0) return []
  return n % 2 === 0 ? [n] : [n - 1, 1]
})
console.log(result) // [4, 1, 4, 20, 16, 1, 18]
```

## 用法

比如这样的情况：[0, 3, 6] -> [6, 12]，需要去除 0，然后乘以 2

map 需要这样：

```js
const arr = [0, 3, 6]

const arr1 = arr.filter(n => n !== 0).map(n => n * 2)
```

而 flatMap 可以简单一些：

```js
const arr = [0, 3, 6]

const arr2 = arr.flatMap(x => (x === 0 ? [] : [x * 2]))
```

## 参考文章

[array.flatMap：一个灵活好用的 Map 方法
](https://juejin.cn/post/7055950245724684318)
