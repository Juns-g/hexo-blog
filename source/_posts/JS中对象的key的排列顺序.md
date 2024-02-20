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
