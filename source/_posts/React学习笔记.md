---
title: React学习笔记.md
tags:
  - React
  - 学习
  - 前端
categories: 前端
cover: 'https://www.krishnakantyadav.com/blog/wp-content/uploads/2022/03/ReactJS-Everything-You-Should-Know-About-It.png'
abbrlink: 45055
date: 2023-12-23 01:33:56
---

# React学习笔记

> 学习的视频链接：[千锋教育前端React18系统精讲教程，基于最新版本新特性源码级剖析](https://www.bilibili.com/video/BV13h4y177jW/?p=4&share_source=copy_web&vd_source=fec74aa0dc6bc131c090122b391ab233)

## 预备知识

### ESLint 和 Prettier配置

#### ESLint

配置自动检查插件`pnpm install vite-plugin-eslint`，然后去 vite 的配置文件中导入使用，效果就是代码如果过不了 esint，网页直接弹窗报错😐。我感觉不如直接编辑器中自动提示，之前在小红书实习的时候有配置过自动校验，这个可以研究一下🧐。

#### Prettier

装插件就可以，配置看自己喜好



## JSX

- jsx 和 React 是独立的，只不过经常放在一起使用
- jsx需要编译才能被浏览器识别

```tsx
function App() {
  return <div className='app'>App</div>
}

// 会编译成一个对象
const obj = {
  type: 'div',
  props: {
    className: 'app',
    children: 'App',
  },
  key: null,
  ref: null,
}
```

- 一些和 html 的区别
  - 标签名要小写
  - 标签必须闭合
  - class -> className, for -> htmlFor
  - 非自定义属性用驼峰命名法，`data-id`这样的自定义属性是合法的
  - `{}`里面可以写 js，但是不能写 if 和 for，以及对象函数等（这里可以研究一下为什么🧐）
  - 根元素唯一，可以使用`<> </>包裹`，等同于`<Fragment> </Fragment>，如果需要给它加属性的话，就不能写空标签了`



## 样式

- 行内样式，略过🥱
- 全局样式 `index.css`，在一个 tsx 文件导入后会影响其他的文件
- 局部样式 `index.module.css`，样式隔离，默认不支持驼峰访问，需要配置（可以研究底层实现🤗）

```tsx
import style from './assets/index.module.css'

function App() {
  return (
    <>
      <div className={style.box}>
        <h1 className={style['head-title']}>32</h1>
        <h1 className={style.headTitle}>需要配置</h1>
      </div>
    </>
  )
}

export default App
```

- 局部样式配置支持驼峰写法

```js
// vite.config.js
 css: {
    modules: {
      localsConvention: 'camelCase'
    }
  }
```



## 基础知识



### 事件操作

- event 合成事件`React.MouseEvent<HTMLButtonElement, MouseEvent>`，events 相较于原生多了一些自定义的参数，访问原生可以 `e.nativeEvent`

- 事件委托到容器元素，也就是挂载的 root，和性能有关（可以深入研究的点🧐）
- 传参，可以使用高阶函数 / 箭头函数，一般推荐用箭头函数🤠

```tsx
import style from './assets/index.module.css'

function App() {
  function handleClick1(num: number) {
    return (e: React.MouseEvent) => {
      console.log(num)
      console.log(e)
    }
  }
  function handleClick2(e: React.MouseEvent, num: number) {
    console.log(num)
    console.log(e)
  }
  return (
    <>
      <div className={style.box}>
        <button onClick={handleClick1(111)}>111</button>
        <button onClick={e => handleClick2(e, 222)}>222</button>
      </div>
    </>
  )
}
```



### 条件渲染

- js 中给一个变量赋值，然后渲染
- `&&` `||` `? :`，注意0的时候不对劲😆



### 数组渲染

- for，while，但是麻烦
- map，推荐
- 循环需要加 key，有助于更新dom，用唯一值，不建议用 index值（面试考点，深入研究一下🧐）



### 组件的点标记写法

```tsx
import style from './assets/index.module.css'

// 对象写法
const Qf = {
  componentA() {
    return <div>componentA</div>
  },
}

// 函数写法
const Qf2 = () => {
  return <div>Qf2</div>
}

Qf2.componentA = () => {
  return <div>componentA</div>
}

function App() {
  return (
    <>
      <div className={style.box}>
        <Qf.componentA />
        <Qf2.componentA />
      </div>
    </>
  )
}
```



### 组件通信

- 父传子，props（略过🥱）
- 插槽的话直接`props.children`
- 通信添加默认值
  - 使用es6的解构添加默认值
  - `组件.defaultProps = { }`

- 通信限定类型（js的），`组件.propTypes = { }`，需要安装`prop-types`模块，个人感觉可以抛弃了，直接上ts吧🥸



### 组件必须是一个纯函数

- 只负责自己的任务，它不会更改在该函数调用前就已存在的对象或变量
- 输入相同，则输出相同。给定相同的输入，纯函数应总是返回相同的结果，使用严格模式检测不纯的计算（调用俩次函数）



### 状态 和 useState

> 2023.12.24 02:48 学到了p22
>
> https://www.bilibili.com/video/BV13h4y177jW/?p=22&share_source=copy_web&vd_source=fec74aa0dc6bc131c090122b391ab233

- 什么是状态？
  - 随时间变化的数据被称为状态（state），状态可以数据驱动视图，但是普通的变量不可以
  - `useState`是可以创建修改状态的方法
- 状态是如何改变视图的
  - 普通变量无法重新渲染jsx
  - state状态可以重新触发函数组件



​	
