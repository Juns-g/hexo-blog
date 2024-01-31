---
title: 虚拟DOM
tags:
  - 学习
  - 前端
categories: 前端
cover: >-
  https://dogefs.s3.ladydaily.com/~/source/wallhaven/full/5g/wallhaven-5gypg8.png?w=2560&h=1440&fmt=webp
abbrlink: 5749
date: 2024-01-31 19:23:20
---

# 虚拟 DOM

## 前言

在上一篇文章[回流与重绘](https://juns-g.github.io/posts/8742)里面我们就提到了，操作 DOM 元素是代价挺高的，也给出了一些优化方案，不过总体来说，直接操作 DOM 其实也是比较麻烦的，而且也有较高的心智负担，这篇文章就来介绍一下虚拟 DOM。

> 关键词：虚拟DOM、diff算法、跨平台、回流、重绘

## 什么是虚拟 DOM

先来说说为什么出现吧，在之前，基本都是使用`jQuery`手动操作 DOM，代码中有很大一部分都是在操作 DOM，随着应用的复杂化，这变得越来越烦琐，而且也有很大的性能开销。

所以说我们需要有一种方式来优化他，根据上一篇文章的优化方案，我们可以很容易的想到，在 js 里面模拟 DOM，然后操作完毕后，一次性渲染 DOM，这样可以减少很多重排次数。

那么把这个操作规范一下，我们只需要负责改变数据/状态，修改 DOM 的行为交给一个东西来做，可以称之为虚拟 DOM。

虚拟 DOM 可以简单理解为用 js 模拟 DOM 结构，然后在 js 中对比，之后再去渲染。

下面这个 DOM 就可以映射成一个虚拟 DOM

```html
<ul id="list">
  <li class="item">item1</li>
</ul>
```

虚拟 DOM：

```js
const listVDom = {
  tag: 'ul',
  attrs: {
    id: 'list',
  },
  children: [
    {
      tag: 'li',
      attrs: {
        className: 'item',
      },
      children: ['item1'],
    },
  ],
}
```

## 相较于真实 DOM 的优点

有一部分人的观点认为，虚拟 DOM 比真实 DOM 快，不过这个肯定是分情况的，无虚拟 DOM 的[Svelte](https://www.svelte.cn/)和[Solid](https://www.solidjs.com/)性能都不差。只能说是，比如在大量数据渲染，同时只改变一小部分数据的时候，虚拟 DOM 更具有优势。

实际上对于虚拟 DOM，下面才算是他更加优秀的地方：

- 打开了函数式的 UI 编程大门，`UI = f(data)`
- 使用 js 对象来描述 DOM，可以把这个对象渲染到除了浏览器 DOM 之外的地方，可以实现跨平台开发

## 简易实现

下面分别用 jQuery 和虚拟 DOM 来写一个例子

jQuery

```html
<div id="container"></div>
<button id="btn-change">改变</button>

<script src="https://cdn.bootcss.com/jquery/3.2.0/jquery.js"></script>

<script>
  const data = [
    {
      name: '张三',
      age: '20',
      address: '北京',
    },
    {
      name: '李四',
      age: '21',
      address: '武汉',
    },
  ]
  //渲染函数
  function render(data) {
    const $container = $('#container')
    $container.html('')
    const $table = $('<table>')
    // 重排
    $table.append($('<tr><td>name</td><td>age</td><td>address</td></tr>'))
    data.forEach(item => {
      //每次进入都重排
      $table.append(
        $(
          `<tr><td>${item.name}</td><td>${item.age}</td><td>${item.address}</td></tr>`
        )
      )
    })
    $container.append($table)
  }
  render(data)

  $('#btn-change').click(function () {
    data[1].age = 30
    data[0].address = '深圳'
    render(data)
  })
</script>
```

可以轻松看出来，每次改动 data 之后，table 标签都会重新创建渲染，这会很消耗浏览器的性能

下面使用[snabbdom](https://github.com/snabbdom/snabbdom)来实现虚拟 DOM 版本

这是一个简易的实现虚拟 DOM 的库，有俩个核心 api：h 函数和 patch 函数，h 函数用于生成 vdom，patch 用于比对 vdom 以及挂载到真实 DOM 上。

```html
<div id="container"></div>
<button id="btn-change">改变</button>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-class.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-props.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-style.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/snabbdom-eventlisteners.min.js"></script>
<script src="https://cdn.bootcss.com/snabbdom/0.7.3/h.js"></script>
<script>
  let sn = window.snabbdom

  // 定义patch
  let patch = sn.init([
    snabbdom_class,
    snabbdom_props,
    snabbdom_style,
    snabbdom_eventlisteners,
  ])

  //定义h
  let h = sn.h

  const data = [
    { name: '姓名', age: '年龄', address: '地址' },
    { name: '张三', age: '20', address: '北京' },
    { name: '李四', age: '21', address: '武汉' },
  ]

  let container = document.getElementById('container')
  let vnode = null
  const render = data => {
    let newVnode = h(
      'table',
      {},
      data.map(item => {
        let tds = []
        for (let key in item) {
          if (item.hasOwnProperty(key)) {
            tds.push(h('td', {}, item[key] + ''))
          }
        }
        return h('tr', {}, tds)
      })
    )

    vnode ? patch(vnode, newVnode) : patch(container, newVnode)

    vnode = newVnode
  }

  render(data)

  let btn = document.getElementById('btn-change')
  btn.addEventListener('click', function () {
    data[1].age = 30
    data[1].address = '深圳'
    render(data)
  })
</script>
```

可以发现，只有数据改变的地方 DOM 才重新渲染了

## diff 算法

在上面也提到过了，新旧虚拟 DOM 之间需要比对，然后才渲染成真实 DOM，而不是直接整个的替换，这其中就涉及到了 diff 算法。

**什么是 diff 算法？**

在其他地方我们也都听到过 diff，比如文本 diff，`git diff`，意思就是对比算法

**为什么需要 diff 算法？**

在上面也说了，DOM 操作十分消耗性能，所以我们需要尽量减少 DOM 操作。所以说需要找出必须更行的 DOM 节点来选择性更新，其他的不更新，这其中的过程就需要应用到 diff 算法。

**简易实现**

```js
// vnode -> dom
const createElement = vnode => {
  let tag = vnode.tag
  let attrs = vnode.attrs || {}
  let children = vnode.children || []
  if (!tag) {
    return null
  }
  //创建元素
  let elem = document.createElement(tag)
  //属性
  for (let attrName in attrs) {
    if (attrs.hasOwnProperty(attrName)) {
      elem.setAttribute(attrName, attrs[attrName])
    }
  }
  //子元素
  children.forEach(childVnode => {
    //给elem添加子元素
    elem.appendChild(createElement(childVnode))
  })
  //返回真实的dom元素
  return elem
}

// diff
function updateChildren(vnode, newVnode) {
  let children = vnode.children || []
  let newChildren = newVnode.children || []

  children.forEach((childVnode, index) => {
    let newChildVNode = newChildren[index]
    if (childVnode.tag === newChildVNode.tag) {
      //深层次对比, 递归过程
      updateChildren(childVnode, newChildVNode)
    } else {
      //替换
      replaceNode(childVnode, newChildVNode)
    }
  })
}
```

## 参考文章

- [面试官问: 如何理解 Virtual DOM？](https://juejin.cn/post/6844903921442422791)
