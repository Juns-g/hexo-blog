---
title: Vue面试知识点
tags:
  - Vue
  - 面试
  - 前端
categories: 面试
cover: 'https://w.wallhaven.cc/full/9d/wallhaven-9d28vd.png'
abbrlink: 19697
date: 2023-05-09 16:50:23
---

# Vue

## 响应式原理

**数据变化更新视图，试图变化更新数据**

**数据劫持 + 发布订阅模式**

Vue2

- 利用的是`Object.defineProperty()`，劫持 data 对象中各个属性的`getter`和`setter`
- 缺点是：
  - 监听不到数组值的变化
  - 监听不到对象属性的新增或者删除
- 解决方案：使用`vm.$set`，`vm.$delete`
- 解除响应式：对数据深拷贝，了或者用`Object.freeze()`对数据进行冻结

Vue 响应式原理是 Vue 框架的核心，它实现了数据和视图的双向绑定。在 Vue2 中，Vue 的响应式原理基于 Object.defineProperty 实现，它会将 data 中的属性转化为 getter/setter，并在 getter/setter 中进行依赖收集和派发更新。当我们修改数据时，Vue 会检测到数据变化，并通知相关的视图进行更新。

具体实现流程如下：

1. 初始化时，将 data 中的属性通过 Object.defineProperty 方法转化为 getter/setter；
2. 在 getter 中收集依赖，即将依赖的 watcher 对象添加到当前属性的依赖列表中；
3. 在 setter 中触发更新，即通知当前属性的所有依赖项进行更新。

在 Vue3 中，为了提高性能，Vue 响应式原理采用了 ES6 的 Proxy 代理对象来代替 Object.defineProperty。使用 Proxy 可以直接监听对象的整个变化，而不像 Object.defineProperty 只能监听对象的属性变化。

具体实现流程如下：

1. 初始化时，通过 Proxy 代理对象监听 data 对象；
2. 当我们修改 data 对象时，会触发代理对象的 set 方法；
3. 在 set 方法中，通知所有依赖项进行更新。

相比于 Vue2，Vue3 使用 Proxy 实现响应式原理，性能得到了极大的提升，同时还支持了**动态添加**和**删除属性**等高级功能。

## defineProperty 和 Proxy 的区别

- Object.defineProperty 是 Es5 的方法，Proxy 是 Es6 的方法
- defineProperty 不能监听到数组下标变化和对象新增属性，Proxy 可以
- defineProperty 是劫持对象属性，Proxy 是代理整个对象
- defineProperty 局限性大，只能针对单属性监听，所以在一开始就要全部递归监听。Proxy 对象嵌套属性运行时递归，用到才代理，也不需要维护特别多的依赖关系，性能提升很大，且首次渲染更快
- defineProperty 会污染原对象，修改时是修改原对象，Proxy 是对原对象进行代理并会返回一个新的代理对象，修改的是代理对象

### 手写

[一文搞懂 Object.defineProperty 和 Proxy，Vue3.0 为什么采用 Proxy？ - 掘金 (juejin.cn)](https://juejin.cn/post/7069397770766909476)

**Object.defineProperty()**

语法：`Object.defineProperty(obj, prop, descriptor)`

```js
let person = {}
let nameT = '张三'

// 在person对象上添加了一个属性叫 nameString ,这个属性的值是 nameT
Object.defineProperty(person, 'nameString', {
  get() {
    return nameT
  },
  set(val) {
    nameT = val
  },
})

//当读取person对象的nameString属性时，触发get方法
console.log(person.nameString)
// 直接修改nameT变量的值
nameT = '李四'
console.log(person.nameString)
// 当设置person对象的nameString属性时，触发set方法
person.nameString = '王五'
console.log(person.nameString)
```

- 监听对象的多个属性比较麻烦

- 对于数组的 push 等操作比较难监听，vue2 的方法是在 Array 原型上重写了
- 嵌套对象需要递归监听
- 对象的新增属性要手动监听

**Proxy**

语法：`const p = new Proxy(target, handler)`

```js
let person = {
  name: '张三',
  age: 12,
}

let handler = {
  get(obj, key) {
    console.log('get')
    return key in obj ? obj[key] : '不存在'
  },
  set(obj, key, val) {
    console.log('set')
    obj[key] = val
    // MDN上明确指出set()方法应该返回一个布尔值，否则会报错TypeError。
    return true
  },
}

// 代理对象
let proxy = new Proxy(person, handler)

console.log(proxy.name)
proxy.age = 15
console.log(proxy.age)
```

## [vue2 和 vue3 的区别](https://juejin.cn/post/7160962909332307981#heading-2)

1. 响应式原理，`defineProperty`->`Proxy`，可以深层响应
2. vue2 只能有一个根节点，3 可以有多个，支持碎片`Fragments`
3. 3 使用组合式 API，2 使用选项式 API
4. 支持`Tree-Shaking`，去除无用代码，打包体积更小
5. 支持在 `<style></style>` 里使用 `v-bind`，给 CSS 绑定 JS 变量(`color: v-bind(str)`)
6. 用 `setup` 代替了 beforeCreate 和 created 这两个生命周期

## 双向绑定

通过数据劫持结合发布订阅模式的方式实现的

## [生命周期](https://cn.vuejs.org/api/options-lifecycle.html)

对比 Vue2 基本就是名字前面加了`on`，`beforeDestroy`->`onBeforeUnmount`，`destroyed`->`onUnmounted`，用`setup`代替了`beforeCreate`和`created`，新增了两个开发环境的调试钩子

1. `setup`，最开始的
2. `onBeforeMount`，Vue 实例挂载之前执行
3. `onMounted`，挂载完成
4. `onBeforeUpdate`，数据发生变化之前
5. `onUpdated`，数据发生变化之后
6. `onBeforeUnmount`，销毁之前
7. `onUnmounted`，销毁之后
8. `onActivated`，keep-alive 组件激活时执行
9. `onDeactivated`，keep-alive 组件销毁时
10. `onErrorCaptured`，捕获后代组件之间传递错误
11. ...还有的去官方文档看

常用的基本就是 mount 相关

![生命周期](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/60f518fa89ba44bb8b59911ecd65a8a4~tplv-k3u1fbpfcp-zoom-in-crop-mark:4536:0:0:0.awebp?)

## v-for 的 key 值

为什么要 key，diff 算法，key 的考量，设置为 index 会有什么问题

## 组件通信

1. `props`传递数据，父传子，子用`defineProps`接受
2. `emit`传递事件，子传父，子用`defineEmits`定义
3. `provide`和`inject`传递数据，父传子，但是层级深也可以
4. `Vuex`或者`Pinia`，状态管理工具

Vue3 中的：

1. `props/context`：通过 props 将数据从父组件传递给子组件，在子组件中使用 setup 函数中的 context 对象访问父组件的属性或方法。
2. `provide/inject`：与 props/context 类似，provide/inject 同样用于父子组件之间的通信，但是它可以让祖先组件向所有后代组件注入数据，而不仅仅是直接后代组件。
3. `ref/teleport`：通过 ref 获取组件的引用，并使用 teleport 将组件挂载到指定的 DOM 节点上。
4. `EventBus/Emitter`：与 Vue2 类似，可以使用一个独立的 Vue 实例作为事件总线，在组件之间发送和接收自定义事件。
5. `Vuex 4.x/Pinia`: 可以使用 Vuex 来管理全局状态，但是在 Vue3 中有了更好的支持，使用新的 API 替换了旧 API，并且提供了更好的类型安全性和开箱即用的响应式需求。

**父组件调用子组件的方法：**

## Router 路由

> [2022 年我的面试万字总结(Vue 下)](https://juejin.cn/post/7151604799077613599#heading-2)
>
> [Vue-Router 面试题汇总 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903961745440775)

SPA 单页面应用，首屏渲染慢

### route 和 router

- `$route`是路由信息对象，包括 path，params，hash 等信息参数

- `$router`是路由示例对象，包括跳转方法，钩子函数等

### hash 和 history 模式

[浅谈前端路由原理 hash 和 history - 掘金 (juejin.cn)](https://juejin.cn/post/6993840419041706014)

- hash 多一个 #，# 后面的路径变化时不会重新发请求，而是触发`onhashchange`事件
- hash 不会刷新页面，不利于 SEO
- history 可以更新 url 而不重新发起请求

### [路由守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

使用场景：登录判断，未登录用户跳转到 login 页面

### 一些面试题

## Pinia

### 基本

基本模式：

- `state`: 状态
- `actions`: 修改状态（包括同步和异步，pinia 中没有`mutations`）
- `getters`: 计算属性

使用`Setup Store`：

- `ref()` 就是 `state` 属性
- `computed()` 就是 `getters`
- `function()` 就是 `actions`

最后要 return，其他和 vue3 的 setup 差不多

### 特点

- 支持 vue2， vue3
- 支持 setup，不必像 vuex 中定义`state`,`mutations`,`actions`,`getters`
- 支持`state`,`actions`,`getters`
- 丢弃了 `mutations`，开发时候不用根据同步异步来决定使用 `mutations `或 `actions`
- 对 ts 友好

不要直接解构`const {name} = store`，会丢失响应式，可以`const {name} = storeToRefs(store) `，不过 action 可以直接解构

## ref 和 reactive

## computed 和 watch

`computed`：

- 支持缓存，依赖数据变化了才重新计算
- 不支持异步，有异步时无法监听数据变化
- 默认走缓存
- 如果一个属性是由其他属性计算而来的，这个属性依赖其他的属性，一般会使用 computed
- 如果 computed 属性的属性值是函数，那么**默认使用 get 方法**，函数的返回值就是属性的属性值；在 computed 中，属性有一个 get 方法和一个 set 方法，当数据发生变化时，会调用 set 方法。

`watch`：

- 不支持缓存，数据变化就会触发相应操作
- 支持异步监听
- 接受俩个参数，`newValue`和`oldValue`
- 还有俩个参数，`immediate`组件加载就立刻触发，`deep`深度监听，在复杂数据类型中使用，例如[{age:1}]，但是无法监听数组和对象内部发生的变化

**总结：**

- `computed` 计算属性 : 依赖其它属性值，并且 `computed `的值有缓存，只有它依赖的属性值发生改变，下一次获取 `computed `的值时才会重新计算 `computed `的值。
- `watch` 侦听器 : 更多的是**观察**的作用，**无缓存性**，类似于某些数据的监听回调，每当监听的数据变化时都会执行回调进行后续操作。
- 计算属性是只读的，不能改。可以用 watch 实现类似的功能
- `computed`第一次加载就监听；`watch`不监听
- `computed`函数里面必须要有 return；`watch`不用

**运用场景：**

- 当需要进行数值计算,并且依赖于其它数据时，应该使用 computed，因为可以利用 computed 的缓存特性，避免每次获取值时都要重新计算。
- 当需要在数据变化时执行异步或开销较大的操作时，应该使用 watch，使用 watch 选项允许执行异步操作 ( 访问一个 API )，限制执行该操作的频率，并在得到最终结果前，设置中间状态。这些都是计算属性无法做到的。

## v-if 和 v-show

- if 是 DOM 的添加删除元素，有编译/卸载的过程，show 仅仅是 CSS 的`display:none;`
- if 是条件渲染，会触发生命周期，show 不会
- if 效率低，show 高，频繁切换用 show
