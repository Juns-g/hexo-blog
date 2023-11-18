---
title: 第一次学习Vue的笔记
tags:
  - Vue
  - 学习
categories: 前端
cover: >-
  https://static001.infoq.cn/resource/image/c4/01/c4251ee38c2039602d69cac1d3ab9d01.jpg
keywords: 学习 前端 Vue
abbrlink: 33990
date: 2023-05-18 19:42:50
---

# Vue

> ## 一些链接：
>
> - [Vue 中 Vuex 的使用](https://blog.csdn.net/lyyrhf/article/details/115217564?spm=1001.2014.3001.5502)
> - [Vue 的脚手架开发详细笔记](https://blog.csdn.net/lyyrhf/article/details/115065386?spm=1001.2014.3001.5502)
> - [Vue 的组件化开发详细笔记](https://blog.csdn.net/lyyrhf/article/details/114977307?spm=1001.2014.3001.5502)
> - [Vue 的 MVVM 架构及语法超详细笔记](https://blog.csdn.net/lyyrhf/article/details/114976702?spm=1001.2014.3001.5502)
> - [Vue3.x 知识图谱](https://gitee.com/jishupang/vue3-knowledge-map)
> - [VUE 组件汇总](https://juejin.cn/post/6844903603421839368)
> - [6 个实用的 vue 组件库](https://juejin.cn/post/7121198442105274405)
> - [vue3 新文档](https://vuejs.org/)
> - [尚硅谷 vue3 笔记](https://wekenw.gitee.io/vuenote/)

## 安装配置

### 对于学习

可以这样使用最新版本：

```html
<script src="https://unpkg.com/vue@next"></script>
```

### npm

在用 Vue 构建大型应用时推荐使用 npm 安装

```sh
# 最新稳定版
$ npm install vue@next
```

对于 Vue 3，你应该使用 `npm` 上可用的 Vue CLI v4.5 作为 `@vue/cli`。

```sh
npm install -g @vue/cli
```

### Vite

[Vite](https://cn.vitejs.dev/) 是一个 web 开发构建工具，由于其原生 ES 模块导入方式，可以实现闪电般的冷服务器启动。

通过在终端中运行以下命令，可以使用 Vite 快速构建 Vue 项目。

使用 npm：

```sh
# npm 6.x
$ npm init vite@latest <project-name> --template vue

# npm 7+，需要加上额外的双短横线
$ npm init vite@latest <project-name> -- --template vue

$ cd <project-name>
$ npm install
$ npm run dev
```

或者 pnpm:

```sh
$ pnpm create vite <project-name> -- --template vue
$ cd <project-name>
$ pnpm install
$ pnpm dev
```

## 常用 Composition API

### Vue3 响应式原理

实现原理:

- 通过 Proxy（代理）: 拦截对象中任意属性的变化, 包括：属性值的读写、属性的添加、属性的删除等。
- 通过 Reflect（反射）: 对源对象的属性进行操作。
- MDN 文档中描述的 Proxy 与 Reflect：
  - Proxy：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy
  - Reflect：https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect

```js
new Proxy(data, {
  // 拦截读取属性值
  get(target, prop) {
    return Reflect.get(target, prop);
  },
  // 拦截设置属性值或添加新属性
  set(target, prop, value) {
    return Reflect.set(target, prop, value);
  },
  // 拦截删除属性
  deleteProperty(target, prop) {
    return Reflect.deleteProperty(target, prop);
  },
});

proxy.name = "tom";
```

#### Vue2 的响应式

- 实现原理：

  - 对象类型：通过`Object.defineProperty()`对属性的读取、修改进行拦截（数据劫持）。

  - 数组类型：通过重写更新数组的一系列方法来实现拦截。（对数组的变更方法进行了包裹）。

    ```js
    Object.defineProperty(data, "count", {
      get() {},
      set() {},
    });
    ```

- 存在问题：

  - 新增属性、删除属性, 界面不会更新。
  - 直接通过下标修改数组, 界面不会自动更新。

### reactive 对比 ref

- 从定义数据角度对比：
  - ref 用来定义：**基本类型数据**。
  - reactive 用来定义：**对象（或数组）类型数据**。
  - 备注：ref 也可以用来定义**对象（或数组）类型数据**, 它内部会自动通过`reactive`转为**代理对象**。
- 从原理角度对比：
  - ref 通过`Object.defineProperty()`的`get`与`set`来实现响应式（数据劫持）。
  - reactive 通过使用**Proxy**来实现响应式（数据劫持）, 并通过**Reflect**操作**源对象**内部的数据。
- 从使用角度对比：
  - ref 定义的数据：操作数据**需要**`.value`，读取数据时模板中直接读取**不需要**`.value`。
  - reactive 定义的数据：操作数据与读取数据：**均不需要**`.value`。

> 个人建议的写法是这样，最后 return 他的 data
>
> ```js
> let data = reactive({
>   person: {},
>   student: {},
> });
> ```

### setup 的两个注意点

- setup 执行的时机
  - 在 beforeCreate 之前执行一次，this 是 undefined。
- setup 的参数
  - props：值为对象，包含：组件外部传递过来，且组件内部声明接收了的属性。
  - context：上下文对象
    - attrs: 值为对象，包含：组件外部传递过来，但没有在 props 配置中声明的属性, 相当于 `this.$attrs`。
    - slots: 收到的插槽内容, 相当于 `this.$slots`。
    - emit: 分发自定义事件的函数, 相当于 `this.$emit`。

### 计算属性与监视

#### 1.computed 函数

- 与 Vue2.x 中 computed 配置功能一致

- 写法，但是 name1 和 name2 中间的空格删掉的话会出 bug

  ```js
  import { reactive, computed } from "vue";
  export default {
    name: "Demo",
    setup() {
      let person = reactive({
        name1: "张",
        name2: "三",
      });
      // 简写，不考虑计算属性被修改的情况
      /* person.fullName = computed(() => {
        return person.name1 + " " + person.name2;
      }); */

      //完整写法，考虑读写
      person.fullName = computed({
        get() {
          return person.name1 + " " + person.name2;
        },
        set(value) {
          const nameArr = value.split(" ");
          person.name1 = nameArr[0];
          person.name2 = nameArr[1];
        },
      });
      return {
        person,
      };
    },
  };
  ```

#### 2.watch 函数

- 与 Vue2.x 中 watch 配置功能一致

- 两个小“坑”：

  - 监视 reactive 定义的响应式数据时：oldValue 无法正确获取、强制开启了深度监视（deep 配置失效）。
  - 监视 reactive 定义的响应式数据中某个属性时：deep 配置有效。

- 普通 ref 属性监视的时候不需要加.value，监视的是属性，不是其中的值

- 但是对象类型的本应该用 reactive 的用了 ref，就需要.value，因为他的 value 里面才是 Proxy；或者直接开启 deep

  ```js
  // 1. 监视ref定义的一个响应式数据
  watch(
    sum,
    (newValue, oldValue) => {
      console.log(newValue, oldValue);
    },
    { immediate: true }
  );

  // 2. 监视ref定义的多个响应式数据
  watch(
    [sum, msg],
    (newValue, oldValue) => {
      console.log(newValue, oldValue);
    },
    { immediate: true } // 刚出现也执行watch
  );

  // 3. 监视reactive定义的一个响应式数据的全部属性
  // ! 此处无法获取正确的oldValue，新旧一样的
  // ! 强制开启了深度监视（deep配置无效）
  watch(
    person,
    (newValue, oldValue) => {
      console.log(newValue, oldValue);
    },
    { deep: false } // 此处deep配置无效
  );

  // 4. 监视reacive定义的一个响应式数据中的一个属性
  watch(
    () => person.age,
    (newValue, oldValue) => {
      console.log(newValue, oldValue);
    }
  );

  // 5. 监视reacivee定义的一个响应式数据中的某些属性
  // ! 能监听到旧数据了
  watch([() => person.name, () => person.age], (newValue, oldValue) => {
    console.log(newValue, oldValue);
  });

  // 特殊情况
  watch(
    () => person.job,
    (newValue, oldValue) => {
      console.log(newValue, oldValue);
    },
    { deep: true } // 监听深层属性，需要开启deep
  );
  ```

#### 3. watchEffect 函数

- watch 的套路是：既要指明监视的属性，也要指明监视的回调。

- watchEffect 的套路是：不用指明监视哪个属性，监视的回调中用到哪个属性，那就监视哪个属性。

- watchEffect 有点像 computed：

  - 但 computed 注重的计算出来的值（回调函数的返回值），所以必须要写返回值。
  - 而 watchEffect 更注重的是过程（回调函数的函数体），所以不用写返回值。

  ```js
  // watchEffect里面的数据只要变化了就会执行
  watchEffect(() => {
    const x1 = sum.value;
    const x2 = person.job.j1.salary;
    console.log(x1, x2);
  });
  ```

### 生命周期

<img src="https://v2.cn.vuejs.org/images/lifecycle.png" alt="Vue2生命周期"  />![vue3生命周期](https://cn.vuejs.org/assets/lifecycle.16e4c08e.png)

- Vue3.0 中可以继续使用 Vue2.x 中的生命周期钩子，但有有两个被更名：

  - `beforeDestroy`改名为 `beforeUnmount`
  - `destroyed`改名为 `unmounted`

  ```js
  export default {
    name: "App",
    components: {
      Demo,
    },
    setup() {},
    beforeCreate() {},
    created() {},
    beforeMount() {},
    mounted() {},
    beforeUpdate() {},
    updated() {},
    beforeUnmount() {},
    unmounted() {},
  };
  ```

  - 但是写在外面访问 setup 里面的数据会很麻烦

- Vue3.0 也提供了 Composition API 形式的生命周期钩子，与 Vue2.x 中钩子对应关系如下：

  - `beforeCreate`==>`setup()`
  - `created`==>`setup()`
  - `beforeMount` ==>`onBeforeMount`
  - `mounted`==>`onMounted`
  - `beforeUpdate`==>`onBeforeUpdate`
  - `updated` ==>`onUpdated`
  - `beforeUnmount` ==>`onBeforeUnmount`
  - `unmounted` ==>`onUnmounted`

### 自定义 hook 函数

- 什么是 hook？—— 本质是一个函数，把 setup 函数中使用的 Composition API 进行了封装。
- 类似于 vue2.x 中的 mixin。
- 自定义 hook 的优势: 复用代码, 让 setup 中的逻辑更清楚易懂。

在 src 下新建 hooks 文件夹，里面放要重复用的 js 文件，然后导出引入即可复用

- usePoint.js

- ```js
  import { reactive, onMounted, onBeforeUnmount } from "vue";
  export default function () {
    let p = reactive({
      x: 0,
      y: 0,
    });

    function savePoint(e) {
      console.log(e.pageX, e.pageY);
      p.x = e.pageX;
      p.y = e.pageY;
    }

    onMounted(() => {
      window.addEventListener("click", savePoint);
    });

    onBeforeUnmount(() => {
      window.removeEventListener("click", savePoint);
    });
    return p;
  }
  ```

- Demo.vue

- ```js
  import { ref } from "vue";
  import usePoint from "../hooks/usePoint";
  export default {
    name: "Demo",
    setup() {
      let sum = ref(0);
      let p = usePoint();
      return {
        sum,
        p,
      };
    },
  };
  ```

### toRef

- 作用：创建一个 ref 对象，其 value 值指向另一个对象中的某个属性。
- 语法：`const name = toRef(person,'name')`
- 应用: 要将响应式对象中的某个属性单独提供给外部使用时。
- 扩展：`toRefs` 与`toRef`功能一致，但可以批量创建多个 ref 对象，语法：`toRefs(person)`

```js
return {
  /* name: toRef(p, "name"),
    age: toRef(p, "age"),*/
  salary: toRef(p.job.j1, "salary"),
  ...toRefs(p), // 这是展开
};
```

```vue
<template>
  <h1>姓名：{{ name }}</h1>
  <h1>年龄：{{ age }}</h1>
  <h1>薪水：{{ salary }}</h1>
  <br /><br />
  <button @click="age++">age++</button>
  <button @click="salary++">salary++</button>
</template>
```

## 其他 Composition API

### 1.shallowReactive 与 shallowRef

- shallowReactive：只处理对象最外层属性的响应式（浅响应式）。
- shallowRef：只处理基本数据类型的响应式, 不进行对象的响应式处理。
- 什么时候使用?
  - 如果有一个对象数据，结构比较深, 但变化时只是外层属性变化 ===> shallowReactive。
  - 如果有一个对象数据，后续功能不会修改该对象中的属性，而是生新的对象来替换 ===> shallowRef。

> 个人感觉就是相当于基本数据

### 2.readonly 与 shallowReadonly

- readonly: 让一个响应式数据变为只读的（深只读）。
- shallowReadonly：让一个响应式数据变为只读的（浅只读）。
- 应用场景: 不希望数据被修改时。

### 3.toRaw 与 markRaw

- toRaw：
  - 作用：将一个由`reactive`生成的**响应式对象**转为**普通对象**。
  - 使用场景：用于读取响应式对象对应的普通对象，对这个普通对象的所有操作，不会引起页面更新。
- markRaw：
  - 作用：标记一个对象，使其永远不会再成为响应式对象。
  - 应用场景:
    1. 有些值不应被设置为响应式的，例如复杂的第三方类库等。
    2. 当渲染具有不可变数据源的大列表时，跳过响应式转换可以提高性能。

> - toRaw 一般用在把已经由 reactive 生成的响应式数据转换回普通对象
> - markRaw 使用场景更多，因为将普通对象转换为响应式对象需要小号性能，这可以把一些复杂并且不需要改变的对象不去转化，可以节约性能

### 4.customRef

- 作用：创建一个自定义的 ref，并对其依赖项跟踪和更新触发进行显式控制。

- 实现防抖效果：

  ```vue
  <template>
    <br /><br />
    <input
      type="text"
      v-model="keyword"
    />
    <h1>{{ keyword }}</h1>
  </template>

  <script>
  // eslint-disable-next-line no-unused-vars
  import { customRef } from "vue";
  export default {
    // eslint-disable-next-line vue/multi-word-component-names
    name: "Demo",
    setup() {
      // 自定义的myRef，里面的配置需要自己完善
      function myRef(value, delay) {
        let timer;
        // 通过customRef来自定义
        return customRef((track, trigger) => {
          return {
            get() {
              track(); // 告诉Vue这个value值需要追踪
              return value;
            },
            set(newValue) {
              clearTimeout(timer);
              timer = setTimeout(() => {
                value = newValue;
                trigger(); //告诉Vue去更新界面
              }, delay);
            },
          };
        });
      }
      // 使用自定义的ref
      let keyword = myRef("hello", 200);
      return { keyword };
    },
  };
  </script>
  ```

### 5.provide 与 inject

- 作用：实现**祖与后代组件间**通信

- 套路：父组件有一个 `provide` 选项来提供数据，后代组件有一个 `inject` 选项来开始使用这些数据

- 具体写法：

  1.  祖组件中：

      ```js
      setup() {
          let data = reactive({
            car: {
              name: "BenChi",
              price: "40W",
            },
          });
          provide("car", data.car); // 给自己的后代组价传递数据
          return { ...toRefs(data.car) };
        },
      ```

  2.  后代组件中：

      ```js
      import { inject } from "vue";
      export default {
        name: "Son",
        setup() {
          let car = inject("car");
          return { car };
        },
      };
      ```

### 6.响应式数据的判断

- isRef: 检查一个值是否为一个 ref 对象
- isReactive: 检查一个对象是否是由 `reactive` 创建的响应式代理
- isReadonly: 检查一个对象是否是由 `readonly` 创建的只读代理
- isProxy: 检查一个对象是否是由 `reactive` 或者 `readonly` 方法创建的代理

## Composition API 的优势

### 1.Options API 存在的问题

使用传统 OptionsAPI 中，新增或者修改一个需求，就需要分别在 data，methods，computed 里修改 。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f84e4e2c02424d9a99862ade0a2e4114~tplv-k3u1fbpfcp-watermark.image)

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e5ac7e20d1784887a826f6360768a368~tplv-k3u1fbpfcp-watermark.image)

### 2.Composition API 的优势

我们可以更加优雅的组织我们的代码，函数。让相关功能的代码更加有序的组织在一起。

![img](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc0be8211fc54b6c941c036791ba4efe~tplv-k3u1fbpfcp-watermark.image)

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6cc55165c0e34069a75fe36f8712eb80~tplv-k3u1fbpfcp-watermark.image)

## 新的组件

### 1.Fragment

- 在 Vue2 中: 组件必须有一个根标签
- 在 Vue3 中: 组件可以没有根标签, 内部会将多个标签包含在一个 Fragment 虚拟元素中
- 好处: 减少标签层级, 减小内存占用

### 2.Teleport

- 什么是 Teleport？—— `Teleport` 是一种能够将我们的**组件 html 结构**移动到指定位置的技术。

- Dialog.vue

  ```vue
  <template>
    <div>
      <button @click="isShow = true">点我弹窗</button>
      <teleport to="body">
        <div
          class="mask"
          v-if="isShow"
        >
          <div class="dialog">
            <h3>我是弹窗</h3>
            <h4>内容诶！</h4>
            <h4>内容诶！</h4>
            <h4>内容诶！</h4>
            <button @click="isShow = false">关闭弹窗</button>
          </div>
        </div>
      </teleport>
    </div>
  </template>

  <script>
  import { ref } from "vue";

  export default {
    name: "Dialog",
    setup() {
      let isShow = ref(false);

      return { isShow };
    },
  };
  </script>

  <style scoped>
  .dialog {
    text-align: center;
    position: absolute;
    /* div的左上角居中 */
    left: 50%;
    top: 50%;
    /* 自身向左上角移动50% */
    transform: translate(-50%, -50%);
    width: 300px;
    height: 300px;
    background-color: yellow;
    padding: 10px;
  }

  .mask {
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.5);
  }
  </style>
  ```

### 3.Suspense

- 等待异步组件时渲染一些额外内容，让应用有更好的用户体验

- 使用步骤：

  - 异步引入组件

    ```js
    import { defineAsyncComponent } from "vue";
    const Child = defineAsyncComponent(() => import("./components/Child.vue"));
    ```

  - 使用`Suspense`包裹组件，并配置好`default` 与 `fallback`

    ```vue
    <template>
      <div class="app">
        <h3>我是App组件</h3>
        <Suspense>
          <template v-slot:default>
            <Child />
          </template>
          <template v-slot:fallback>
            <h3>加载中.....</h3>
          </template>
        </Suspense>
      </div>
    </template>
    ```

## Vue3 对于 2 的其他改变

- data 选项应始终被声明为一个函数。

- 过度类名的更改：

  - Vue2.x 写法

    ```css
    .v-enter,
    .v-leave-to {
      opacity: 0;
    }
    .v-leave,
    .v-enter-to {
      opacity: 1;
    }
    ```

  - Vue3.x 写法

    ```css
    .v-enter-from,
    .v-leave-to {
      opacity: 0;
    }

    .v-leave-from,
    .v-enter-to {
      opacity: 1;
    }
    ```

- **移除**keyCode 作为 v-on 的修饰符，同时也不再支持`config.keyCodes`

- **移除**`v-on.native`修饰符

  - 父组件中绑定事件

    ```vue
    <my-component
      v-on:close="handleComponentEvent"
      v-on:click="handleNativeClickEvent"
    />
    ```

  - 子组件中声明自定义事件

    ```vue
    <script>
    export default {
      emits: ["close"],
    };
    </script>
    ```

- **移除**过滤器（filter）

  > 过滤器虽然这看起来很方便，但它需要一个自定义语法，打破大括号内表达式是 “只是 JavaScript” 的假设，这不仅有学习成本，而且有实现成本！建议用方法调用或计算属性去替换过滤器。

- ......
