---
title: JS异步的俩道题
tags:
  - JS
  - 面试
  - 前端
categories: 面试
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
abbrlink: 50424
date: 2024-09-11 14:32:27
---

# JS 异步的俩道题

> 京东面试遇到的一个事件循环输出题，其中有俩点新知识，以及还有一个之前开发遇到的

## Promise.then 传非函数

```js
Promise.resolve(1)
  .then(2)
  .then(3)
  .catch(4)
  .then(res => console.log(res))
```

问 log 什么，当时就没有答对，说的 3，因为对于 then，一般都是传入函数，成功回调和失败回调，没遇到过直接传数字这个情况，以及中间还有个 catch，这个属于是意料之外了。

实际输出的是 1，至于为什么，查询 MDN 文档可以知道

> Promise.prototype.then()
> 将一个兑现处理器和拒绝处理器附加到 Promise 上，并返回一个新的 Promise，解决为调用处理器得到的返回值，或者如果 Promise 没有被处理（即相关处理器 onFulfilled 或 onRejected 不是函数），则以原始敲定值解决。

理解就是，如果 then 没有传给他函数，就相当于没有他，用上次的结果。因为 catch 也是 then 的变体，所以最后这个代码相当于

```js
Promise.resolve(1).then(res => console.log(res))
```

那自然是输出 1 了

## Promise 内执行多个 resolve 会怎么样

这个是实际开发遇到的了，分条件 resolve 不同的，那么不提前 return，下面的会怎么执行呢。

```js
const { log } = console

const flag = true
new Promise((resolve, reject) => {
  if (flag) {
    resolve(111)
  }
  log(222)
  resolve(222)
  log(333)
  resolve(333)
}).then(res => log(res))
```

这个倒是 promise 的基础就能推出来的，promise 的状态一旦确定后就不会再改变，所以在 if 中已经把状态改成了完成，后面的 resolve 就不会生效了

然后因为 resolve，是吧 then 里面的函数放到了微任务队列中，下面的还是同步代码，所以 log 是 `222,333,111`

## Promise 不执行 resolve/reject 会怎么样

这个就挺简单的了，就一直处于 pending 等待中了，后面的也不会被执行

```js
const { log } = console
const async1 = async () => {
  log('async1')
  await new Promise(resolve => {
    log('promise1')
  })
  log('async1 end')
  return 'async1 success'
}

async1().then(res => log(res))
```

只会输出`async1, promise1`，因为 promise 没有状态流转
