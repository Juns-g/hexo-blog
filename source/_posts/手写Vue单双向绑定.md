---
title: 手写Vue单双向绑定
tags:
  - Vue
  - 面试
  - 前端
categories: 面试
cover: 'https://files.codelife.cc/wallhaven/full/o3/wallhaven-o3wel5.jpg?x-oss-process=image/resize,limit_0,m_fill,w_2560,h_1440/quality,Q_92/format,webp'
abbrlink: 40037
date: 2024-02-12 14:59:48
---

# 手写 Vue 单双向绑定

> 代码仓库地址：https://github.com/Juns-g/vue-insights/tree/main/src/%E5%AE%9E%E7%8E%B0%E5%8D%95%E5%8F%8C%E5%90%91%E7%BB%91%E5%AE%9A

## 前言

首先我们得了解这是个啥 🤓。先介绍一下数据驱动视图，可以理解为我们更改了一个数据，对应的展示视图就会自动变化：

```html
<h1>name</h1>
<script>
  let name = 'juns' // 这个时候展示应该是 <h1>juns</h1>
  // ...
  name = 'other' // 这个时候视图自动改变，变成了<h1>other</h1>
</script>
```

也就是说，我们之需要改变数据，对应的视图展示的内容其实就是数据，数据变化时，视图也会自动随之变化。单向绑定就可以理解成这样，我们使用特定语法把一个数据与视图绑定，然后改变数据的时候视图就会自动更新，比如这样：

```html
<h1>{{ name }}</h1>
<input v-bind:value="name" />
<script>
  let name = 'juns'
</script>
```

其中的`{{ 变量 }}`还有`v-bind:value="变量"`就是 vue 中的单向绑定语法。

那么双向绑定也不难理解，就是视图也可以驱动数据，简单的例子就是，我们通过 input 输入内容，大概率是希望它可以改变数据的，比如这样：

```html
<input v-model="name" />
<h1>{{ name }}</h1>
<script>
  let name = 'juns'
</script>
```

当我们在 input 中输入时，理想的状态是 input 中的内容和 h1 的内容是同时变化的，不过直接写 v-model 属性可能有一些难以理解，他可以等价于：

```html
<input
  :value="name"
  @input="updateName"
/>
<h1>{{ name }}</h1>
<script>
  let name = 'juns'
  function updateName(event) {
    name = event.target.value
  }
</script>
```

其中使用的都是 vue 的语法与原生 js 的混合，可能会影响理解。`:value`其实就是`v-bind:value`的简写，`@input`可以理解为给这个 input 标签添加了 input 事件监听：`document.querySelector(input).addEventListener('input', (event)=>{ name = event.target.value })`。

也就是说，我们在 input 中输入的时候，vue 帮我们把输入的值，赋值给了 name 变量，然后由于数据驱动试图，又去更新了 h1 的显示内容，这样就可以理解为双向绑定。

## 简单实现

上面我们也理解了单向绑定和双向绑定，现在我们可以简单地尝试实现一下，只考虑简单场景来实践理论：

```html
<input />
<h1>inputVal</h1>
<script>
  let inputVal = 'hello vue'
</script>
```

我们期望：一开始进入页面看到 input 和 h1 都显示 hello vue，修改 input 中的值的时候，input 和 h1 都同步显示新值。

先想想思路，我们需要用 inputVal 驱动 input 和 h1 这俩个元素的内容，也就是说，当 inputVal 改变时，我们也需要更新元素内容，那么可以劫持一下`window.inputVal`的 set，在当中加入更新的逻辑：

```html
<input />
<h1>inputVal</h1>
<script>
  // 视图更新函数
  function updateView(val) {
    const input = document.querySelector('input')
    const h1 = document.querySelector('h1')
    h1.textContent = val
    input.value = val
  }

  let inputVal = 'hello vue'
  updateView(inputVal) // 首次更新

  Object.defineProperty(window, 'inputVal', {
    get() {
      return inputVal
    },
    set(newVal) {
      inputVal = newVal
      updateView(inputVal) // 数据更新时更新视图
    },
  })
  setTimeout(() => {
    window.inputVal = '43543643'
  }, 2000)
</script>
```

其实这样就已经完成了单向绑定，当我们修改`window.inputVal`的时候，视图就会自动更新了，接下来是需要实现双向绑定。我们需要在 input 输入的时候改变`window.inputVal`的值，那么思路也是显而易见，直接给 input 绑定事件，获取里面的值来更新数据，从而推动视图更新：

```js
input.addEventListener('input', event => {
  window.inputVal = event.target.value
})
```

其实现在就已经实现了单双向绑定了，不过这只是一个非常简陋的版本，事实上需要考虑很多，下面尝试尝试，附上完整代码：

```html
<input />
<h1>inputVal</h1>
<script>
  // 视图更新函数
  const input = document.querySelector('input')
  const h1 = document.querySelector('h1')
  function updateView(val) {
    h1.textContent = val
    input.value = val
  }

  let inputVal = 'hello vue'
  updateView(inputVal) // 首次更新

  Object.defineProperty(window, 'inputVal', {
    get() {
      return inputVal
    },
    set(newVal) {
      inputVal = newVal
      updateView(inputVal) // 数据更新时更新视图
    },
  })

  input.addEventListener('input', event => {
    window.inputVal = event.target.value
  })
</script>
```

## 仿 vue 实现

> 笔者对于 vue 的响应式原理还不够熟悉，仅仅是简易的模仿，细节有待商榷。这里的思路参考的是 vue2 版本

### new Vue(options)

当我们`new Vue()`的时候，需要传入一个选项`options`，其中这个对象最重要的俩个属性是 el 和 data，el 是 vue 挂载的节点，data 则是响应式数据，用于驱动试图。

```js
class Vue {
  constructor(options) {
    this.$options = options
    this.$data = options.data
    // 将data转换成响应式数据
    // 编译渲染视图
  }
}
```

这里就可以明显看到，我们有俩个地方需要去补充，把`vue.$data`转换成响应式数据，以及将数据与视图链接起来。

### 响应式数据转换

在 vue2 中，都是使用的`Object.defineProperty()`来劫持一个对象的属性，其实这也就是 data 需要是一个对象的原因。

使用`defineProperty`之后我们是可以监听到这个对象属性的变化了，但是我们也需要进行一些处理，可以把这个处理副作用的功能用一个类`Watcher`来表示。

我们还需要一个地方来存储副作用/依赖，可以使用 Dep 类来做处理

### 编译

这是一个很复杂的部分，这里就只用简化一些的写法了，只处理`v-model`还有`{{ }}`。他的核心就是解析 html 文本，然后再进行处理再渲染。

### 完整代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>实现单双向绑定</title>
  </head>
  <body>
    <div id="app">
      <input
        class="input"
        type="text"
        v-model="inputVal"
      />
      <h1>{{ inputVal }}</h1>
      <button @click="addOne">+1</button>
      <input v-bind:placeholder="inputVal" />
    </div>
  </body>
  <style>
    #app {
      margin-left: 400px;
      margin-top: 200px;
      height: 300px;
    }
  </style>
  <script src="./index.js"></script>
  <script>
    const app = new Vue({
      el: '#app',
      data: {
        inputVal: 'hello world',
      },
      methods: {
        addOne() {
          this.$data.inputVal += '1'
        },
      },
    })
  </script>
</html>
```

```js
class Vue {
  constructor(options) {
    console.log('🚀 ~ options:', options)
    this.$options = options
    this.$data = options.data
    this.$methods = options.methods
    observe(this.$data)
    new Compile(options.el, this)
  }
}

function observe(data) {
  if (typeof data !== 'object' || data === null) {
    return data
  }
  Object.keys(data).forEach(key => {
    defineReactive(data, key, data[key])
  })
}

function defineReactive(obj, key, val) {
  observe(val) // 递归所有属性的值
  const dep = new Dep()
  Object.defineProperty(obj, key, {
    get() {
      dep.depend()
      console.log('🚀 ~ dep:', dep)
      return val
    },
    set(newVal) {
      if (val === newVal) return val
      val = newVal
      dep.notify()
    },
  })
}

// 存放依赖
class Dep {
  constructor() {
    this.subs = []
  }
  notify() {
    this.subs.forEach(effect => {
      effect.update()
    })
  }
  depend() {
    if (!window.nowWatcher || this.subs.includes(window.nowWatcher)) {
      return
    }
    this.subs.push(window.nowWatcher)
  }
}

class Watcher {
  constructor(vm, exp, callback) {
    this.cb = callback
    this.vm = vm
    this.exp = exp
    this.value = this.get()
  }
  get() {
    window.nowWatcher = this
    const val = this.vm.$data[this.exp]
    window.nowWatcher = null
    return val
  }
  update() {
    const oldVal = this.value
    const newVal = this.get()
    this.cb.call(this.vm, newVal, oldVal)
  }
}

class Compile {
  constructor(el, vm) {
    this.vm = vm
    const dom = document.querySelector(el)
    this.compile(dom)
  }
  compile(node) {
    const childNodes = node.childNodes
    childNodes.forEach(cNode => {
      console.dir(cNode)
      //  {{ }}
      if (this.isInter(cNode)) {
        this.compileText(cNode)
      } else {
        if (!cNode.attributes) return
        const attrs = cNode.attributes
        Array.from(attrs).forEach(attr => {
          const { name: attrName, value: exp } = attr
          if (attrName.startsWith('v-bind:')) {
            const attrValue = attrName.substring(7)
            this.vBind(cNode, attrValue, exp)
            return
          }
          if (attrName.startsWith('v-')) {
            const type = attrName.substring(2)
            this[type](cNode, exp)
            return
          }
          if (attrName.startsWith('@')) {
            const type = attrName.substring(1)
            this[type](cNode, exp)
          }
        })
      }
    })
  }
  isInter(node) {
    const reg = /\{\{(.*)\}\}/
    return reg.test(node.textContent)
  }
  getInter(node) {
    const reg = /\{\{(.*)\}\}/
    const exp = node.textContent.replace(reg, '$1').trim()
    return exp
  }
  compileText(node) {
    const exp = this.getInter(node)
    node.textContent = this.vm.$data[exp]
    new Watcher(this.vm, exp, val => {
      node.textContent = val
    })
  }
  model(node, exp) {
    console.log('🚀 ~ model:')
    node.value = this.vm.$data[exp]
    new Watcher(this.vm, exp, val => {
      node.value = val
    })
    node.addEventListener('input', event => {
      this.vm.$data[exp] = event.target.value
    })
  }
  click(node, exp) {
    console.log('🚀 ~ exp:', exp)
    const fn = this.vm.$methods[exp]
    node.onclick = () => {
      fn.call(this.vm)
    }
  }
  vBind(node, attrValue, exp) {
    console.dir(node)
    node[attrValue] = this.vm.$data[exp]
    new Watcher(this.vm, exp, val => {
      node[attrValue] = val
    })
  }
}
```

参考文章：[ vue2 熬夜编写教你们去深入理解双向绑定，指令并手写一个](https://juejin.cn/post/7000768402205704206)
