---
title: JS中的==运算符
tags:
  - JS
  - 学习
cover: 'https://pic.imgdb.cn/item/65cdd6cf9f345e8d032f5627.jpg'
keywords: 学习 前端 JS
categories: 前端
abbrlink: 53257
date: 2024-02-16 13:05:59
---

# JS 中的`==`运算符

😡 学 JS 的肯定被`==`运算符折磨过，他会发生隐式类型转换，而且里面的逻辑也比较复杂，如果面试一问，感觉直接寄 😫。所以这里来看看他到底是个啥机制，至少要能说出来些啥，后面也可以来引出`===`、`Object.is()`还有[SameValueZero](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Equality_comparisons_and_sameness#%E9%9B%B6%E5%80%BC%E7%9B%B8%E7%AD%89) (零值相等)算法
探讨 JS 中的相等机制。

## 宽松相等算法

JS 的行为遵循 ES 语言说明书，其中这个`==`的底层原理定义在[IsLooselyEqual(x, y)](https://262.ecma-international.org/#sec-islooselyequal)，这个也叫宽松相等算法。

这里直接翻译一下他的定义：

1. `Type(x) === Type(y)`，`return x === y`。
2. `x === null && y === undefined`，true。
3. `x === undefined && y === null`，true。
4. (兼容一些非 ES 语言的对象)
   1. 若 x 是 `Object` 类型且 x 有一个 `[[IsHTMLDDA]]` 内部插槽，且 `y === undefined || y === null`，true。
   2. 若 `x === undefined || x === null`，且 y 是 `Object` 类型，且 y 有一个 `[[IsHTMLDDA]]` 内部插槽，true。
5. `Type(x) === Number && Type(y) === String`，return IsLooselyEqual(x, ToNumber(y))。
6. `Type(x) === String && Type(y) === Number`，return IsLooselyEqual(ToNumber(x), y)。
7. `Type(x) === BigInt && Type(y) === String`
   1. `let n = StringToBigInt(y)`
   2. `n === undefined`，return false。
   3. `return IsLooselyEqual(x, n)`
8. `Type(x) === String && Type(y) === BigInt`, `return IsLooselyEqual(y, x)`
9. `Type(x) === Boolean`，return IsLooselyEqual(ToNumber(x), y)。
10. 同上，xy 互换。
11. x 是`String || Number || BigInt || Symbol`，y 是`Object`，`return IsLooselyEqual(x, ToPrimitive(y))`
12. 同上，xy 互换。
13. 若 x 是 `BigInt` 类型且 y 是 `Number` 类型，或者 x 是 `Number` 类型且 y 是 `BigInt` 类型，则
    a. 若 x 不是有穷数（not finite）或者 y 不是有穷数，则 return false。
    b. 若 `ℝ(x) = ℝ(y)`，则 return true，否则 return false。
14. false。

## 手写

下面直接手写实现来看看

```js
const isUndefined = $ => typeof $ === 'undefined'
const isNumber = $ => typeof $ === 'number'
const isString = $ => typeof $ === 'string'
const isBoolean = $ => typeof $ === 'boolean'
const isObject = $ => typeof $ === 'object' && $ !== null
const isSymbol = $ => typeof $ === 'symbol'
const isBigInt = $ => typeof $ === 'bigint'
const isNull = $ => $ === null

export const isLooselyEqual = (x, y) => {
  // 类型相同转 ===
  if (typeof x === typeof y) return x === y

  // undefined == null = true
  if (isNull(x) && isUndefined(y)) return true
  if (isUndefined(x) && isNull(y)) return true

  // 特殊对象，document.all == null = true
  if (isUndefined(x) && x instanceof Object && (isUndefined(y) || isNull(y)))
    return true
  if (isUndefined(y) && y instanceof Object && (isUndefined(x) || isNull(x)))
    return true

  // '1' == 1 = true
  if (isNumber(x) && isString(y)) return isLooselyEqual(x, Number(y))
  if (isString(x) && isNumber(y)) return isLooselyEqual(Number(x), y)

  // 9n == '9' -> 9n === BigInt('9')
  if (isBigInt(x) && isString(y)) {
    let n
    try {
      n = BigInt(y)
    } catch {
      return false
    }
    return isLooselyEqual(x, n)
  }
  if (isString(x) && isBigInt(y)) return isLooselyEqual(y, x)

  // [] == false -> [] == +false
  if (isBoolean(x)) return isLooselyEqual(+x, y)
  if (isBoolean(y)) return isLooselyEqual(x, +y)

  // 原始值与对象比较
  if (
    isObject(x) &&
    (isNumber(y) || isString(y) || isBoolean(y) || isSymbol(y) || isBigInt(y))
  )
    return isLooselyEqual(x.toString(), y)
  if (
    isObject(y) &&
    (isNumber(x) || isString(x) || isBoolean(x) || isSymbol(x) || isBigInt(x))
  )
    return isLooselyEqual(x, y.toString())

  // 9n == 9 -> Number(9) === 9
  if ((isBigInt(x) && isNumber(y)) || isNumber(x) & isBigInt(y)) {
    return (
      x !== Number.POSITIVE_INFINITY &&
      x !== Number.NEGATIVE_INFINITY &&
      y !== Number.POSITIVE_INFINITY &&
      y !== Number.NEGATIVE_INFINITY &&
      Number(x) === Number(y)
    )
  }
  return false
}

export { isLooselyEqual }
```

## 规律

👀 了实现，现在可以总结出一个规律了

1. null 只能和 undefined 宽松相等
2. 原始类型优先，对象转原始值
3. 数字类型优先，布尔值转数字

所以说 `null == 0 = false`，他直接走到了最后一步的 false。

MDN 总结：

1. 类型相同时
   - Object: 引用值相同为 true
   - String: 字符串相同为 true
   - Number: 数字相同为 true，`+0 == -0`，如果有一个是`NaN`，就返回 false
   - BigInt: 大整数相同为 true
   - Boolean: 布尔值相同为 true
   - Symbol: 引用相同时为 true，`Symbol() != Symbol()`
2. 有一个是 null / undefined，另一个也必须是 null / undefined，才是 true，否则 false
3. 一个是基本类型，一个是对象，那么按这个顺序将对象转基本类型再比较：`@@toPrimitive() -> valueOf() -> toString()`

## 有一方为对象

在 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Equality#%E6%8F%8F%E8%BF%B0) 中如下解释：

> 如果其中一个操作数是对象，另一个是基本类型，按此顺序使用对象的 @@toPrimitive()（以 "default" 作为提示），valueOf() 和 toString() 方法将对象转换为基本类型。（这个基本类型转换与相加中使用的转换相同。）

toPrimitive 一般不会自己定义，但是 valueOf 和 toString 会，所以可以重写这两个方法来改变`==`的行为。

先看看他们的默认返回值是什么:

```js
const obj = {
  name: 'temp',
  age: 10,
}
console.log(obj.valueOf())
// { name: 'temp', age: 10 }
console.log(obj.toString())
// [object Object]

const arr = [1, 34]
console.log(arr.valueOf())
// [ 1, 34 ]
console.log(arr.toString())
// 1,34
```

可以看出，对象的`valueOf`默认返回对象本身，`toString`默认返回`[object Object]`，数组的`valueOf`默认返回数组本身，`toString`默认返回数组的内容，相当于 arr.join(',')。

因此就可以直接写出这样的比较：

```js
const obj = {
  name: 'temp',
  age: 10,
}

console.log(obj == '[object Object]') // true
console.log([1, 2] == '1,2') // true
```

我们可以知道，优先调用的是`valueOf`，但是如果他返回的不是一个基本类型，那么就会调用`toString`。

```js
const obj = {
  valueOf() {
    return 1
  },
  toString() {
    return '2'
  },
}

console.log(obj == 1) // true
console.log(obj == '2') // false
```

这里就是因为`valueOf`返回了一个基本类型，所以就不会调用`toString`了。

```js
const obj = {
  valueOf() {
    return []
  },
  toString() {
    return '2'
  },
}

console.log(obj == 1) // false
console.log(obj == '2') // true
```

这里就是因为 valueOf 返回的不是基本类型，所以就会调用 toString 了。

所以我们也可以实现这样的比较：

```js
String.prototype.valueOf = function () {
  console.log('valueOf')
  return 2
}

String.prototype.toString = function () {
  console.log('toString')
}
console.log(2 == new String('3'))
```

## 常见误区

1. `==`不比较类型
   看代码就知道了，会比较类型，然后能判定是否要类型转换
2. `==`一定会隐式类型转换
   不一定，如果类型相同，直接返回`===`的结果
3. 只有`==`会触发隐式类型转换
   并不是

## 参考链接

- [前端进阶：== 运算符哪里难了？手写 == 底层原理根本没在怕的](https://juejin.cn/post/7281536730900103180)
- [7.2.14 IsLooselyEqual](https://262.ecma-international.org/#sec-islooselyequal)
