---
title: qiankun微前端学习笔记
tags:
  - 微前端
  - 学习
cover: >-
  https://dogefs.s3.ladydaily.com/~/source/wallhaven/full/ex/wallhaven-exrqrr.jpg?w=2560&h=1440&fmt=webp
keywords: 学习 前端 微前端 qiankun
categories: 前端
abbrlink: 20669
date: 2024-01-22 15:16:49
---

# qiankun微前端学习笔记

> 参考链接：
>
> - [qiankun官网](https://qiankun.umijs.org/zh)
> - [微前端的核心价值](https://zhuanlan.zhihu.com/p/95085796)
> - [Why Not Iframe](https://www.yuque.com/kuitos/gky7yw/gesexv)
> - [【微前端】single-spa 到底是个什么鬼](https://zhuanlan.zhihu.com/p/378346507)
> - [黑马前端基于qiankun搭建微前端项目实战教程](https://www.bilibili.com/video/BV12T411q7dq)
> - [Vue3 + Vite + qiankun微前端实践](https://juejin.cn/post/7116002929168875533)
> - [如何让 vite 完美接入 qiankun](https://juejin.cn/post/7189117358697349178)

## 介绍

### 什么是微前端

微前端的概念其实类似于后端的微服务架构，出发点就是把一个复杂的大应用分为多个子应用，每个子应用可以独立运行、开发、部署，从而达到分布式，多团队并行的项目需求。

微前端也有一个主应用称为基座项目，嵌入其他n个子应用，子应用之间相互独立，不过也可以相互通信

### 价值

- 技术栈无关

  主框架不限制接入应用的技术栈，微应用具备完全自主权

- 独立开发、独立部署

  微应用仓库独立，前后端可独立开发，部署完成后主框架自动完成同步更新

- 增量升级

  在面对各种复杂场景时，我们通常很难对一个已经存在的系统做全量的技术栈升级或重构，而微前端是一种非常好的实施渐进式重构的手段和策略

- 独立运行时

  每个微应用之间状态隔离，运行时状态不共享



个人认为核心价值在于**技术栈无关**。由于历史原因，前端框架还是比较多的，`jQuery`, `Angular`, `React`, `Vue`，他们都有过大规模的应用，但是都不能保证谁笑到最后。不管是一个老项目，还是一个新项目，都需要考虑长远维护的问题，需要保证这套方案在3～5年还有生命力，但是目前很多前端框架/库有的也有过破坏性的更新，难以保证一年后，把所有依赖包都更新到最新版本还能正常运行。

实际上这种微前端的需求在ToB市场更加常见，ToC软件少有活得过3年以上的，ToB就常见多了，企业内部的中台后台的使用年限可以很长。同时企业软件的升级也会有很多麻烦，项目升级或者迁移绝对不是一个轻松的事情，因此，微前端这样的解决方案就很有价值，它**能够实现代码平滑的迁移，以及确保在若干年后还能用上当时热门的技术栈**。

所以说微前端最开始诞生的初衷，是为了给遗产项目续命。

### 架构姿势

秉承着核心价值为**「技术栈无关」**，那么在整个架构方案的是线上，就不应该违背这一原则，**应用之间不应该有任何直接或间接的技术栈、依赖、以及实现上的耦合**。

所以正确的微前端目标应该是：方案上和使用iframe一样简单，同时也解决的了iframe带来的体验问题



## 微前端方案对比

### iframe

iframe 最大的特性就是提供了浏览器原生的硬隔离方案，不论是样式隔离、js 隔离这类问题统统都能被完美解决。但他的最大问题也在于他的隔离性无法被突破，导致应用间上下文无法被共享，随之带来的开发体验、产品体验的问题。



1. url 不同步。浏览器刷新 iframe url 状态丢失、后退前进按钮无法使用。
2. UI 不同步，DOM 结构不共享。想象一下屏幕右下角 1/4 的 iframe 里来一个带遮罩层的弹框，同时我们要求这个弹框要浏览器居中显示，还要浏览器 resize 时自动居中..
3. 全局上下文完全隔离，内存变量不共享。iframe 内外系统的通信、数据同步等需求，主应用的 cookie 要透传到根域名都不同的子应用中实现免登效果。
4. 慢。每次子应用进入都是一次浏览器上下文重建、资源重新加载的过程。



其中有的问题比较好解决(问题1)，有的问题可以睁一只眼闭一只眼(问题4)，但有的问题我们则很难解决(问题3)甚至无法解决(问题2)，而这些无法解决的问题恰恰又会给产品带来非常严重的体验问题。

简而言之就是iframe隔离太彻底了，带来了很多不便。

### single-spa

链接：[https://zh-hans.single-spa.js.org/docs/getting-started-overview/](https://zh-hans.single-spa.js.org/docs/getting-started-overview/)

最早出现的微前端框架，可以兼容很多技术栈

首先在基座中注册所有子应用的路由，当URL改变时就会去匹配，匹配到哪个子应用就去加载对应的。

相较于iframe，single-spa中基座和各个子应用之间共享一个全局上文，并且不存在URL不同步和UI不同步的情况，但是也有缺点：

- 没有实现js隔离和css隔离
- 需要修改大量的配置，不能开箱即用

### qiankun

- 基于single-spa封装的，提供了更加开箱即用的API
- 技术栈无关
- HTML Entry的方式接入，像用iframe一样简单
- 实现了样式隔离和js隔离
- 资源预加载，在浏览器空闲时加载未打开的微应用资源

## 实战

项目目录：

```txt
qiankun-learn
 ├── micro-base // 基座
 ├── sub-react // react子应用，使用cra创建，webpack打包
 ├── sub-vue // vue子应用，vite创建打包
 └── sub-umi // umi脚手架创建的子应用
```

- 基座（主应用）：主要负责集成所有的子应用，提供一个入口能够访问所需要的子应用，尽量不要写复杂的业务逻辑
- 子应用：根据不同业务划分的模块，每个子应用都打包成`umd`模块的形式供基座来加载

### 基座

下载qiankun，在入口js文件中写如下代码

```js
import { registerMicroApps, start } from 'qiankun'

// 子应用列表
const apps = [
  {
    name: 'sub-react',
    entry: '//localhost:8000',
    container: '#sub-app',
    activeRule: '/sub-react',
  },
  {
    name: 'vue app',
    entry: { scripts: ['//localhost:7100/main.js'] },
    container: '#yourContainer2',
    activeRule: '/yourActiveRule2',
  },
]
// 生命周期
const lifeCycles = {
  beforeLoad: [
    async app => {
      console.log('beforeLoad', app)
    },
  ],
  beforeMount: [
    async app => {
      console.log('beforeMount', app)
    },
  ],
  afterMount: [
    async app => {
      console.log('afterMount', app)
    },
  ],
  beforeUnmount: [
    async app => {
      console.log('beforeUnmount', app)
    },
  ],
  afterUnmount: [
    async app => {
      console.log('afterUnmount', app)
    },
  ],
}
// 注册
registerMicroApps(apps, lifeCycles)
// 启动微服务
start()
```

api使用见：[registerMicroApps(apps, lifeCycles?)](https://qiankun.umijs.org/zh/api#registermicroappsapps-lifecycles)



### react子应用

使用vite创建，然后就是改造子应用的入口文件

1. 导出相应的生命周期钩子：[https://qiankun.umijs.org/zh/guide/getting-started](https://qiankun.umijs.org/zh/guide/getting-started)

```ts
import { createRoot } from 'react-dom/client'
import App from './App.tsx'
import './index.css'
import {
  renderWithQiankun,
  qiankunWindow,
} from 'vite-plugin-qiankun/dist/helper'

// 将render方法用函数包裹，供后续主应用与独立运行调用
function render(props: any) {
  const { container } = props
  const dom = container
    ? container.querySelector('#root')
    : document.getElementById('root')
  const root = createRoot(dom)
  root.render(<App />)
}

if (!qiankunWindow.__POWERED_BY_QIANKUN__) {
  render({})
}

renderWithQiankun({
  /**
   * bootstrap 只会在微应用初始化的时候调用一次，下次微应用重新进入时会直接调用 mount 钩子，不会再重复触发 bootstrap。
   * 通常我们可以在这里做一些全局变量的初始化，比如不会在 unmount 阶段被销毁的应用级别的缓存等。
   */
  bootstrap() {
    console.log('react app bootstrap')
  },
  /**
   * 应用每次进入都会调用 mount 方法，通常我们在这里触发应用的渲染方法
   */
  mount(props) {
    console.log('mount props', props)
    render(props)
  },
  /**
   * 应用每次 切出/卸载 会调用的方法，通常在这里我们会卸载微应用的应用实例
   */
  unmount(props: any) {
    console.log('unmount props', props)
    /* const { container } = props
    const mountRoot = container?.querySelector('#root')
    ReactDOM.unmountComponentAtNode(
      mountRoot || document.querySelector('#root')
    ) */
  },
  /**
   * 可选生命周期钩子，仅使用 loadMicroApp 方式加载微应用时生效
   */
  update(props: any) {
    console.log('update props', props)
  },
})
```

2. 配置微应用的打包工具

官方没有提供vite支持，需要自己下载插件配置：[vite-plugin-qiankun](https://www.viterc.cn/en/vite-plugin-qiankun.html)

```js
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react-swc'
import qiankun from ' '

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [react(), qiankun('sub-app')],
  server: {
    port: 8000,
  },
})
```

**尝试了几次失败了，暂时下放下，搞vue的试试**



### Vue子应用

```zsh
pnpm i vite-plugin-qiankun
```

vite配置文件

```ts
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import qiankun from 'vite-plugin-qiankun'

// https://vitejs.dev/config/
export default defineConfig({
  base: '/sub-vue',
  plugins: [
    vue(),
    qiankun('sub-vue', {
      useDevMode: true,
    }),
  ],
  server: {
    port: 7000,
    cors: true,
    origin: 'http://localhost:7000',
  },
})
```

修改main.ts

```ts
import { createApp } from 'vue'
import './style.css'
import App from './App.vue'
import {
  renderWithQiankun,
  qiankunWindow,
} from 'vite-plugin-qiankun/dist/helper'

let app: any
if (!qiankunWindow.__POWERED_BY_QIANKUN__) {
  createApp(App).mount('#sub-vue-app')
} else {
  renderWithQiankun({
    mount(props) {
      console.log('🚀 ~ mount:', props)
      app = createApp(App)
      app.mount('#sub-vue-app')
    },
    unmount() {
      console.log('🚀 ~ unmount:', app)
      app.unmount('#sub-vue-app')
      app._container.innerHTML = ''
      app = null
    },
    bootstrap() {
      console.log('🚀 ~ bootstrap')
    },
    update() {
      console.log('🚀 ~ update')
    },
  })
}
```

同时将vue子应用的index.html中的app改为sub-vue-app，不然会出现俩个id是app的div

不过也存在问题，挂载的时候会闪一下，预计是样式没有隔离的问题

qiankun官方还没有支持vite，引入的三方库似乎没有解决js隔离和样式隔离

明天再进行react子应用的处理，改成webpack创建react子应用
