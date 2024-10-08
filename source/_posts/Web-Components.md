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

## 生命周期

简单分析代码可以知道，我们使用一个类来表示一个组件，这就有点像 react 的类组件了，那么理论上也是有一些生命周期的概念的。

> 自定义元素生命周期回调包括：
>
> - connectedCallback()：每当元素添加到文档中时调用。规范建议开发人员尽可能在此回调中实现自定义元素的设定，而不是在构造函数中实现。
> - disconnectedCallback()：每当元素从文档中移除时调用。
> - adoptedCallback()：每当元素被移动到新文档中时调用。
> - attributeChangedCallback()：在属性更改、添加、移除或替换时调用。有关此回调的更多详细信息，请参见响应属性变化。
