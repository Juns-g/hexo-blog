---
title: Web Components
tags:
  - 前端
categories: 前端
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
abbrlink: 34909
date: 2024-10-08 23:01:33
---

# Web Components

> 完善中～～～

## 前言

在写 vue 还有 react 的时候，有时候也会好奇，原生三件套怎么实现组件化呢？因为觉得组件化是一个很重要的东西，可以把一个页面拆分成一个个小的模块，方便维护和复用，组件代码也提倡不要过长过耦合。对于原生三件套来说，组件化不解决的话，确实开发上是很大的问题。

查阅资料，以及结合之前微前端所了解的知识，发现有 Web Components 这个东西，他是一种标准，相当于是提供了原生的组件化能力，下面简要了解看看。

## 是什么

直接翻看[官方文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_components):

> Web Component 是一套不同的技术，允许你创建可重用的定制元素（它们的功能封装在你的代码之外）并且在你的 web 应用中使用它们。

和理解的一样，就是提供一个组件化能力，不过具体怎么用还得接着看文档。

## 如何实现组件化

文档后面介绍了他的三个主要技术，基于这几个技术实现组件功能。

> - Custom element（自定义元素）：一组 JavaScript API，允许你定义 custom elements 及其行为，然后可以在你的用户界面中按照需要使用它们。
> - Shadow DOM（影子 DOM）：一组 JavaScript API，用于将封装的“影子”DOM 树附加到元素（与主文档 DOM 分开呈现）并控制其关联的功能。通过这种方式，你可以保持元素的功能私有，这样它们就可以被脚本化和样式化，而不用担心与文档的其他部分发生冲突。
> - HTML template（HTML 模板）： `<template>` 和 `<slot>` 元素使你可以编写不在呈现页面中显示的标记模板。然后它们可以作为自定义元素结构的基础被多次重用。

Shadow DOM 可以保证这个组件不受外部影响，Custom element 可以自定义组件，HTML template 的话就是用来辅助写 html 的。

## 怎么用

### 基本使用

我们可以基于 Custom element 这个特性，实现自定义的标签，比如`<my-button/>`这样的非原生标签，然后在 js 中定义这个标签的行为。

```html
<script>
  class MyButton extends HTMLElement {
    constructor() {
      super()

      const shadow = this.attachShadow({ mode: 'open' })
      const button = document.createElement('button')
      button.innerText = '我是自定义按钮'
      shadow.appendChild(button)
    }
  }

  window.customElements.define('my-button', MyButton)
</script>

<h1>Temp</h1>
<my-button></my-button>
```

上面是一个简易的示例代码，不太规范，不过能够体现功能，我们只需要在其他地方使用`<my-button/>`这个标签，就可以看到一个自定义的组件了，可以拆分成多个文件出来，然后单独调用组件。

### 使用模板抽离成单 js 文件

除此之外，我们还可以使用 js 的模板字符串结合 template 标签，实现封装到一个 js 文件中：

```js
const template = document.createElement('template')
template.innerHTML = `
    <style>
        button {
        background-color: #007bff;
        color: white;
        padding: 10px 20px;
        border-radius: 5px;
        border: none;
        cursor: pointer;
    }
    </style>

    <button>我是自定义按钮</button>
`

class MyButton extends HTMLElement {
  constructor() {
    super()

    const content = template.content.cloneNode(true)
    const shadow = this.attachShadow({ mode: 'open' })
    shadow.appendChild(content)
  }
}

window.customElements.define('my-button', MyButton)
```

```html
<script src="./index.js"></script>

<h1>Temp</h1>
<my-button></my-button>
```

### 添加 attributes

组件是需要支持一些静态属性的，下面根据文档给我们自定义的 button 添加一些静态属性支持

```js
const template = document.createElement('template')
template.innerHTML = `
    <style>
        button {
        background-color: #007bff;
        color: white;
        padding: 10px 20px;
        border-radius: 5px;
        border: none;
        cursor: pointer;
    }
    </style>

    <button>我是自定义按钮</button>
`

class MyButton extends HTMLElement {
  constructor() {
    super()

    const content = template.content.cloneNode(true)
    const shadow = this.attachShadow({ mode: 'open' })

    const buttonText = this.getAttribute('text') || '我是自定义按钮'
    content.querySelector('button').innerHTML = buttonText

    shadow.appendChild(content)
  }
}

setTimeout(() => {
  window.customElements.define('my-button', MyButton)
}, 100)
```

```html
<script src="./index.js"></script>

<my-button text="自定义内容"></my-button>
<my-button text="自定义"></my-button>
```

这样我们就实现了静态属性的支持，可以根据不同的属性，自己判断显示内容。不过这个还是有待完善的，当我们在外部改变 text 属性的时候，里面的内容没有更新，因为我们只在初始化的时候获取了一次属性，所以我们需要添加一个监听属性变化的方法。

```js
const template = document.createElement('template')
template.innerHTML = `
    <style>
        button {
        background-color: #007bff;
        color: white;
        padding: 10px 20px;
        border-radius: 5px;
        border: none;
        cursor: pointer;
    }
    </style>

    <button>我是自定义按钮</button>
`

class MyButton extends HTMLElement {
  // 俩种写法都可以
  static observedAttributes = ['text']
  /* static get observedAttributes() {
    return ['text']
  } */
  constructor() {
    super()

    const content = template.content.cloneNode(true)
    const shadow = this.attachShadow({ mode: 'open' })
    shadow.appendChild(content)
    this.renderText()
  }
  renderText() {
    const buttonText = this.getAttribute('text') || '我是自定义按钮'
    this.shadowRoot.querySelector('button').innerHTML = buttonText
  }
  attributeChangedCallback(name, oldValue, newValue) {
    this.renderText()
  }
}

setTimeout(() => {
  window.customElements.define('my-button', MyButton)
}, 100)
```

我们可以使用上面的 api 来实现监听属性变化的功能，这样当我们在外部改变 text 属性的时候，里面的内容也会随之改变。

### 生命周期

简单分析代码可以知道，我们使用一个类来表示一个组件，这就有点像 react 的类组件了，那么理论上也是有一些生命周期的概念的。

> 自定义元素生命周期回调包括：
>
> - connectedCallback()：每当元素添加到文档中时调用。规范建议开发人员尽可能在此回调中实现自定义元素的设定，而不是在构造函数中实现。
> - disconnectedCallback()：每当元素从文档中移除时调用。
> - adoptedCallback()：每当元素被移动到新文档中时调用。
> - attributeChangedCallback()：在属性更改、添加、移除或替换时调用。有关此回调的更多详细信息，请参见响应属性变化。

由此可见，我们上面其实不太规范，我们把渲染操作放到了 constructor 中，规范应该是放到 connectedCallback 中，这样可以保证在元素添加到文档中时才会渲染。

除此之外，我们也可以利用生命周期方法，做一些其他的操作。

## 实践

下面尝试根据上面的知识，实现一个简单的按钮组件，并在 solid 组件中使用它。

> 代码仓库地址: [https://github.com/Juns-g/solid_web_component](https://github.com/Juns-g/solid_web_component)

```ts
// MyButton.ts, web component
const COLOR_MAP = {
  primary: '#007bff',
  danger: 'red',
}

const template = document.createElement('template')
template.innerHTML = `
    <style>
        button {
            color: white;
            background-color: var(--button-color,${COLOR_MAP.primary});
            padding: 10px 20px;
            border-radius: 5px;
            border: none;
            cursor: pointer;
    }
    </style>

    <button>默认文案</button>
`

class MyButton extends HTMLElement {
  static observedAttributes = ['text', 'color']

  constructor() {
    super()

    const content = template.content.cloneNode(true)
    const shadow = this.attachShadow({ mode: 'open' })
    shadow.appendChild(content)
  }

  connectedCallback() {
    console.log('挂载')
    this.render()
  }

  render() {
    const btn = this.shadowRoot!.querySelector('button') as HTMLButtonElement

    // text
    const buttonText = this.getAttribute('text') || '我是自定义按钮'
    btn!.innerHTML = buttonText

    // color
    const color = (this.getAttribute('color') ||
      'primary') as keyof typeof COLOR_MAP
    btn.style.setProperty('--button-color', COLOR_MAP[color])

    // handleClick
    btn.onclick = () => {
      this.dispatchEvent(new Event('handleClick'))
    }
  }

  attributeChangedCallback(name: string, oldValue: string, newValue: string) {
    console.log('attribute change', { name, oldValue, newValue })
    this.render()
  }
}

setTimeout(() => {
  window.customElements.define('my-button', MyButton)
}, 100)
```

```tsx
import { createEffect, createSignal } from 'solid-js'

export const Buttons = () => {
  let buttonsRef: HTMLDivElement | undefined
  const [btnText, setBtnText] = createSignal('我是按钮')
  createEffect(() => {
    const btn = buttonsRef!.querySelector('my-button') as HTMLElement
    btn.setAttribute('text', btnText())
  })
  return (
    <div
      class='flex items-center justify-center gap-3'
      ref={buttonsRef}
    >
      <my-button
        on:handleClick={(event: MouseEvent) => {
          console.log('点击了按钮', event)
          setBtnText('我被点击了')
        }}
      />
      <my-button
        text='danger按钮'
        color='danger'
      />
    </div>
  )
}
```

这只是一个十分简单的实践，具体情况还可以优化很多地方。

## 拓展

### Lit

[Lit](https://lit.dev/docs/)是 Google 开发的一个用于构建快速，轻量级 Web 组件的简单库。他在原生的基础上提供了一些状态相关的逻辑，以及更好的模板语法支持，下面是他官网的一个例子。

```ts
import { LitElement, html, css } from 'lit'
import { customElement, property, state } from 'lit/decorators.js'

@customElement('my-timer')
export class MyTimer extends LitElement {
  static styles = css`...`

  @property() duration = 60
  @state() private end: number | null = null
  @state() private remaining = 0

  render() {
    const { remaining, running } = this
    const min = Math.floor(remaining / 60000)
    const sec = pad(min, Math.floor((remaining / 1000) % 60))
    const hun = pad(true, Math.floor((remaining % 1000) / 10))
    return html`
      ${min ? `${min}:${sec}` : `${sec}.${hun}`}
      <footer>
        ${remaining === 0
          ? ''
          : running
          ? html`<span @click=${this.pause}>${pause}</span>`
          : html`<span @click=${this.start}>${play}</span>`}
        <span @click=${this.reset}>${replay}</span>
      </footer>
    `
  }

  start() {
    this.end = Date.now() + this.remaining
    this.tick()
  }

  pause() {
    this.end = null
  }

  reset() {
    const running = this.running
    this.remaining = this.duration * 1000
    this.end = running ? Date.now() + this.remaining : null
  }

  tick() {
    if (this.running) {
      this.remaining = Math.max(0, this.end! - Date.now())
      requestAnimationFrame(() => this.tick())
    }
  }

  get running() {
    return this.end && this.remaining
  }

  connectedCallback() {
    super.connectedCallback()
    this.reset()
  }
}

function pad(pad: unknown, val: number) {
  return pad ? String(val).padStart(2, '0') : val
}
```

### Omi

官网地址: [https://github.com/Tencent/omi/blob/master/README.CN.md](https://github.com/Tencent/omi/blob/master/README.CN.md)

> - 📶 信号 Signal 驱动的响应式编程，reactive-signal 强力驱动
> - 🧱 TDesign Web 组件
> - ⚡ 微小的尺寸，极速的性能
> - 💗 目标 100+ 模板 & OMI 模板源码
> - 🐲 OMI Form & OMI Form 游乐场 & Lucide Omi 图标
> - 🌐 你要的一切都有: Web Components, JSX, Function Components, Router, Suspense, Directive, > - Tailwindcss...
> - 💒 使用 Constructable Stylesheets 轻松管理和共享样式

示例代码：

```ts
import { render, signal, tag, Component, h } from 'omi'

const count = signal(0)

function add() {
  count.value++
}

function sub() {
  count.value--
}

@tag('counter-demo')
export class CounterDemo extends Component {
  static css = 'span { color: red; }'

  render() {
    return (
      <>
        <button onClick={sub}>-</button>
        <span>{count.value}</span>
        <button onClick={add}>+</button>
      </>
    )
  }
}
```
