---
title: Vue查漏补缺
tags:
  - Vue
  - 学习
categories: 前端
cover: >-
  https://static001.infoq.cn/resource/image/c4/01/c4251ee38c2039602d69cac1d3ab9d01.jpg
keywords: 学习 前端 Vue
abbrlink: 43040
date: 2023-05-22 21:29:01
---

# Vue 查漏补缺

## [v-bind](https://cn.vuejs.org/api/built-in-directives.html#v-bind)

### **绑定 class**

1. 对象方式

`  <h1 :class="{ red: isRed, black: !isRed }">index</h1>`

渲染出来就是`class="red"`，也可以在 js 里面声明对象，再添加

格式：`{active: boolean}`

2. 数组方式，用的更多

```vue
<template>
  <div class="container">
    <h1 :class="['red', title, { black: !isRed }]">index</h1>
  </div>
</template>

<script setup lang="ts">
import { ref, type Ref } from 'vue'
let color: Ref<string> = ref('black')
let isRed: Ref<boolean> = ref(true)
let title: Ref<string> = ref('title')
</script>
```

### 绑定 style

1. 对象方式

使用`{}`，要加引号`  <h1 :style="{ color: 变量, fontSize: '30px' }">index</h1>`，如果不用驼峰命名法，需要加上引号`'font-size': '30px'`

2. 数组方式

`  <h1 :class="[styleObj, {}]">index</h1>`

### 动态绑定属性

`:[name]="value"`

如果要渲染多个，直接`v-bind="info"`就可以

`<h1 v-bind="info">index</h1>`->`  <h1 name="t" age="18">index</h1>`

```ts
const info: Ref<object> = ref({
  name: 't',
  age: 18,
})
```

## [v-on](https://cn.vuejs.org/api/built-in-directives.html#v-on)

缩写：@

修饰符：

- `.stop` - 调用 `event.stopPropagation()`。
- `.prevent` - 调用 `event.preventDefault()`。
- `.capture` - 在捕获模式添加事件监听器。
- `.self` - 只有事件从元素本身发出才触发处理函数。
- `.{keyAlias}` - 只在某些按键下触发处理函数。
- `.once` - 最多触发一次处理函数。
- `.left` - 只在鼠标左键事件触发处理函数。
- `.right` - 只在鼠标右键事件触发处理函数。
- `.middle` - 只在鼠标中键事件触发处理函数。
- `.passive` - 通过 `{ passive: true }` 附加一个 DOM 事件。

可以绑定方法，表达式，对象

绑定对象：`v-on="{click: btnClick, mousemove: mouseMove}"`

### 参数传递

默认会把`event`传给函数，函数写不写都可以接受

```vue
<template>
  <div class="container">
    <h1>index</h1>
    <n-button @click="btnClick">换色</n-button>
  </div>
</template>

<script setup lang="ts">
import { NButton } from 'naive-ui'

function btnClick(e: Event) {
  console.log(e)
}
</script>
```

调用函数传参写`event`的时候要加`$`

```vue
<template>
  <div class="container">
    <h1>index</h1>
    <n-button @click="btnClick($event, 'name111')">换色</n-button>
  </div>
</template>

<script setup lang="ts">
import { NButton } from 'naive-ui'

function btnClick(e: Event, name: string) {
  console.log(e, name)
}
</script>
```

## v-for

可以使用 in 或者 of，第一个参数是`item`，第二个参数是`index`，可以遍历数组和对象，对象的话就是`value`和`key`以及`index`，小括号加不加都可以

```vue
<template>
  <ul>
    <li
      v-for="(list, index) of lists"
      :key="list"
    >
      {{ index }}.{{ list }}
    </li>
  </ul>
</template>

<script setup lang="ts">
import { ref, type Ref } from 'vue'
const lists: Ref<string[]> = ref(['海贼王', '火影忍者', '哔哩哔哩'])
</script>
```

也可以用来遍历数字，但是是从 1 开始的，`v-for= "i in 10"`就是 1-10，`v-for="(n,index) in 10"`，索引是 0-9

对于是否有 key，vue 调用的方法是不同的，这里可以深入源码读一读`packages\runtime-core\src\renderer.ts`

## computed

对比直接使用 function 或者 watch，有缓存，依赖的数据不变化就直接返回缓存

```ts
import { computed } from 'vue'
let a = ref(1)
let b = ref(2)
let c = computed(() => a.value + b.value)
```

写全了就可以有 get 和 set，默认就是用 get

```ts
let c = computed(() => {
  get: () => {}
  set: () => {}
})
```

## v-model

本质上就是一个语法糖来实现双向绑定

`<input v-model="text">`就是

`<input :value="text" @input="event => text = event.target.value">`
