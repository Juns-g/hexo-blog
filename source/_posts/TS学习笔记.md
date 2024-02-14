---
title: TS学习笔记
tags:
  - JS
  - TS
  - 学习
cover: >-
  https://dogefs.s3.ladydaily.com/~/source/wallhaven/full/ex/wallhaven-ex3m8k.jpg?w=2560&h=1440&fmt=webp
keywords: 学习 前端 TS
categories: 前端
abbrlink: 51463
date: 2023-05-18 20:03:24
---

# TS 学习笔记

> 目前是一些很基本的东西，过段时间学进阶用法

## 数组类型

两种声明方式

- `Array<number>`
- `number[]`

使用示例：

```ts
const arr1: Array<number> = [1, 3, 4]
const arr2: number[] = [1, 4, 5]

function getSum(...args: number[]) {
  return args.reduce((prev, cur) => prev + cur, 0)
}
console.log(getSum(1, 2, 4, 5))
```

## 元组

```ts
const tuple: [string, number] = ['ace', 14]
const [name, age] = tuple
```

## 枚举

不指定初始值就从 0 开始累加，指定了就从第一个值开始累加

```ts
const enum PostStatus {
  // 不用const的话会入侵代码
  Draft = 0,
  Unpublished = 1,
  Published = 2,
}

const post = {
  title: '1212',
  content: '1212',
  status: PostStatus.Draft,
}
```

## 函数

对于可选值，可以使用 es6 的方法或者`?=`；返回值一般可以自动推断；如果是回调函数可以使用箭头的方式

`sum: (a: number, b: number = 10, c?: number, ...rest: number[]) => number`

```ts
function sum(a: number, b: number = 10, c?: number, ...rest: number[]): number {
  return a + b
}

console.log(sum(1, 3))
```

## 类型断言

有的时候自动推断出来的类型可能是`number|undefined`，可以自己断言确定变量的类型

- 不推荐的方式，因为在 tsx 中会冲突`const num = <number>res`
- 推荐`const num = res as number`

- `const res = nums.find((i) => i > 0) as number;`
- 非空断言，直接加一个`!`：`const res = nums.find((i) => i > 0)!;`

## 接口 interface

约束对象的结构

```ts
interface Post {
  // , 和 ; 都可以，也可以不加
  title: string
  content: string
}

function printPost(post: Post) {
  console.log(post.title)
  console.log(post.content)
}

printPost({
  title: 'sss',
  content: '123',
})
```

可选成员与只读成员：

```ts
interface Post {
  // , 和 ; 都可以，也可以不加
  title: string
  content: string
  subtitle?: string //可选成员,相当于string | undefined
  readonly summary: string // 只读成员，不可以修改
}

const test: Post = {
  title: 'sss',
  content: '123',
  summary: 'summary',
}
```

动态成员，可以动态添加属性，但是要满足类型的要求

```ts
interface Cache {
  [prop: string]: string
}

const cache: Cache = {}
cache.foo = '22'
```

## 类

可以用成员访问修饰符

```ts
class Person {
  name: string // 默认不加的为public
  private age: number //私有成员，外部无法访问
  protected gender: boolean // 外部无法访问,子类可以访问
  constructor(name: string, age: number) {
    this.name = name
    this.age = age
    this.gender = true
  }
  sayHi(msg: string): void {
    console.log(`i am ${this.name}, ${msg}`)
  }
}

class Student extends Person {
  constructor(name: string, age: number) {
    super(name, age)
    console.log(this.gender)
  }
}

let tom = new Student('tom', 15)
console.log(tom)
```

## 类与接口

类里面类型相似的可以抽象成接口

```ts
interface Eat {
  eat(food: string): void
}

interface Run {
  run(distance: number): void
}

class Person implements Eat, Run {
  eat(food: string): void {
    console.log(`优雅的进餐${food}`)
  }
  run(distance: number): void {
    console.log(`直立行走${distance}`)
  }
}
```

## 抽象类

只能被继承，不能被 new

```ts
abstract class Animal {
  eat(food: string): void {
    console.log(`呼噜呼噜的吃：${food}`)
  }
  abstract run(distance: number): void
}

class Dog extends Animal {
  run(distance: number): void {
    console.log('111', distance)
  }
}

const d = new Dog()
d.eat('11')
d.run(10)
```

## 泛型

调用的时候再指定类型

```ts
function createArray<T>(length: number, value: T): T[] {
  const arr = Array<T>(length).fill(value)
  return arr
}

const res = createArray<string>(3, 'test')
console.log('res:', res)
```

## 类型声明

```ts
import { camelCase } from 'lodash'

declare function camelCase(str: string): string

console.log(camelCase('test_aaa'))
```
