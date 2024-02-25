---
title: JS中的事件循环机制
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 42193
date: 2024-02-25 19:55:12
---

# JS 中的事件循环机制

> TODO

## 题目

```js
async function async1() {
  console.log('async1 start')
  await async2()
  console.log('async1 end')
}
async function async2() {
  console.log('async2 start')
  return new Promise((resolve, reject) => {
    resolve()
    console.log('async2 promise')
  })
}
console.log('script start')
setTimeout(function () {
  console.log('setTimeout')
}, 0)
async1()

new Promise(function (resolve) {
  console.log('promise1')
  resolve()
})
  .then(function () {
    console.log('promise2')
  })
  .then(function () {
    console.log('promise3')
  })
console.log('script end')

// script start
// async1 start
// async2 start
// async2 promise
// promise1
// script end
// promise2
// promise3
// async1 end
// setTimeout
```
