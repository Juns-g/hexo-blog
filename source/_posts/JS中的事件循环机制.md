---
title: JS中的事件循环机制
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 42193
date: 2024-3-1 12:55:12
---

# JS 中的事件循环机制

> 这个网站可以可视化运行 https://www.jsv9000.app

这个面试经常问诶，但是我好几次回答的都是不尽如人意，这次来好好搞懂一下看看 😡。

## 单线程的 JS

首先我们需要知道 JS 是单线程的，为啥 😧？

因为一开始设计之初，他只是一个浏览器的脚本语言，主要就是操控 DOM 和与用户互动，如果他是多线程的话，那么就会出现很多问题，比如多个线程同时操作一个 DOM，谁知道会发生啥。

当然 JS 也可以多线程，下面引用[阮一峰博客](https://www.ruanyifeng.com/blog/2014/10/event-loop.html)的话：

> 为了利用多核 CPU 的计算能力，HTML5 提出 Web Worker 标准，允许 JavaScript 脚本创建多个线程，但是子线程完全受主线程控制，且不得操作 DOM。所以，这个新标准并没有改变 JavaScript 单线程的本质。

## 任务队列

那这样我们可以认为，所有的任务都放到了一个队列里面，然后一个一个的执行。但是这样应该还会有问题，比如有的任务计算量很大，有的任务占用时间很长，这样就会导致后面的任务无法执行，这样就会导致页面卡死(因为浏览器的渲染线程和 JS 线程是互斥的)。

如果是前者，CPU 忙不过来，倒也是合理的，直接等待运行结束就行。但是如果是后者，CPU 空闲，但是因为 IO 设备慢，典型的就是从网络接口后去数据，这个时候就没必要等待他，可以先去处理别的任务，等他好了再回来处理。

所以说，这样就可以把任务分为两类，同步(sync)任务，和异步(async)任务。同步任务指的是，在主线程上排队执行的任务，必须得按顺序执行的了。异步任务指的是，不进入主线程，而是进入任务队列(task queue)的任务，只有任务队列通知主线程，某个异步任务可以执行了，这个任务才会进入到主线程中中执行。

下面引用阮一峰的博客：

> 具体来说，异步执行的运行机制如下。（同步执行也是如此，因为它可以被视为没有异步任务的异步执行。）
>
> （1）所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）。
> （2）主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件。
> （3）一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行。
> （4）主线程不断重复上面的第三步。

### 宏任务与微任务

在任务队列中，又有俩种任务，宏任务和微任务。

宏任务（macro-task）是一组需要在主线程上执行的任务操作，它们被添加到宏任务队列中。常见的宏任务包括整体的 script 代码块、setTimeout、setInterval、I/O 操作、UI 渲染等。宏任务的执行时机是在当前执行栈为空时，主线程会从宏任务队列中选择一个任务执行，并在执行完该任务后，再次检查是否有其他的宏任务需要执行。

微任务（micro-task）是一组需要在当前任务执行结束后立即执行的任务操作，它们被添加到微任务队列中。常见的微任务包括 Promise 的回调函数（.then()、.catch()、.finally()）、MutationObserver、process.nextTick 等。微任务的执行时机是在当前宏任务执行结束后、下一个宏任务开始之前。也就是说，当主线程执行完当前宏任务后，会立即检查微任务队列，并按照先进先出的顺序执行所有微任务，直到微任务队列为空，然后再执行下一个宏任务。

## 事件与回调函数

上文提到，任务队列中放置的是事件，当 IO 设备完成一项任务，就会在任务队列中添加一个事件。除此之外，用户也可以产生一些事件，只要指定过回调函数，这些事件发生时就会进入任务队列，等待主线程的读取。

> 所谓"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当主线程开始执行异步任务，就是执行对应的回调函数。

## 事件循环

主线程从任务队列中读取事件的过程是循环不断的，所以说这整个运行机制又被称为事件循环(Event Loop)。

简要描述一下这个过程就是，首先执行当前代码这个宏任务，然后清空这个宏任务的微任务队列，再从宏任务队列中取出来下一个宏任务放入执行栈中执行，然后重复 🔁 这个过程。

## 定时器 宏任务

任务队列中除了异步任务，还可以放置定时事件，一般是 setTimeout() 和 setInterval()，他们的内部运行机制是一样的，所以只讨论 setTimeout()就可以。

```js
const { log } = console

log(1)
setTimeout(() => {
  log(2)
}, 100)
log(3)
```

输出很明显，132，因为定时器把 2 延迟了。

```js
setTimeout(() => {
  log(1)
}, 0)
log(2)
```

那无延时的定时器呢？答案是 21，这表示当前的执行栈清空后，立刻执行指定的回调函数。

所以说，setTimeout(fn, 0) 的含义就是，指定这个任务在主线程最早可得的空闲时间执行。

> HTML5 标准规定了 setTimeout()的第二个参数的最小值（最短间隔），不得低于 4 毫秒，如果低于这个值，就会自动增加。在此之前，老版本的浏览器都将最短间隔设为 10 毫秒。另外，对于那些 DOM 的变动（尤其是涉及页面重新渲染的部分），通常不会立即执行，而是每 16 毫秒执行一次。这时使用 requestAnimationFrame()的效果要好于 setTimeout()。
>
> 需要注意的是，setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在 setTimeout()指定的时间执行。

> 注意： setTimeOut 并不是直接的把你的回调函数放进上述的异步队列中去，而是在定时器的时间到了之后，把回调函数放到执行异步队列中去。如果此时这个队列已经有很多任务了，那就排在他们的后面。这也就解释了为什么 setTimeOut 为什么不能精准的执行的问题了。setTimeOut 执行需要满足两个条件：
>
> 主进程必须是空闲的状态，如果到时间了，主进程不空闲也不会执行你的回调函数。这个回调函数需要等到插入异步队列时前面的异步函数都执行完了，才会执行

来俩个例子

```js
console.time('time')
setTimeout(() => {
  console.timeEnd('time')
}, 10)
// time: 10.6201171875 ms
```

这样就是相对比较准时的

```js
console.time('time')
let i = 0
for (let j = 1; j < 100000; j++) i += j
setTimeout(() => {
  console.timeEnd('time')
}, 10)
// time: 14.416015625 ms
```

这里就是因为上面的循环消耗了较多的时间，所以在定时器时间到了之后并没有立刻执行回调函数，而是等待耗时任务完结，才去执行回调函数。

总结说就是，定时器的回调函数执行时间间隔 >=10ms。以及，创建定时器时，相应的任务会立即被添加到宏任务队列中，但是任务不会立即执行，而是会在设定的延迟时间之后，被取出并放入执行栈中执行。

### 顺序题目

```js
const { log } = console

log(1)

setTimeout(() => {
  log(2)
}, 50)

setTimeout(() => {
  log(3)
}) // 不填 delay 则为默认值0

setTimeout(() => {
  log(4)
}, 0)
```

我的答案是 1342，她的执行顺序还是很好推测的，首先 1，然后就是剩下的放入宏任务队列中，按照定时器的时间拿出来，所以就是 342

```js
const { log } = console

setTimeout(() => {
  log(1)
}, 100)
setTimeout(() => {
  log(2)
}, 50)
setTimeout(() => {
  log(3)
}, 0)
setTimeout(() => {
  log(4)
}, 0)
log(5)
```

同理，53421

## Promise 微任务

当 Promise 的状态变为已兑现或已拒绝时，会生成微任务。微任务是一组需要在当前任务执行结束后立即执行的任务操作。Promise 的回调函数`.then()`、`.catch()`、`.finally()`就是微任务。

所以说，new Promise(fn)里面的 fn 是同步执行的。

### 顺序题目

```js
const { log } = console

Promise.resolve().then(() => {
  log(1)
})

log(2)

new Promise((resolve, reject) => {
  log(3)
  resolve()
})
  .then(() => {
    log(4)
  })
  .then(() => {
    log(5)
  })

Promise.reject()
  .then(() => {
    log(7)
  })
  .catch(() => {
    log(8)
  })
  .finally(() => {
    log(9)
  })
```

这个就有点长了。首先执行当前的同步代码，23，然后按顺序执行微任务，14859。

上面的分析是有错误的，实际答案是 2314589，原因是：
先遇到 1，把它放到微任务队列中，然后输出 2，遇到 3 输出，然后把 45 放到微任务队列中，后面就是 89。

## await 微任务

await 是异步操作符，它可以等待一个 Promise 对象的状态发生变化，并返回 Promise 的结果。await 后面的代码会被放到微任务队列中。

### 顺序题目

```js
const { log } = console

async function async1() {
  log(1)
  await async2()
  log(2)
}

async function async2() {
  log(3)
  return new Promise((resolve, reject) => {
    resolve()
    log(4)
  })
}

async1()
```

1342

1. 首先，执行 async1() 函数。
2. 在 async1() 函数内部，遇到 log(1)，打印数字 1。
3. 然后，遇到 await async2()，它会暂停 async1() 函数的执行，并等待 async2() 函数的 Promise 对象解决。
4. 执行 async2() 函数。
5. 在 async2() 函数内部，遇到 log(3)，打印数字 3。
6. 接着，创建一个新的 Promise 对象，并立即执行 resolve() 方法，表示 Promise 成功解决。
7. 在执行 resolve() 后，遇到 log(4)，打印数字 4。
8. async2() 函数执行完毕，返回解决的 Promise 对象。
9. 回到 async1() 函数，继续执行。
10. 由于 async2() 函数的 Promise 对象已经解决，await async2() 表达式会恢复执行，并返回解决的值（这里是一个空值）。
11. 接着，遇到 log(2)，打印数字 2。
12. 执行完 async1() 函数，程序结束。

## 综合题目

```js
const { log } = console

async function async1() {
  log('async1 start')
  await async2()
  log('async1 end')
}

async function async2() {
  log('async2 start')
  return new Promise((resolve, reject) => {
    resolve()
    log('async2 promise')
  })
}

log('script start')

setTimeout(function () {
  log('setTimeout')
}, 0)

async1()

new Promise(function (resolve) {
  log('promise1')
  resolve()
})
  .then(function () {
    log('promise2')
  })
  .then(function () {
    log('promise3')
  })

log('script end')

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

其中 async 的部分可以改成如下的 promise 的话就是
关键点可以看这个文章：[我终于搞懂了 async/await、promise 和 setTimeout 的执行顺序](https://juejin.cn/post/7171002835016892453)
// promise2
// promise3
// async1 end

```js
function async1() {
  log('async1 start')
  return async2().then(() => {
    log('async1 end')
  })
}

function async2() {
  log('async2 start')
  return new Promise((resolve, reject) => {
    resolve()
    log('async2 promise')
  })
}
```

## 参考文章

- [在？聊聊浏览器事件循环机制](https://juejin.cn/post/7257044686815313977)
