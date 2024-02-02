---
title: Vue-Router
tags:
  - Vue
  - 面试
  - 前端
categories: 前端
cover: >-
  https://picx.zhimg.com/70/v2-5f4987b5e19bd68dc29f3959df2510bf_1440w.avis?source=172ae18b&biz_tag=Post
abbrlink: 426
date: 2024-02-01 15:47:20
---

# Vue-Router

## 前言

`Vue-Router`就是`Vue`的官方路由库，至于路由是什么，以及为什么需要他，他解决什么问题，有什么好处，下面来展开讲讲。

## 什么是路由

路由（Route）就是指 App 如何响应 URL 的机制。它用于把不同的 URL 映射到 App 的不同页面/试图，从而实现页面之间的导航和切换。

在以前，我们通过地址栏切换不同的 URL 时，会向服务器发送请求，服务器返回拼接好的 HTML 文件，然后展示在页面，这也称为后端路由。他的优点是减少了前端的压力，因为 HTML 由后端拼接，不过他也有缺点：依赖网速、项目很大时对服务器的压力很大、前后端没有分离。

现在的话，得益于 SPA 的兴起，出现了前端路由的概念。

> SPA: Singal Page Application，单页应用。就是指这个 Web 项目只有一个 HTML 页面，SPA 通过在单个 HTML 页面中加载并动态更新内容，实现了无需刷新整个页面的用户体验。

页面的刷新、跳转都和路由相关，也实现了前后端分离，前端负责页面呈现，仅仅需要获取数据的时候再向后端请求，模块化程度更高，用户体验也更好了。

## 路由有哪些模式？

### Hash 模式

形如：https://example.com/#/home

- 把前端路由的路径用`#`拼接在真实 url 后面，不过会覆盖锚点定位元素的功能，可以监听 URL 的 hash 部分的变化来更新页面
- 路由处理都放在客户端，路由变化时，只改变 URL 中的 hash 部分，而且也不会想服务器发送新请求，而是触发`onhanshchange`事件
- 只有`#`之前的内容才会包含在请求中，hash 不会提交到服务器
- hash 值的改变会在浏览器的访问历史中新增一个记录，所以可以通过浏览器前进、回退按钮改变
- 不会造成页面 404，因为路由信息都是在客户端解析处理，服务器只提供初始 HTML 和资源，不关心路由匹配问题

```js
// onhashchage事件，可以在window对象上监听这个事件
window.onhashchange = function (event) {
  console.log(event.oldURL, event.newURL)
  const hash = location.hash.slice(1)
  console.log('🚀 ~ hash:', hash)
}
```

- 可以`location.hast = '#/xxx'`来修改 hash 值，触发更新
- 可以通过监听`onhashchange`事件来监听浏览器的前进、后退

### History 模式

[History API](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)

- HTML5 推出的功能，更加美观
- 刷新页面时，会按照路径发送真是的资源请求，如果 nginx 没有配置的话会出现 404
- 需要通过服务端支持允许地址可访问，如果没设置可能导致 404
- 提供了[pushState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/pushState)和[replaceState](https://developer.mozilla.org/zh-CN/docs/Web/API/History/replaceState)来记录路由状态，只改变 URL，不刷新页面
- 通过`onpopstate`监听 history 变化

```js
window.onpopstate = function (e) {
  console.log('🚀 ~ e:', e)
  console.log('🚀 ~ location:', location)
}

history.pushState({}, null, '/a') // 路由压栈，记录浏览器的历史栈 不刷新页面
history.pushState({}, null, '/b')
history.back() // 返回
history.forward() // 前进
history.go(-2) // 后退2次
```

## 基本使用

上面是介绍的路由，现在到了 Vue 的官方路由，他为 Vue 提供了路由配置和导航功能。

功能包括：

- **路由映射**: 将 URL 映射到 Vue 组件。
- **嵌套路由映射**: 可以在路由下定义子路由，实现复杂页面结构和嵌套组件的渲染。
- **动态路由选择**: 可以通过路由参数传递数据。
- **模块化、基于组件的路由配置**: 每个路由都可以指定一个 Vue 组件。
- **路由参数、查询、通配符**
- **导航守卫**：提供了全局导航守卫和路由级别的导航守卫，可以在路由跳转前后执行操作，比如用户权限验证。
- **展示由 Vue.js 的过渡系统提供的过渡效果**: 可以为路由组件添加过渡效果，使页面切换更加平滑和有动感。
- **细致的导航控制**: 可以通过编程式导航（通过 js 控制路由跳转）和声明式导航（通过组件实现跳转）实现页面的跳转。
- **HTML5 history 模式或 hash 模式**
- **可定制的滚动行为**:当页面切换时，Vue Router 可以自动处理滚动位置。定制滚动行为，例如滚动到页面顶部或指定的元素位置。
- **URL 的正确编码**

### 路由组件

`<router-link/>`：就是一个添加了一些属性的`a`标签，可以让 Vue-Router 在不刷新页面的情况下更改 URL，源代码：[RouterLink.ts](https://github.com/vuejs/router/blob/main/packages/router/src/RouterLink.ts)

`<router-view/>`：他显示与 URL 对应的组件，源代码：[RouterView.ts](https://github.com/vuejs/router/blob/main/packages/router/src/RouterView.ts)

### 几个概念

第一次看见很容易弄混这几个单词的概念：route、routes、router

#### route

指的是当前路由信息对象，只读，可以通过`this.$route`访问

```js
const route = {
  name: '', // 路由名称
  fullPath: '/', // 当前路由完整路径，包含查询参数和 hash 的完整路径
  path: '/', // 字符串，对应当前路由的路径
  query: {},
  hash: '', // 当前路由的 hash 值
  params: {}, // 一个 key/value 对象，表示 URL 查询参数。跟随在路径后用'?'带的参数
  matched: [], // 包含当前路由的所有嵌套路径片段的路由记录
  meta: {}, // 路由文件中自赋值的meta信息
  href: '/',
}
```

#### routes

就是由 route 构成的数组，也是给 VueRouter 来配置路由实例的对象

```js
const routes = [route1, route2]
```

#### router

是 VueRouter 的实例对象，全局路由对象，可以通过`this.$router`访问。他也提供了一些方法：

```js
// 导航守卫
router.beforeEach((to, from, next) => {
  /* 必须调用 `next` */
})
router.beforeResolve((to, from, next) => {
  /* 必须调用 `next` */
})
router.afterEach((to, from) => {})

动态导航到新路由
router.push
router.replace
router.go
router.back
router.forward
```

### 动态路由

这是一个十分常见的场景，比如`https://example.com/user/1`，user 后面的那个 id 是需要动态匹配的，可以这样写

```js
const routes = [
  {
    path: '/user/:id',
    name: 'User',
    components: User,
  },
]
```

可以在组件里，通过`route.param`获取动态值：

```js
const route = {
  // ...
  params: { id: '1' },
}
```

### 路由传参

这也是一个常见的场景，从 A 组件跳转到 B 组件，需要携带 id 之类的，在 VueRouter 中有俩种传参方式，但是也是有所区别的。

#### params 传参

- 必须使用 name 跳转
- 参数不显示在 url 上
- 强制刷新会清空参数

```js
router.push({ path: '/', params: { p: 1 } }) // 这样等于没有传params，因为query会覆盖params
router.push({ name: 'home', params: { p: 1 } }) // 其实也失效了
```

在[更新日志](https://github.com/vuejs/router/blob/main/packages/router/CHANGELOG.md#414-2022-08-22)中提到，未声明的 params 会失效，但是如果我们还是想要实现如同 params 的效果呢？

可以使用上文提到的，history API 的功能 😆

```js
const params = {
  p: 1,
}
router.push({ name: 'home', state: { params } })

// 接受
const historyParams = history.state.params
```

#### query 传参

- 可以用 name 传参，也可以用 path
- 参数会显示在 url 上
- 刷新参数不丢失

```js
// 拼接字符串
router.push('/home?username=xxx&age=18')

// query 对象
router.push({
  path: '/home',
  query: {
    username: 'xxx',
    age: 18,
  },
})
```

## [路由守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

有些 Web App 需要做鉴权系统，就是某些页面需要一定的权限才可以看，这时候就可以使用路由守卫来实现，如果这个用户没有权限，就返回原本的页面，或者跳转到提示页面。

也可以把路由守卫理解为路由跳转过程中的生命周期函数，具体可以分为：全局路由守卫、路由独享守卫、组件路由守卫

### 全局路由守卫

总共有三个：

- 全局前置守卫：router.beforeEach
- 全局解析守卫：router.beforeResolve
- 全局后置钩子：router.afterEach

#### beforeEach(to, from, next?)

接受参数：

- to：要进入的 route
- from：当前正在离开的路由

一般数据结构如下，和 route 很类似

```js
const to = {
  fullPath: '/',
  path: '/',
  query: {},
  hash: '',
  name: 'home',
  params: {},
  matched: [],
  meta: {},
  href: '/',
  redirectedFrom: undefined,
}
```

使用如下

```js
router.beforeEach((to, from) => {
  console.log('🚀 ~ to:', to)
  console.log('🚀 ~ from:', from)
  return true
})
```

其中可以 return 的值如下：

- `false`：取消这次路由跳转操作，重置到 from 的路由。
- 一个路由地址：相当于重定向到另外一个路由。
- `undefined`或`true`：没有拦截操作，正常跳转。

`beforeEach`还有一个可选参数`next`，在以前的版本是必选的，现在有过改动，当使用它时，我们必须确保`next`只被调用一次。

#### beforeResolve(to, from, next?)

他也是会每次都触发，不过他的调用时机是导航被确认之前、**所有组件内守卫和异步路由组件被解析之后**，也就是 `beforeEach` 和 组件内 `beforeRouteEnter` 之后，`afterEach`之前调用。

#### afterEach(to, from)

一般用于分析、更改标题等

### 路由独享守卫

可以直接在路由的配置上定义`beforeEnter`守卫

```js
const routes = [
  { name: 'home', path: '/', component: HelloWorld },
  {
    name: 'about',
    path: '/about',
    component: AboutPage,
    beforeEnter: (to, from) => {
      // ...
    },
  },
]
```

这个钩子只有在进入路由时触发，不会在 `params`、`query` 或 `hash` 改变时触发。这个属性也可以接受一个函数数组，有利于代码复用。

### 组件路由守卫

- beforeRouteEnter： 在渲染该组件的对应路由被验证前调用
- beforeRouteUpdate：在当前路由改变，但是该组件被复用时调用
- beforeRouteLeave：在导航离开渲染该组件的对应路由时调用

vue3 的 setup 中提供了钩子`onBeforeRouteLeave`和`onBeforeRouteUpdate`

### 完整的路由解析流程

1. 导航被触发。
2. 在失活的组件里调用 `beforeRouteLeave` 守卫。
3. 调用全局的 `beforeEach` 守卫。
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫(2.2+)。
5. 在路由配置里调用 `beforeEnter`。
6. 解析异步路由组件。
7. 在被激活的组件里调用 `beforeRouteEnter`。
8. 调用全局的 `beforeResolve` 守卫(2.5+)。
9. 导航被确认。
10. 调用全局的 `afterEach` 钩子。
11. 触发 DOM 更新。
12. 调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数，创建好的组件实例会作为回调函数的参数传入。

## 参考文章

- [史上最全 vue-router 讲解 ！！！](https://juejin.cn/post/7243611408628498491)
- [Vue Router 官网](https://router.vuejs.org/zh/)
