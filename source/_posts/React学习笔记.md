---
title: React学习笔记.md
tags:
  - React
  - 学习
  - 前端
categories: 前端
cover: 'https://jslib.dev/wp-content/uploads/2022/03/Reactlogo.jpg'
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



### 状态 和 [useState](https://react.dev/reference/react/useState)

> 2023.12.24 02:48 学到了p22
>
> https://www.bilibili.com/video/BV13h4y177jW/?p=22&share_source=copy_web&vd_source=fec74aa0dc6bc131c090122b391ab233

- 什么是状态？
  - 随时间变化的数据被称为状态（state），状态可以数据驱动视图，但是普通的变量不可以
  - `useState`是可以创建修改状态的方法

- 状态是如何改变视图的
  - 普通变量无法重新渲染jsx
  - state状态可以重新触发函数组件

- 多状态是如何正确记忆的
  - 在同一个组件的每次渲染中，`useState`都依托于一个稳定的调用顺序
  - 所以不要在逻辑中调用`useState`，会改变内部的顺序

- 什么是状态的快照以及快照的陷阱

  - 原理是函数的闭包（待深入）
  - `state`变量看着像js的变量，但是实际特性更像是一个快照。

- 状态队列和批处理

  - 等待所有代码都运行完毕之后再处理`state`更新。队列都执行完毕后，再进行UI更新

  - 更新的函数写法，每次拿到的就都是更新之后的值了

    - ```ts
      const [count, setCount] = useState(0)
      setCount(count + 1) // 0 + 1
      setCount(count + 1) // 0 + 1
      setCount((c) => c + 1) // 0 + 1
      setCount((c) => c + 1) // 1 + 1

  - 其实`setState(x)`实际会像`setState(n => x)`一样运行，`setCount(count + 1)`其实就是`setCount(c => count + 1)`，没有调用入参而已

- 状态是不可变的

  - 当修改状态的值没有改变的时候，函数组件不会重新渲染
  - 所以不要去直接修改状态，而是通过set方法去修改

- 对象/数组的解决方案

  - 避免使用会修改原数组的方法，推荐使用返回新值的方法，或者直接拷贝（深拷贝面试要点！）
  - 深拷贝会带来性能问题，可以引入`immer`模块来减少性能消耗

- 惰性初始化值，`useState`可以传入一个函数，但是会每次都触发，有性能问题，所以可以直接写一个匿名函数来解决这个问题

  - ```tsx
    import { useState } from 'react'
    
    function computed(n: number) {
      console.log('computed函数被调用了') // 每次点击都会触发
      return n + 3
    }
    
    function App() {
      const [count, setCount] = useState(computed(0))
      const [count, setConut] = useState(() => computed(0)) // 只有初始化的时候会调用一次
      const handleClick = () => {
        setCount(count + 1)
      }
      return (
          <button onClick={handleClick}>{count}</button>
      )
    }
    ```

- 状态提升来解决共享问题

  - 就是把子组件的状态提升到父组件中，通过`props`传递给子组件

- 状态的重置处理问题

  - 组件被销毁时，状态会被重置

  - 当组件位置没有发生变化时，状态会被保留

  - 结构不同或者添加了key，状态会重置

  - ```tsx
    {/* 组件的状态会被保留 */ }
    { isStyle ? <Counter style={{ border: '1px red solid' }} /> : <Counter /> }
    {/* 结构不同时会重置状态 */ }
    { isStyle ? <Counter style={{ border: '1px red solid' }} /> : <div><Counter /></div> }
    {/* 添加key也会重置状态 */ }
    { isStyle ? <Counter style={{ border: '1px red solid' }} key='counter1' /> : <Counter key='counter2' /> }
    ```

- 利用状态产生计算变量

  - 类似`vue`中的`computed`

  - 因为重新渲染组件的时候拿到的状态快照都是新的，所以直接用普通变量就可以了

  - ```tsx
    function App() {
      const [count, setCount] = useState(0)
      const doubleCount = count * 2
      return (
          <div className={style.box}>
            <button onClick={() => setCount(count + 1)}>count: {count}</button>
            <button>doubleCount: {doubleCount}</button>
          </div>
      )
    }
    ```

实战：写一个`todoList`组件

```tsx
import { useMemo, useState } from 'react'
import styles from './index.module.css'

interface ListItem {
  id: string
  content: string
  isDone: boolean
}

interface ChildProps {
  isDone: boolean
  list: ListItem[]
  handleChangeStatus: (id: string) => void
}

function Child({ isDone, list, handleChangeStatus }: ChildProps) {
  return (
    <div className={styles['flex-box']}>
      <h2>
        {isDone ? '已' : '未'}完成的任务: {list.length}个
      </h2>
      <ul>
        {list.map(item => (
          <li key={item.id}>
            <input
              type='checkbox'
              checked={item.isDone}
              onChange={() => handleChangeStatus(item.id)}
            />
            <span className={isDone ? styles['done-item'] : ''}>
              {item.content}
            </span>
          </li>
        ))}
      </ul>
    </div>
  )
}

function TodoList() {
  console.log('渲染一次')
  const [msg, setMsg] = useState('')
  const [list, setList] = useState<ListItem[]>([])

  function handleInputChange(e: React.ChangeEvent<HTMLInputElement>) {
    setMsg(e.target.value)
  }

  function handleAdd() {
    if (!msg.trim()) return
    setList([
      ...list,
      {
        id: Date.now().toString(),
        content: msg,
        isDone: false,
      },
    ])
    setMsg('')
  }

  const unDoneList = useMemo(() => list.filter(item => !item.isDone), [list])
  const doneList = useMemo(() => list.filter(item => item.isDone), [list])

  function handleChangeStatus(id: string) {
    setList(prevList => {
      const newList = prevList.map(item => {
        if (item.id !== id) return item
        return {
          ...item,
          isDone: !item.isDone,
        }
      })
      return newList
    })
  }

  return (
    <div className={styles.container}>
      <div className={styles['flex-box']}>
        <div>
          <input
            className={styles.input}
            value={msg}
            onChange={handleInputChange}
          />
          <button onClick={handleAdd}>添加任务</button>
        </div>
      </div>
      <Child
        list={unDoneList}
        isDone={false}
        handleChangeStatus={handleChangeStatus}
      />
      <Child
        list={doneList}
        isDone
        handleChangeStatus={handleChangeStatus}
      />
    </div>
  )
}

export default TodoList
```



## Hooks进阶

### 什么是hooks

- 在`Reac`t 中，`useState` 以及任何其他以"use〞开头的函数都被称为 `Hook`（即钩子），所以`Hooks`就是代表着use函数的集合，也就是钩子的集合
- `Hooks`其实就是一堆功能函数，一个组件想要实现哪些功能就可以引入对应的钩子函数，像插件一样非常的方便

### ref 和 [useRef](https://react.dev/reference/react/useRef)

`useRef(initialValue)`返回的是`{ current: initialValue }`

他是可以改变的，改变时也不会触发重新渲染

用法：

#### 用`ref`引用一个值做记忆功能

```tsx
import { useRef, useState } from 'react'

function App() {
  const [count, setCount] = useState(0)
  const num = useRef(0)
  function handleClick() {
    setCount(count + 1)
    num.current++
    console.log('🚀 ~ num:', num)
  }
  return <button onClick={handleClick}>count: {count}</button>
}
```

和`useState`的区别

|                           **ref**                            |                          **state**                           |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|  `useRef(initialValue)`返回的是`{ current: initialValue }`   | `useState(initialValue)`返回的是`state`变量的当前值和一个`state`设置函数 |
|                     更改时不触发重新渲染                     |                      更改时触发重新渲染                      |
|    可变 —— 可以在渲染过程之外修改和更新`ref.current`的值     | 不可变 —— 必须用提供的舍之函数来修改`state`变量，然后排队重新渲染 |
| 在渲染过程中不要写入或读取 `ref.current` ，初始化除外。这使得组件的行为不可预测。 |    可以随时读取`state`。但是每次渲染都有不变的`state`快照    |

使用场景：定时器的清除

```tsx
// 这样会导致多个定时器同时运行，无法正确清除
const [count, setCount] = useState(0)
let timer: number | null = null
function handleClick() {
  setCount(count + 1)
  if(timer){
    clearInterval(timer)
  }
  timer = setInterval(() => {
    console.log(111)
  }, 1000)
}
```



```tsx
const [count, setCount] = useState(0)
const timer = useRef<number | null>(null)
function handleClick() {
  setCount(count + 1)
  if (timer.current) {
    clearInterval(timer.current)
  }
  timer.current = setInterval(() => {
    console.log(111)
  }, 1000)
}
```

#### 通过`ref`操作`DOM`

```tsx
function App() {
  const divRef = useRef<HTMLDivElement>(null)
  function handleClick() {
    if (!divRef.current) return
    console.log(divRef.current.innerHTML)
    // 通过ref操作原生DOM
    divRef.current.style.background = 'red'
  }
  return (
    <>
      <div className={style.box}>
        <button onClick={handleClick}>click</button>
        <div ref={divRef}>112131</div>
      </div>
    </>
  )
}
```

在逻辑中通过`ref`操控`DOM`

```tsx
function App() {
  const list = [
    { id: 11, text: '234' },
    { id: 131, text: '23432' },
    { id: 141, text: '23224' },
  ]
  return (
    <ul>
      {list.map(item => (
        <li
          key={item.id}
          ref={myRef => {
            console.log(myRef)
            myRef.style.backgroundColor = 'red'
          }}
        >
          {item.text}
        </li>
      ))}
    </ul>
  )
}
```



#### 使用`fowWardRef`转发`ref`

当组件添加`ref`属性的时候，需要`forwardRef`进行转发，`forwardRef`让组件通过` ref` 向父组件公开DOM 节点

```tsx
const MyInput = forwardRef<HTMLInputElement>((_, inputRef) => {
  return <input ref={inputRef} />
})

function App() {
  const inputRef = useRef<HTMLInputElement>(null)
  function handleClick() {
    if (!inputRef.current) return
    inputRef.current.focus()
    inputRef.current.style.backgroundColor = 'red'
  }
  return (
    <>
      <button onClick={handleClick}>click</button>
      <MyInput ref={inputRef} />
    </>
  )
}
```



#### [useImperativeHandle](https://react.dev/reference/react/useImperativeHandle) 自定义`ref`的暴露

其实就类似`vue`的`defineExpose`

```tsx 
interface InputRef {
  focus: () => void
}

const MyInput = forwardRef<InputRef>((_, inputRef) => {
  const innerRef = useRef<HTMLInputElement>(null)
  useImperativeHandle(inputRef, () => ({
    focus() {
      innerRef.current?.focus()
    },
  }))
  return <input ref={innerRef} />
})

function App() {
  const inputRef = useRef<InputRef>(null)
  function handleClick() {
    if (!inputRef.current) return
    inputRef.current.focus()
  }
  return (
    <>
      <button onClick={handleClick}>click</button>
      <MyInput ref={inputRef} />
    </>
  )
}
```



### 副作用 和 [useEffect](https://react.dev/reference/react/useEffect)

- 纯函数的概念
  - 只负责自己的任务，它不会更改在该函数调用前就己存在的对象或变
  - 输入相同，则输出相同。给定相同的输入，纯函数应总是返回相同的结果
- 副作用的概念
  - 函数在执行过程中对外部造成的影响称之为副作用，例如：Ajax调用，DOM操作，与外部系统同步等
  - 在React组件中，事件操作是可以处理副作用的，但有时候需要初始化处理副作用，那么就需要useEffec钩子

#### 基本使用

```tsx
function App() {
  const inputRef = useRef<HTMLInputElement>(null)

  // 可以在初始的时候进行副作用操作
  // useEffect触发的时机：JSX渲染后触发的
  // 初始渲染和更新渲染，都会触发
  useEffect(() => {
    if (!inputRef.current) return
    inputRef.current.focus()
  })

  return (
    <>
      <button>click</button>
      <input ref={inputRef} />
    </>
  )
}
```

可以指定依赖项，初始的时候所有的`useEffect`都会触发，之后只有依赖更新的时候才会触发，内部其实是通过`Object.is()`来判断是否改变的

当依赖项是空数组的时候，只会初始触发，更新不触发,**无依赖项是每次更新触发**

```tsx
function App() {
  const [count, setCount] = useState(0)
  const [msg] = useState('hello')
  useEffect(() => {
    console.log(count)
  }, [count])
  useEffect(() => {
    console.log(msg)
  }, [msg])

  return (
    <>
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        {count}++
      </button>
      <button>{msg}</button>
    </>
  )
}
```

#### 尽量在`useEffect`内部定义函数

因为对于依赖是否改变是调用的`Object.is()`，但是俩个函数的内存地址是不一样的`Object.is(function(){}, function(){}) === false`，所以说定义在外部的函数作为依赖项，会导致每次都刷新。也可以用`useCallback`来解决。

```tsx
const [count, setCount] = useState(0)

const logCount = useCallback(() => {
  console.log(count)
}, [count])

useEffect(() => {
  logCount()
}, [logCount])
// 但是这样过其实是很麻烦的
```

最好的解决办法就是把函数定义在`useEffect`内部

```tsx
const [count, setCount] = useState(0)

useEffect(() => {
  const logCount = () => {
    console.log(count)
  }
  logCount()
}, [count])
```



#### 清理操作的重要性

```tsx
import { useEffect, useState } from 'react'

function Child({ title }: { title: string }) {
  useEffect(() => {
    console.log('子组件挂载', title)
    // useEffect 的清理工作
    // 卸载当前组件的时候，会执行清理工作
    return () => {
      console.log('子组件卸载', title)
    }
  }, [title])
  return <p>我是子组件: {title}</p>
}

function App() {
  const [isShow, setIsShow] = useState(true)
  const [title, setTitle] = useState('聊天室1')

  const handleChange = (e: any) => {
    setTitle(e.target.value)
  }

  return (
    <>
      父组件!!!!!!
      <select
        onChange={handleChange}
        value={title}
      >
        <option value='聊天室1'>聊天室1</option>
        <option value='聊天室2'>聊天室2</option>
      </select>
      <button
        onClick={() => {
          setIsShow(!isShow)
        }}
      >
        卸载子组件
      </button>
      {isShow && <Child title={title} />}
    </>
  )
}
```

初始化数据时，要注意清理操作，所以更简洁的方式是使用第三方的hook



#### 实验性的[useEffectEvent](https://react.dev/reference/react/experimental_useEffectEvent)

可以将非响应式逻辑提取到效果事件中

`const onSomething = useEffectEvent(callback)`



#### [useLayoutEffect](https://react.dev/reference/react/useLayoutEffect) 同步执行状态更新

`useEffect`是在渲染被绘制到屏幕之后执行的，是异步的

`useLayoutEffect`是在渲染之后但在屏幕更新之前，是同步的

大部分情况下我们采用`useEffect`，性能更好。

但当你的`useEffect`里面的操作需要处理DOM，并且会改变页面的样式，就需要用`useLayoutEffect`，否则可能会出现闪屏问题



#### [useInsertionEffect](https://react.dev/reference/react/useInsertionEffect) DOM更新前触发

应用场景非常少，因为获取不到DOM元素，所以只在CSS-in-JS库中才会使用



### [useReducer](https://react.dev/reference/react/useReducer)统一的状态管理集合

是处理状态的另一种方式，可以把更改状态的逻辑集合起来

```tsx
import { useReducer } from 'react'

interface ListItem {
  id: number
  text: string
}

type ListState = ListItem[]

interface ListAction {
  type: string
  id?: number
}

function listReducer(state: ListState, action: ListAction): ListState {
  switch (action.type) {
    case 'add':
      return [...state, { id: Math.random(), text: 'new!!!' }]
    case 'edit':
      return state.map(item => {
        if (item.id !== action.id) return item
        return {
          ...item,
          text: 'edit!!!' + item.text,
        }
      })
    case 'delete':
      return state.filter(item => item.id !== action.id)
    default:
      return state
  }
}
function App() {
  const [list, listDispatch] = useReducer(listReducer, [
    { id: 1, text: '213' },
    { id: 13, text: '2143' },
    { id: 11, text: '2413' },
  ])
  return (
    <div className={style.box}>
      <button
        onClick={() => {
          listDispatch({ type: 'add' })
        }}
      >
        add
      </button>
      <ul>
        {list?.map(item => {
          return (
            <li key={item.id}>
              {item.text}
              <button
                onClick={() => {
                  listDispatch({ type: 'edit', id: item.id })
                }}
              >
                edit
              </button>
              <button
                onClick={() => {
                  listDispatch({ type: 'delete', id: item.id })
                }}
              >
                delete
              </button>
            </li>
          )
        })}
      </ul>
    </div>
  )
}
```



### [useContext](https://react.dev/reference/react/useContext)

就类似`vue`的`provide`和`inject`

```tsx
import { createContext, useContext } from 'react'

const ThemeContext = createContext(null)

function App() {
  return (
    <ThemeContext.Provider value='dark'>
      <Form />
    </ThemeContext.Provider>
  )
}

function Form() {
  return (
    <Panel title='Welcome'>
      <button>Sign up</button>
    </Panel>
  )
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext)
  const className = 'panel-' + theme
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}
```

简单的状态管理可以 `Reducer`+`Context`，复杂一点的可以用第三方库



### [Memo](https://react.dev/reference/react/memo)

在组件的属性保持不变时跳过重新渲染组件

```tsx
const MemoizedComponent = memo(SomeComponent, arePropsEqual?)
```



### [useMemo](https://react.dev/reference/react/useMemo)

在重新渲染之间缓存计算结果，类似于vue中的computed，是一个响应式变量

```tsx
const cachedValue = useMemo(calculateValue, dependencies)
```

因为是否重新渲染根据的是`Object.is()`，所以当渲染依据的值是数组的时候，每次渲染都会重新创建变量，内存地址不一样，则结果不同，所以回造成子组件的重新渲染

```tsx
import { memo, useMemo, useState } from 'react'

const Child = memo(({ title }: { title: string[] }) => {
  return (
    <div>
      {title + ''}
      <br />
      {Math.random()}
    </div>
  )
})

function App() {
  const [count, setCount] = useState(0)
  const [msg] = useState('hello world')
  const title = useMemo(() => [msg.toLowerCase(), msg.toUpperCase()], [msg])
  return (
    <>
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        {count}++
      </button>
      <Child title={title} />
    </>
  )
}
```



### [useCallback](https://react.dev/reference/react/useCallback) 缓存函数

> 2024.1.12  https://www.bilibili.com/video/BV13h4y177jW/?p=51&spm_id_from=pageDriver&vd_source=9928f2a9263e4b4f8d78f76eb5a79a03
