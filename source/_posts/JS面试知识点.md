---
title: JS面试知识点
tags:
  - JS
  - 面试
  - 前端
categories: 面试
cover: "https://w.wallhaven.cc/full/3l/wallhaven-3lw5g9.jpg"
abbrlink: 40630
date: 2023-05-09 16:50:01
---

# JS 基

## let，const，var

1. var 声明变量的作用域是函数级别的，不受块级作用域的限制。在全局作用域中声明的变量会成为全局对象的属性。 
2. let 声明的变量是块级作用域的，只在声明的块内有效。在 for 循环中，每次迭代都会创建一个新的变量。 
3. const 声明的变量也是块级作用域的，和 let 类似，但是其值不能被重新赋值，只能被赋值一次。

## 数据类型

首先分为两种，基本类型和引用类型

基本：

- number
- string
- boolean
- undefined
- symbol
- null

引用统称 object

- array
- object
- function

区别：

- 原始数据类型存储在**栈**里面，因为频繁使用，占据空间小，大小固定
- 引用数据类型存储在**堆**里面，占据空间大，大小不固定。
  - 在栈里面存储了指针，指针指向堆中该实体的其实位置。
  - 寻找引用值时，会现在栈里面检索地址，再从堆里面获得实体

## 判断数据类型

```js
typeof 3; // 'number'
typeof "a"; // 'string'
typeof true; // 'boolean'
typeof undefined; // 'undefined'
typeof Symbol(); // 'symbol'

// 特殊
typeof BigInt(10); // 'bigint'
//
typeof null; // 'object'
typeof {}; // 'object'
typeof new Map(); // 'object'
typeof new Set(); // 'object'
// 函数
typeof function () {}; // 'funtion'
```

1. `typeof`

   - 能判断基本类型，但是 null 和其他一些输出 obj

   - null，array，object 都输出 object，function 输出 function

2. `instanceof`

   - 沿着原型链去找，检测实例对象是不是属于某一个构造函数
   - 不能检测基本数据类型
   - 不一定准确，原型链可能被修改，在原型链上找到构造函数就返回 true
   - `[] instanceof Array`true

3. `Object.prototype.toString`

   - 专门检测数据类型的方法，返回值是字符串如`[object String]`
   - `toString.call(null)`，[object Null]，可以从第八位截取

4. 检测数组`Array.isArray([])`，true

**判断变量是否是数组的方式：**

1. `Array.isArray()`
2. `toString.call()`
3. `instanceof`
4. 原型链
5. `constructor.name`

## [String 常用方法](https://web.qianguyihao.com/04-JavaScript基础/15-内置对象 String：字符串的常见方法.html#内置对象简介)

## [Number 和 Math](https://web.qianguyihao.com/04-JavaScript%E5%9F%BA%E7%A1%80/16-%E5%86%85%E7%BD%AE%E5%AF%B9%E8%B1%A1%EF%BC%9ANumber%E5%92%8CMath.html)

## [Array](https://web.qianguyihao.com/04-JavaScript%E5%9F%BA%E7%A1%80/19-%E6%95%B0%E7%BB%84%E7%9A%84%E5%B8%B8%E8%A7%81%E6%96%B9%E6%B3%95.html#%E6%95%B0%E7%BB%84%E7%9A%84%E6%96%B9%E6%B3%95%E6%B8%85%E5%8D%95)

常见方法：

### 数组元素的添加和删除

| 方法      | 描述                                                                       | 是否改变原数组 |
| :-------- | :------------------------------------------------------------------------- | :------------- |
| push()    | 向数组的**最后面**插入一个或多个元素，返回结果为新数组的**长度**           | 是             |
| pop()     | 删除数组中的**最后一个**元素，返回结果为**被删除的元素**                   | 是             |
| unshift() | 在数组**最前面**插入一个或多个元素，返回结果为新数组的**长度**             | 是             |
| shift()   | 删除数组中的**第一个**元素，返回结果为**被删除的元素**                     | 是             |
|           |                                                                            |                |
| splice()  | 从数组中**删除**指定的一个或多个元素，返回结果为**被删除元素组成的新数组** | 是             |
| slice()   | 从数组中**提取**指定的一个或多个元素，返回结果为**新的数组**               | 不是           |
|           |                                                                            |                |
| concat()  | 合并数组：连接两个或多个数组，返回结果为**新的数组**                       | 不是           |
| fill()    | 填充数组：用固定的值填充数组，返回结果为**新的数组**                       | 是             |

- `sort()`，按照的是 Unicode 编码排序，所以需要在里面写函数，改变原数组
- `reverse()`，反转数组，改变原数组

### 查找元素

| 方法                  | 描述                                                                           | 备注                                                     |
| :-------------------- | :----------------------------------------------------------------------------- | :------------------------------------------------------- |
| indexOf(value)        | 从前往后索引，检索一个数组中是否含有指定的元素                                 |                                                          |
| lastIndexOf(value)    | 从后往前索引，检索一个数组中是否含有指定的元素                                 |                                                          |
| includes(item)        | 数组中是否包含指定的内容                                                       |                                                          |
| find(function())      | 找出**第一个**满足「指定条件返回 true」的元素                                  |                                                          |
| findIndex(function()) | 找出**第一个**满足「指定条件返回 true」的元素的 index                          |                                                          |
| every()               | 确保数组中的每个元素都满足「指定条件返回 true」，则停止遍历，此方法才返回 true | 全真才为真。要求每一项都返回 true，最终的结果才返回 true |
| some()                | 数组中只要有一个元素满足「指定条件返回 true」，则停止遍历，此方法就返回 true   | 一真即真。只要有一项返回 true，最终的结果就返回 true     |

### 遍历

- for
- forEact
- for of
- map
- filter
- reduce

forEach 只能遍历，不能改动原数组，map 返回新数组

### [数组排序](https://web.qianguyihao.com/04-JavaScript基础/19-数组的常见方法.html#数组排序)

### [reduce](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/reduce)

```ts
/*
    reduce()
    reduce(
    (previousValue, currentValue, currentIndex, array) => {},
    initialValue);

    previousValue：上一次调用 callbackFn 时的返回值。在第一次调用时，若指定了初始值 initialValue，其值则为 initialValue，否则为数组索引为 0 的元素 array[0]。
    currentValue：数组中正在处理的元素。在第一次调用时，若指定了初始值 initialValue，其值则为数组索引为 0 的元素 array[0]，否则为 array[1]。
    currentIndex：数组中正在处理的元素的索引。若指定了初始值 initialValue，则起始索引号为 0，否则从索引 1 起始。
    array：用于遍历的数组。

    没有initialValue的时候previousValue初始值默认为数组的第一项，此时循环从数组的第二项开始，有第二个参数的时候previousValue为第二个参数值，此时循环从数组的第一项开始。
*/

const arr = [2, 4, 7, 2, 3, 3];

// 求和
const sum = arr.reduce((lastValue, nowValue) => lastValue + nowValue);
console.log(sum);

// 求平均
const average = arr.reduce((lastValue, nowValue, index, arr) => {
  lastValue += nowValue;
  if (index == arr.length - 1) return lastValue / arr.length;
  else return lastValue;
});
console.log(average);

// 求value在arr中出现的次数
function repeatCount(arr: number[], value: number) {
  if (!arr || arr.length == 0) return 0;
  return arr.reduce((totalCount, item) => {
    if (item == value) totalCount++;
    return totalCount;
  }, 0);
}
console.log(repeatCount(arr, 3));

// 求最大值
let max = arr.reduce((last, now) => Math.max(last, now));
console.log(max);

export {};
```

### [filter](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)

```ts
// filter()
// 筛选过滤
// 语法：
// array.filter(function(item, index, arr), thisValue)
let arr1: number[] = [1, 2, 3, -3, 6, 78];

// 找出大于4的元素
let arr2: number[] = arr1.filter((item: number) => item > 4);
console.log(arr2); // [ 6, 78 ]

// 将指定类型的对象找出来
let arr3: { name: string; type: string }[] = [
  { name: "张三", type: "A" },
  { name: "李四", type: "B" },
  { name: "王五", type: "A" },
];

let arr4 = arr3.filter((item) => item.type === "A");
console.log(arr4);

export {};
```

## for...in 和 for...of

in 是遍历对象，of 是遍历数组

in 获取的是键名，of 是键值对的值

for...of 可以配合 for 循环的语句使用，可以随时跳出循环

```js
let arr = [1, 2, 5];

let obj = {
  age: 18,
  name: "张三",
  info: {
    sex: "男",
  },
};

for (let i of arr) {
  console.log(i); // 直接是值
}
for (let i in arr) {
  console.log("i:", i, ",arr[i]:", arr[i]);
  // i: 0 ,arr[i]: 1
  // i: 1 ,arr[i]: 2
  // i: 2 ,arr[i]: 5
}

for (let i in obj) {
  console.log("i:", i, ",obj[i]:", obj[i]);
  // i: name ,obj[i]: 张三
  // i: age ,obj[i]: 18
  // i: info ,obj[i]: { sex: '男' }
}
```

## 常用对象

- Array
- String
- Date
- Globle
- window
- Math

## == 和 === 的区别

[你确定你知道 == 和 === 的区别吗?](https://juejin.cn/post/7168761409629601800)

1. 如果 `x` 和 `y` 的类型相同:
   1. 如果 `x` 是 `undefined`,返回 `true`;
   2. 如果 `x` 是 `null`,返回 `true`;
   3. 如果 `x` 是`Number` 类型,则:
      1. 如果 `x` 是 `NaN`,则返回 `false`;
      2. 如果 `y` 是 `NaN`,则返回 `false`;
      3. 如果 `x` 和 `y` 相同,则返回 `true`;
      4. 如果 `x`是 `-0` 和 `y` 是 `+0`,则返回 `true`;
      5. 如果 `y`是 `-0` 和 `x` 是 `+0`,则返回 `true`;
      6. 其他情况返回 `false`;
   4. 如果 `x` 是 `string` 类型,并且 `x` 和 `y` 的完全相同的值(长度相同，对应位置的字符相同),则返回 `true`;
   5. 如果 `x` 的 `Boolean` 类型,并且 `x` 和 `y` 都是 `true` 或者 `false`,则返回 `true`,否则返回 `false`;
   6. 如果 `x` 和 `y` 引用同一个对象,则返回 `true`,否则返回 `false`;
2. 如果 `x` 为 `null` 且 `y` 为 `undefined`，则返回 `true`;
3. 如果 `y` 为 `null` 且 `x` 为 `undefined`，则返回 `true`;
4. 如果 `x` 是 `number` 类型且 `y` 是 `string` 类型,则返回 `x == ToNumber(y)`的比较结果;
5. 如果 `y` 是 `number` 类型且 `x` 是 `string` 类型,则返回 `y == ToNumber(x)`的比较结果;
6. 如果 `x` 是 `boolean` 类型,返回 `ToNumber(x) == y`的比较结果;
7. 如果 `y` 是 `boolean` 类型,返回 `ToNumber(y) == x`的比较结果;
8. 如果 `x` 是 `string` 类型或者 `number` 类型,并且 `y` 是 `object` 类型,返回 `x == ToPrimitive(y)` 的比较结果;
9. 如果 `y` 是 `string` 类型或者 `number` 类型,并且 `x` 是 `object` 类型,返回 `y == ToPrimitive(x)` 的比较结果;
10. 否则返回 `false`;

# JS 进阶

## 原型链

`__proto__`作为不同对象之间的桥梁，用来指向创建它的构造函数的原型对象的

每个对象的`__proto__`都是指向它的构造函数的原型对象`prototype`的

理解为`xxx.__proto__ === 构造他的函数.prototype`

```js
// person 的构造函数是 Person
person.__proto__ === Person.prototype;
```

```js
// Person 的构造函数是 Object
Person.__proto__ === Function.prototype;
```

```js
// Person 的构造函数是 Object
Object.__proto__ === Function.prototype;
```

```js
// Object 的原型的__proto__指向 null
Object.prototype.__proto__ === null;
```

总结：

- 一切对象都是继承自`Object`对象，`Object` 对象直接继承根源对象`null`
- 一切的函数对象（包括 `Object` 对象），都是继承自 `Function` 对象
- `Object` 对象直接继承自 `Function` 对象
- `Function`对象的`__proto__`会指向自己的原型对象，最终还是继承自`Object`对象

```js
function Person(name) {
  this.name = name;
  this.age = 18;
  this.sayName = () => {
    console.log(this.name);
  };
}

let person = new Person("张三");
console.log("🚀 ~ person:", person);

// 这俩个是一样的
console.log("🚀 ~ person.__proto__:", person.__proto__);
console.log("🚀 ~ Person.prototype:", Person.prototype);

// person 的构造函数是 Person
console.log(
  "🚀 ~ person.__proto__ === Person.prototype:",
  person.__proto__ === Person.prototype
);

// Person 的构造函数是 Object
console.log(
  "🚀 ~ Person.__proto__ === Object.prototype:",
  Person.__proto__ === Function.prototype
);

// Object 的构造函数是 Function
console.log(
  "🚀 ~ Object.__proto__ === Function.prototype:",
  Object.__proto__ === Function.prototype
);
```

![](https://tsejx.github.io/javascript-guidebook/static/prototype-chain.bbfd7b97.jpg)

[原型链 - JavaScript Guidebook (tsejx.github.io)](https://tsejx.github.io/javascript-guidebook/object-oriented-programming/inheritance/prototype-chain/)

- **对象**：`__proto__` 和 `constructor` 是对象独有的。
- **函数**：`prototype` 是函数独有的。但是函数也是对象，所以函数也有 `__proto__` 和 `constructor`。

## [闭包](https://web.qianguyihao.com/04-JavaScript%E5%9F%BA%E7%A1%80/26-%E9%97%AD%E5%8C%85.html)

**闭包的定义**：指有权访问另一个函数作用域中的变量的函数，一般情况就是在一个函数中包含另一个函数。

**闭包的作用**：

- 访问函数内部变量、保持函数在环境中一直存在，不会被垃圾回收机制处理
- 函数内部声明的变量是局部的，只能在函数内部访问到，但是函数外部的变量是对函数内部可见的。
- 子级可以向父级查找变量，逐级查找，直到找到为止或全局作用域查找完毕。
- 因此我们可以在函数内部再创建一个函数，这样对内部的函数来说，外层函数的变量都是可见的，然后我们就可以访问到他的变量了。

保留了作用域链，外部可以访问

```js
function outerFn() {
  let outerText = "我是outerFn";
  function innerFn() {
    console.log("outerText:", outerText);
  }
  return innerFn;
}

let res = outerFn();
res(); // 输出：我是outerFn
```

**用途：**

- 封装私有变量

  ```js
  function add() {
    let count = 0;
    function addCount() {
      count++;
      console.log(count);
    }
    return addCount;
  }

  let test = add();
  test(); //1
  test(); //2
  test(); //3
  ```

- 做缓存

- 模块化编程

**缺点：**容易内存泄露，所以要及时释放闭包，或者使用立即执行函数

**使用场景：**

- `return`返回一个函数
- 节流防抖
- 柯里化

## [this 指向](https://web.qianguyihao.com/04-JavaScript%E5%9F%BA%E7%A1%80/25-this%E6%8C%87%E5%90%91.html#%E6%89%A7%E8%A1%8C%E6%9C%9F%E4%B8%8A%E4%B8%8B%E6%96%87)

> [js 中 this 是什么，代表谁，详解看懂了却总碰到难以解释的现象？](https://juejin.cn/post/7233057834286596155)

## [事件循环，宏任务，微任务](https://tsejx.github.io/javascript-guidebook//core-modules/executable-code-and-execution-contexts/concurrency-model/event-loop#%E4%BA%8B%E4%BB%B6%E5%BE%AA%E7%8E%AF)

运行机制：

1. 所有同步任务都在主线程上执行，形成一个 **执行栈**（Execution Context Stack）
2. 主线程之外，还存在一个 **任务队列**（Task Queue）。只要异步任务有了运行结果，就在 **任务队列** 之中放置一个事件
3. 一旦 **执行栈** 中的所有同步任务执行完毕，系统就会读取 **任务队列**，看看里面有哪些待执行事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行
4. 主线程不断重复上面的第三步

JavaScript 的异步任务根据事件分类分为两种：宏任务（MacroTask）和微任务（MicroTask）

- **宏任务**：main script、setTimeout、setInterval、setImmediate（Node.js）、I/O（Mouse Events、Keyboard Events、Network Events）、UI Rendering（HTML Parsing）、MessageChannel
- **微任务**：Promise.then（非 new Promise）、process.nextTick（Node.js）、MutationObserver

宏任务与微任务的区别在于队列中事件的**执行优先级**。进入整体代码（宏任务）后，开始首次事件循环，当执行上下文栈清空后，事件循环机制会优先检测微任务队列中的事件并推至主线程执行，当微任务队列清空后，才会去检测宏任务队列中的事件，再将事件推至主线程中执行，而当执行上下文栈再次清空后，事件循环机制又会检测微任务队列，如此反复循环。

**宏任务与微任务的优先级**

- 宏任务的优先级高于微任务
- 每个宏任务执行完毕后都必须将当前的微任务队列清空
- 第一个 `<script>` 标签的代码是第一个宏任务
- `process.nextTick` 优先级高于 `Promise.then`

![事件循环机制中宏任务和微任务图解](https://tsejx.github.io/javascript-guidebook/static/workflow.7125d86b.jpg)

```js
console.log(1);

setTimeout(() => {
  console.log(2);
}, 0);

let promise = new Promise((res) => {
  console.log(3);
  resolve();
})
  .then((res) => {
    console.log(4);
  })
  .then((res) => {
    console.log(5);
  });

console.log(6);

// 1 3 6 4 5 2
// 先是1，然后setTimeout把2放到了宏任务里面，然后是Promise的3，并且吧4和5放到了promise的微任务里面，然后是6，然后45，然后2
```

```js
setTimeout(() => {
  console.log(1);
  Promise.resolve().then(() => {
    console.log(7);
  });
}, 0);

console.log(2);

Promise.resolve().then(() => {
  console.log(3);
});

setTimeout(() => {
  console.log(8);
  setTimeout(() => {
    console.log(5);
  }, 0);
}, 0);

setTimeout(() => {
  Promise.resolve().then(() => {
    console.log(4);
  });
}, 0);

console.log(6);

// 2 6 3 1 7 8 4 5
```

## Promise 异步

promise 的三个状态：

- pending，等待除了下面俩个就一直在这个状态
- fulfiled，成功的回调，resolve
- rejected，失败回调

promise.all 等：https://juejin.cn/post/7069805387490263047

## [async/await 异步](https://web.qianguyihao.com/06-JavaScript基础：异步编程/10-Async Await 函数详解.html)

## [深拷贝](https://github.com/Sunny-117/js-challenges/issues/58)

**浅拷贝：**

- 如果属性是基础类型，拷贝的就是值
- 如果是引用类型，拷贝的就是内存地址（指针）

**浅拷贝和赋值的区别：**

- 赋值是赋的对象在栈内存中的地址，不是堆内存中的数据。所以俩个对象指向的同一个存储空间，改一个都会变化
- 浅拷贝是按位拷贝，会创建一个新对象，新对象有原始对象属性值的一份精确拷贝。基本类型就是拷贝值，引用类型就拷贝内存地址

用 for in 浅拷贝：

```js
const obj1 = {
  name: "obj",
  age: 21,
  info: {
    desc: "我是obj11111",
  },
};

const obj2 = {};

for (let key in obj1) {
  obj2[key] = obj1[key];
}

console.log("obj2:", JSON.stringify(obj2));
// obj2: {"name":"obj","age":21,"info":{"desc":"我是obj11111"}}
obj1.info.desc = "我修改了obj1的info的desc";
console.log("obj2:", JSON.stringify(obj2));
// obj2: {"name":"obj","age":21,"info":{"desc":"我修改了obj1的info的desc"}}
```

ES6 提供了语法糖`Object.assgin()`

```js
Object.assign(obj2, obj1);
// 将 obj1 拷贝给 obj2
```

深拷贝：（递归）

```js
const obj1 = {
  name: "obj",
  age: 19,
  info: {
    desc: "我是obj11111",
  },
  arr: [1, 2, 3],
};

let obj2 = {};

deepCopy(obj2, obj1);

console.log("obj1:", JSON.stringify(obj1));
// obj1: {"name":"obj","age":19,"info":{"desc":"我是obj11111"},"arr":[1,2,3]}
console.log("obj2:", JSON.stringify(obj2));
// obj2: {"name":"obj","age":19,"info":{"desc":"我是obj11111"},"arr":[1,2,3]}
obj1.info.desc = "我修改了obj1的info的desc";
console.log("obj1:", JSON.stringify(obj1));
// obj1: {"name":"obj","age":19,"info":{"desc":"我修改了obj1的info的desc"},"arr":[1,2,3]}
console.log("obj2:", JSON.stringify(obj2));
// obj2: {"name":"obj","age":19,"info":{"desc":"我是obj11111"},"arr":[1,2,3]}

function deepCopy(newObj, oldObj) {
  for (let key in oldObj) {
    let item = oldObj[key];
    if (item instanceof Array) {
      newObj[key] = [];
      deepCopy(newObj[key], item);
    } else if (item instanceof Object) {
      newObj[key] = {};
      deepCopy(newObj[key], item);
    } else {
      newObj[key] = item;
    }
  }
}
```

考虑了循环引用的深拷贝

```js
const _completeDeepClone = (target, map = new Map()) => {
  // 1. 需要考虑函数、正则、日期、ES6新对象
  // 2. 需要考虑循环引用问题
  if (typeof target !== "object") return target;
  if (!target) return target;
  const types = ["Function", "RegExp", "Date", "Symbol", "Map", "Set"];
  const constructor = target.constructor;
  if (types.includes(constructor.name)) return new constructor(target);
  if (map.get(target)) return map.get(target);
  map.set(target, true);
  const res = new constructor();
  for (let key in target) {
    if (target.hasOwnProperty(key)) {
      res[key] = _completeDeepClone(target[key], map);
    }
  }
  return res;
};
```

## 节流防抖

>  [【offer 收割机之手写系列】10 分钟带你掌握原理并手写防抖与节流的立即/非立即执行版本 ](https://juejin.cn/post/7078870853315723301)

节流

```js
function throttle(fn, timeout) {
  let timer = null;
  return function (...arg) {
    if (timer) return;
    timer = setTimeout(() => {
      fn.apply(fn, arg);
      timer = null;
    }, timeout);
  };
}
```

防抖

```js
function debounce(fn, wait) {
  let timer = null;
  return function () {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, arguments);
    }, wait);
  };
}
```



## [正则](https://web.qianguyihao.com/04-JavaScript%E5%9F%BA%E7%A1%80/34-%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F.html#%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F%E7%AE%80%E4%BB%8B)

## ES6

- 多了 let，const，块级作用域
- 解构赋值
- 箭头函数，没有自己的 this
- Set，不允许重复，数组去重:`[...new Set(arr)]`
- 模板字符串`name:&{name}`
- Promise
- Set，Map

## 跨域

见计网部分的

## this

**this 的五种情况**

1. 作为普通函数执行时，`this`指向`window`。
2. 当函数作为对象的方法被调用时，`this`就会指向`该对象`。
3. 构造器调用，`this`指向`返回的这个对象`。
4. 箭头函数 箭头函数的`this`绑定看的是`this所在函数定义在哪个对象下`，就绑定哪个对象。如果有嵌套的情况，则 this 绑定到最近的一层对象上。
5. 基于 Function.prototype 上的 `apply 、 call 和 bind `调用模式，这三个方法都可以显示的指定调用函数的 this 指向。`apply`接收参数的是数组，`call`接受参数列表，`bind`方法通过传入一个对象，返回一个`this`绑定了传入对象的新函数。这个函数的 `this`指向除了使用`new `时会被改变，其他情况下都不会改变。若为空默认是指向全局对象 window。

## new 运算符的实现机制

1. 首先创建了一个新的`空对象`
2. `设置原型`，将对象的原型设置为函数的`prototype`对象。
3. 让函数的`this`指向这个对象，执行构造函数的代码（为这个新对象添加属性）
4. 判断函数的返回值类型，如果是值类型，返回创建的对象。如果是引用类型，就返回这个引用类型的对象。

```js
function objectFactory() {
  let newObject = null;
  let constructor = Array.prototype.shift.call(arguments);
  let result = null;
  // 判断参数是否是一个函数
  if (typeof constructor !== "function") {
    console.error("type error");
    return;
  }
  // 新建一个空对象，对象的原型为构造函数的 prototype 对象
  newObject = Object.create(constructor.prototype);
  // 将 this 指向新建对象，并执行函数
  result = constructor.apply(newObject, arguments);
  // 判断返回对象
  let flag =
    result && (typeof result === "object" || typeof result === "function");
  // 判断返回结果
  return flag ? result : newObject;
}
// 使用方法
objectFactory(构造函数, 初始化参数);
```

```js
const _new = function () {
  let newObj = null;
  let res = null;
  let constructor = Array.prototype.shift.apply(arguments);
  if (typeof constructor !== "function") {
    throw new TypeError("传入参数不是函数");
  }
  newObj = Object.create(constructor.prototype);
  res = constructor.apply(newObj, arguments);
  let flag = res && (typeof res === "object" || typeof res === "function");
  return flag ? res : newObj;
};
```

## call，apply，bind

作用都是改变 this 的指向

**基本语法：**

```js
fun.call(thisArg, param1, param2, ...)
fun.apply(thisArg, [param1, param2, ...])
fun.bind(thisArg, param1, param2, ...)
```

`call`和`apply`返回的是 fun 的执行结果，**立即执行**

`bind`的返回值是 fun 的拷贝，并拥有指定的`this`值和初始参数，**返回函数，不立即执行**

**传入的参数：**

`thisArg`

1. fun 的 this 指向 thisArg 对象
2. 非严格模式下：thisArg 指定为 null，undefined，fun 的 this 指向 window
3. 严格模式下：fun 的 this 为 undefined
4. 值为原始值（数字，字符串，布尔值）的时候 this 会指向原始值的自动包装对象，如 String，Number，Boolean

`param:`

1. 如果没有或者 null ， undefined，表示不需要传入
2. apply 第二个参数是数组

调用他们的必须是函数，因为他们是挂在 Function 上面的方法，如`Object.prototype.toString.call(data)`

**核心理念**：复用方法，代码复用，节省内存

**call 和 apply 用哪个：**

- 效果完全一样，区别如下
- **参数数量/顺序确定就用 call，参数数量/顺序不确定的话就用 apply**。
- 考虑可读性：参数数量不多就用 call，参数数量比较多的话，把参数整合成数组，使用 apply。
- 参数集合已经是一个数组的情况，用 apply，比如获取数组最大值/最小值

## 跨域

> [前端跨域整理总结](https://juejin.cn/post/6992268157364731911)

跨域的原因是：浏览器的同源策略，意思是指 JS 只能操作与其宿主网页有相同的"协议 + 域名 + 端口号"三个部分的 DOM

所以就像使用 AJAX 向其他地址发请求时浏览器就会拒绝

解决方案：

### 代理服务器

常见的解决跨域的方式，用自己的服务器来转发请求，比如用 Nginx 配置反向代理服务器。

缺点：需要配置额外的服务器，高并发场景下可能导致性能问题

### 跨域资源共享 CORS

浏览器解决跨域的方式，可以让服务器允许与其他域名的客户端交互

服务器在响应头中添加`Access-Control-Allow-Origin`字段来告诉浏览器允许哪些来源的请求跨域

虽然 CORS 的安全性比代理服务器更高，但是需要服务器端进行相应的配置，同时一些老旧的浏览器可能不支持 CORS。

### JSONP

JSON with Padding 的缩写，利用 script 标签可以跨域的特性来实现数据传输的方式

就是在页面插入一个`<scripy>`标签来加载远程数据

```html
<script src="http://example.com/api/user?callback=handleResponse"></script>
function handleResponse(data) { console.log(data); }
```

```html
<script>
  let script = document.createElement("script");
  script.type = "text/javascript";
  script.src = "http://juejin.com/xxx?callback=handleCallback";
  document.body.appendChild(script);

  function handleCallback(res) {
    console.log(res);
  }
</script>
```

JSONP 是一种比较老的跨域技术，它存在一些安全风险，容易受到中间人攻击，同时只支持 GET 请求，并且无法使用 POST 等其他请求方式。

### WebSocket

双向通信的协议，可以实现浏览器和服务器之间高效实时通信，基于 TCP 协议，可以服务器主动推送消息到客户端。

基于 TCP，不受同源策略限制，性能和安全更优秀

### postMessage

[嗦嗦 postMessage 和 webSocket - 掘金 (juejin.cn)](https://juejin.cn/post/7152773676876857374)

postMessage 是 HTML5 标准中的 API，它可以给我们解决如下问题：

- 页面和新打开的窗口间数据传递
- 多窗口之间数据传递
- 页面与嵌套的 iframe 之间数据传递
- 上面三个场景之间的`跨域传递`

postMessage 接受两个参数，用法如下：

- **参数一**：发送的数据
- **参数二**：你要发送给谁就写谁的地址`(协议 + 域名 +端口`)，也可以设置为`*`，表示任意窗口，为`/`表示与当前窗口同源的窗口
