---
title: Vue
date: 2020-01-30 10:31:34
categories:
  - - 前端
tags:
  - F1
---

For Vue 2.

<!-- more -->

# Start

## 历史

- 2015 年，1.0 版，是 MVVM 框架
- 2016 年，2.0 版，没有完全遵循 MVVM 模型
- 2019 年，2.6 版
- 2020 年，3.0 版，完全不是 MVVM

## 创建一个 vue 项目

- 使用 @vue-cli
- 或者使用 webpack 或者 rollup 从零开始

## 完整版和运行时版本

- **完整版**：同时包含编译器和运行时的版本，也就是 CDN 里面的 `vue.js`，可以直接在页面里面写 ，相当于把视图放在 html 里面写
- **运行时**：用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码。基本上就是除去编译器的其它一切，也就是 CDN 里面的 `vue.runtime.js`，不支持从 html 里面获取视图，也不支持在 template 里面写，没有编译器（compiler），代码体积小 30%
- webpack 中的 vue-loader 可以在最后打包的时候将 template 里面的东西编译成 JS，因此我们不需要使用完整版

|               | 完整版                                 | 运行时版                                                     | 评价                               |
| ------------- | -------------------------------------- | ------------------------------------------------------------ | ---------------------------------- |
| 特点          | 有 compiler                            | 没有 compiler                                                | compiler 占 40% 的体积             |
| 视图          | 写在 HTML 里，或者写在 template 选项中 | 写在 render 函数里，用 h 来创建标签	h 是 Vue 写好传给 render 的 |                                    |
| CDN 引入      | vue.js                                 | vue.runtime.js                                               | 文件名不同，生产环境后缀为 .min.js |
| webpack 引入  | 需要配置 alias                         | 默认使用此版                                                 |                                    |
| @vue/cli 引入 | 需要额外配置                           | 默认使用此版                                                 |                                    |

# Options

## Data

### data

- 接收内部属性
- 有两种写法：对象和函数，函数 return 一个对象
- 优先使用函数，避免两个组件共用一个 data，此外在 vue 组件中的 data 不能使用对象
- data 有 bug

关于更多 data 与 响应式的内容，可以看看 [我的一篇博客](https://hais-teatime.com/post/2019-12-25-vue-1/)

### props

- 接收外部属性，一个数组
- `message="n"` 传入字符串
- `:message="n"` 传入 this.n 数据
- `:fn="add"` 传入 this.add 函数
- 如果外部有谁用了这个组件，可以在他的 `template` 里面把数据传给这个组件
- 如果传的时候在前面加上冒号，传的就是 JS 代码（变量），**比如要传数字、布尔值、数组、对象等都需要加冒号**
- 子组件 **不能** 修改父组件传来的 prop，子组件如果想要进行修改或转换，最好是定义一个本地的 data，然后把 prop 作为初值赋给 data，或者使用 computed

如果想要将一个对象的所有属性都作为 prop 传入，可以使用不带参数的 `v-bind` 这样写：

```html
<blog-post v-bind="post"></blog-post>
```

可以为 prop 指定类型（`String` `Number` `Boolean` `Array` `Object` `Function` `Date` `Symbol` 或者一个自定义的构造函数）或进行验证：

```js
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }const User = {
     template: '...',
     watch: {
       '$route' (to, from) {
         // 对路由变化作出响应...
       }
     }
   }
})
```



{% note warning %}
`props` 和 `data` 的区别：
如果需要传值，就放在 `props` 里面
{% endnote %}

### computed

可以查看 [在 CodeSandbox 上的这个例子](https://codesandbox.io/s/beautiful-blackburn-5dn48)

```js
new Vue({
  data: {
    user: {
      email: "harvey@example.com",
      nickname: "hai",
      phone: '1234567890',
    }
  },
  computed:{ // 能够将计算而来的属性作为属性
    displayName(){
      const user = this.user
      return user.nickname || user.email || user.phone
    },
    // 或者写成：
    displayName: {
      get(){
        const user = this.user
        return user.nickname || user.email || user.phone
      },
      set(value){
        this.user.nickname = value
      }
    }
  },
  template:`
    <div>
      {{displayName}}
    </div>
  `
}).$mount('#app')
```

{% note warning %}
computed 是有缓存的，如果 computed 依赖的属性没有变化，那么就不会重新计算
而普通的函数（方法），或者 getter / setter 默认是不会做缓存的，Vue 做了特殊处理
{% endnote %}

### watch

简单来说就是当数据变化的时候执行一个函数，可以参考 [这个简单的计算器的例子](https://codesandbox.io/s/dry-architecture-6cp4p)
watch 也可以用来实现 computed 的效果，可以参考 [这个例子](https://codesandbox.io/s/long-dust-7jb3q)


{% note warning %}
什么叫数据变化？其实就是 `===` 的规则，简单类型比较值，引用类型比较地址
`obj.a` 进行修改 → `obj` 没变
`obj` 进行修改 → `obj` 变了
{% endnote %}

```js
watch: {
  obj: {
    handler(){ console.log('changed') },
    deep: true // 这个可以使得监听到内部数据的变化，如果里面变了，那么就触发
  }
}
```

{% note warning %}
不要使用箭头函数来定义 watcher，`this` 是 `window`
{% endnote %}

{% note warning %}
`computed` 和 `watch` 的区别？

- `computed` 是用来计算一个值的，调用的时候不需要加括号，会根据依赖缓存
- `watch` 有两个比较常用的选项：`immediate` 和 `deep`，通常是在需要进行异步或者开销比较大的操作的时候使用
  {% endnote %}

## DOM

### el

- 想要挂载到哪个节点，节点内容会被替换
- 在里面写了内容，基本上是不太可能被用户看见的，除非用户网速特别慢
- 可以用不用 el，替换为 $mount，效果一样：

```js
new Vue({
  render: h => h(demo)
}).$mount('#app')
```

### template

#### 三种写法

1. Vue 完整版，写在 HTML 里面

```html
<div id=xxx>
  {n}}
  <button @click="add">+1</button>
</div>
```

```js
new Vue({
  el: "#xxx",
  data: { n: 0 }, // data 可以改成函数
  methods: { add(){ this.n += 1 } }
})
```

2. Vue 完整版，写在 options 里面

```html
<div id=app></div>
```

```js
new Vue({
  template: `
    <div id=xxx>
      {{n}}
      <button @click="add">+1</button>
    </div>
  `,
  data: { n: 0 }, // data 可以改成函数
  methods: { add(){ this.n += 1 } }
}).$mount('#app') // 注意这个时候 div#app 会被替代
```

3. Vue 运行时版，写在 `.vue` 文件里面

```html
<!-- app.vue-->

<!-- template 不是 HTML，是 XML，有闭合标签-->
<template>
  <div id=xxx>
    {{n}}
    <button @click="add">+1</button>
  </div>
</template>

<script>
  export default {
    data(){ return { n:0 } }, // data 必须为函数
    methods: { add(){ this.n += 1 } }
  }
</script>

<style>
  /* CSS-Code */
</style>
```

```js
// main.js

import App from './App.vue' // App 是一个 options 对象
new Vue({
  render: h => h(App)
}).$mount('#app')
```

{% note warning %}
HTML 与 XML 写法不同：
`<input name="username">` - HTML
`<input name="username"/>` - XML
`<div></div>` - HTML
`<div/>` - XML
{% endnote %}

#### 模板语法

##### 展示内容

###### 表达式

```html
{{ object.a }} <!-- 表达式 -->
{{ n + 1 }} <!-- 运算 -->
{{ fn(n) }} <!-- 调用函数 -->
<div v-text="表达式"></div>
```

{% note warning %}
如果值为 undefined 或 null 就不显示
{% endnote %}

###### HTML 内容

```html
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

`span` 将会被 `rawHtml` 的内容所替换，并且 `rawHtml` 里面的数据绑定将会失效，因此我们不能用 `v-html` 来在 `template` 里面使用 `template`，这个时候需要用 `component` 来进行组件的组合。

###### 纯文本

```html
<!-- v-pre 不会对模板进行编译 -->
<div v-pre>{{ n }}</div>
```

##### 绑定属性

Mustache 语法（双大括号）并不能使用在 HTML 属性上，这时我们需要依靠 `v-bind` 来帮我们绑定属性。

```html
<img v-bind:src="x" />
<!-- 可简写为 -->
<img :src="x" />
<div
  :style="{border: '1px solid red', height: 100}">
这里可以把 '100px' 写成 100
</div>
```

###### 绑定 Class

- 对象语法

```html
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
```

通过设置 `isActive` 和 `hasError` 的值为 `true` 或者 `false` 来控制 `div` 是否有 `active` 和 `text-danger` 这两个 class

- 数组语法

```html
<div v-bind:class="[activeClass, errorClass]"></div>
```

通过给 `activeClass` 和 `errorClass` 赋值来控制 `div` 的 class

可以在数组语法中使用三元表达式：

```html
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```

这样将始终添加 `errorClass` 里面的值，但是只有在 `isActive` 为真时才添加 `activeClass` 里面的值

也可以在数组语法中使用对象语法：

```html
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```

给一个自定义组件加上 class 的话，会自动渲染到他的最外面的那个元素上面，并且最外面元素自身的 class 不会被覆盖，同样也支持对象语法等

###### 绑定 Style

- 对象语法

```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

可以为 `activeColor` 和 `fontSize` 赋值，来改变 Style

- 数组语法

```html
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

数组中可以装下多个样式对象

##### 绑定事件

```html
<button v-on:click="add">+1</button>
<!-- 点击之后运行 add()，自动加括号 -->

<button v-on:click="add(1)">+1</button>
<!-- 点击之后运行 add(1)，不自动加括号 -->

<button v-on:click="n+=1">+1</button>
<!-- 点击之后运行 n+=1 -->

<button @click="add">+1</button>
<!-- 可以缩写 -->
```

可以通过 `$event` 传递原始的 DOM 事件“

```html
<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```

##### 指令

- v- 开头的就是指令： `v-指令名:参数=值`，比如 `v-on:click=add`
- 如果值里面没有特殊字符，可以不加引号
- 有的指令没有参数和值，比如 `v-pre`
- 有的指令没有值，比如 `v-on:click.prevent`，阻止默认动作

###### 动态参数

可以用方括号括起来的 JavaScript 表达式作为一个指令的参数：

```html
<a v-bind:[attributeName]="url"> ... </a>
<!-- attributeName 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用 -->
```

###### 修饰符

有的指令支持修饰符，比如：

```
@click.stop="add" 阻止冒泡
@click.prevent="add" 阻止默认动作
@click.stop.prevent="add"
@keypress.13 当按下回车时执行，或者用 @keypress.enter，有很多都有别名
```

- .sync 修饰符

查看 [这个在 CodeSandbox 上的例子](https://codesandbox.io/s/kind-cookies-1o6rs)，或者 [这个例子](https://codesandbox.io/s/kind-cookies-1o6rs)

- 组件不能修改 `props` 外部数据
- `$emit` 可以触发一个事件，并传参（发布）， `v-on` 可以监听他（订阅）
- `$event` 可以获取 `$emit` 的参数
- 由于经常会出现让组件想要更新数据的情况，所以有了 `.sync`

```
:money.sync="total"
等价于
:money="total" v-on:update:money="total = $event"
update:money 是事件名
```

##### 条件渲染

###### v-if

```html
<div v-if="x > 0">
  x 大于 0
</div>
<div v-else-if="x === 0">
  x 为 0
</div>
<div v-else>
  x 小于 0
</div>
```

有时候我们需要加上 `key` 来管理元素的复用

```html
<template v-if="loginType === 'username'">
  <label>Username</label>
<!-- 因为 label 没有 key，所以 label 复用了，仅仅是替换掉了文字 -->
  <input placeholder="Enter your username" key="username-input">
<!-- 如果没有 key，那么切换的时候 input 里面的内容将会保留，这就是元素的复用 -->
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```

###### v-show

```html
<div v-show="n % 2 === 0"> n 是偶数 </div>
<!-- 近似等于 -->
<div :style="{display: n%2===0 ? 'block' : 'none'}"> n 是偶数 </div>
<!-- 但是注意，不是所有看得见的元素 display 都是 block -->
<!-- table 是 table，li 是 list-item -->
```

{% note warning %}
`v-if` 与 `v-show`

- `v-show` 不支持 `<template>` 元素，也不支持 `v-else`，`v-if` 则都支持
- `v-if` 在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建，`v-show` 则只是切换了 `display` 属性
- 若初始条件为假，`v-if` 不会渲染，`v-show` 则会渲染
  {% endnote %}

##### 列表渲染

```html
<ul>
  <li v-for="(user, index) in users" :key="index">
    索引: {{index}} 值：{{user.name}}
  </li>
</ul>

<ul>
  <li v-for="(value, name, index) in obj" :key="name">
    属性名: {{name}} 属性值：{{value}} 索引: {{index}}
  </li>
</ul>
```

{% note warning %}
`:key="index"` 有 Bug， `v-for` 一定要有 `:key`，尽量要是不重合的值
{% endnote %}

当 `v-if` 和 `v-for` 同时使用的时候， `v-for` 的优先级会更高，比如：

```html
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```

上面的代码将只渲染未完成的 todo

在组件上使用 `v-for` 时需要手动将数据给传进去：

```html
<my-component
  v-for="(item, index) in items"
  v-bind:item="item"
  v-bind:index="index"
  v-bind:key="item.id"
></my-component>
```

##### 双向绑定

`v-model` 可以实现常用表单元素的双向绑定，包括文本、多行文本、复选框、单选按钮、选择框等

```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

相当于这样的简写：

```html
<input type="text" :value="message"
       @input="message = $event.target.value">
```

## Lifecycle Hooks

### Vue Lifecycle

![](https://cn.vuejs.org/images/lifecycle.png)

在调用每个生命周期钩子的时候，已经完成了哪些事情：

1. **beforeCreate**：创建一个新的 Vue 实例，初始化事件与生命周期
2. **created**：初始化数据，进行数据的观测（比如将使用 `Object.defineProperty` 改造 data，并将 vm 作为代理）
3. **beforeMount**：将模板编译为 render 函数，将 v-if 变为 if、v-for 变为 map 等
4. **mounted**：给 vm 添加 $el 成员，并且替换掉挂载的 DOM 元素
5. **updated**：数据发生改变，触发组件更新，将会使用新的数据构造一份新的 DOM，替换掉原来的
6. **destroyed**：vm 指示的所有东西都会解绑，所有的事件监听器被移除，所有的子实例被销毁

对于父子组件：

1. 加载渲染过程
   父 BeforeCreate -> 父 Created -> 父 BeforeMount -> 子 BeforeCreate -> 子 Created -> 子 BeforeMount -> 子 Mounted -> 父 Mounted
2. 子组件更新过程
   父 BeforeUpdate -> 子 BeforeUpdate -> 子 Updated -> 父 Updated
3. 父组件更新过程
   父 BeforeUpdate -> 父 Updated
4. 销毁过程
   父 BeforeDestroy -> 子 BeforeDestroy -> 子 Destroyed -> 父 Destroyed

## Assets

### directives

#### 指令声明

可以查看 [在 CodSandbox 上的这个例子](https://codesandbox.io/s/dank-cloud-99drb)

声明全局指令：

```js
Vue.directive('x', directiveOptions)
// 然后就可以全局使用 v-x 了
```

声明局部指令：

```js
directives: {
  x: { derectiveOptions }
}
// 只能在当前组件用 v-x，其子组件也不能用
```

#### directiveOptions

```js
bind(el, info, vnode, oldVnode) // 类似 created
inserted(el, info, vnode, oldVnode) // 类似 mounted
update(el, info, vnode, oldVnode) // 类似 updated
componentUpdated(el, info, vnode, oldVnode)
unbind(el, info, vnode, oldVnode) // 类似 destroyed
```

#### 指令的作用

- **用于 DOM 操作**
  - Vue 的实例/组件主要用于数据绑定、事件监听、DOM 更新；Vue 指令主要目的是原生 DOM 操作
- **减少重复**
  - 如果某个 DOM 操作经常使用/很复杂，可以封装成指令

### filters

尽量不用，用 methods

### components

```js
// 可以引入一个 vue 文件，叫做 Demo，作为 frank 组件（局部注册）
// 优先使用这种，但是局部注册的组件在其子组件中不可用

import Demo from './Demo.vue'
new Vue({
  components: {
    frank: Demo, // 如果是 Demo: Demo，就可以简化为 Demo
  },
})

// 也可以新建一个组件，后面接受的参数与 new Vue(options) 里面的 options 一样（除了没有 el），这叫全局组件

Vue.component('my-component-name', options)

// 也可以这样写，options 也跟外面的一样（除了没有 el）

new Vue({
  components: {
    frank: options,
  },
})
```

- 组件是一个抽象概念
- 文件名没有特殊规定，如果非常讲究，可以全用小写，因为比如 Windows 10 就不分大小写
- 组件名推荐首字母大写

## Composition

### mixins

目的：减少 data、methods、钩子的重复，可以查看在 [Codesandbox 上的这个例子](https://codesandbox.io/s/lingering-microservice-6cw8h)
选项会智能合并：data、钩子等都会合并，在冲突的时候优先使用组件自己的

### extends

```js
const MyVue = Vue.extend(options)
// 然后就可以用
new MyVue(options)

// 或者
import MyVue from "../MyVue.js";
export default {
  extends: MyVue
}
```

可以查看在 [CodeSandbox 上的这个例子](https://codesandbox.io/s/amazing-frog-f9xz5)

### provide && inject

用于大范围的数据共用，简单数据类型不能直接更改，需要通过函数，传过去的函数的 this 会自动绑定到 Provider 上面，可以查看在 [Codesandbox 上的这个例子](https://codesandbox.io/s/affectionate-jennings-v4y1t)

# Component

1. 用 JS 对象

```js
export default {
  options
}
```

2. 使用 TS 类

```typescript
import Vue from 'source/_posts/Others-QA-Vue';
import {Component, Prop} from 'vue-property-decorator';

@Component({
  // components: {...}
})
export default class MyComponent extends Vue {
  num: number = 0;
  @Prop(Number) propA: number | undefined;
  add(num: number) {
    this.num = num ++;
  }
  mounted() {
    console.log(this.propA);
  }
}
```

3. 使用 JS 类

```js
import Vue from 'vue';
import {Component, Prop} from 'vue-property-decorator';

@Component
export default class MyComponent extends Vue {
  num = 0;
  @Prop(Number) propA
  add(num) {
    this.num = num ++;
  }
  mounted() {
    console.log(this.propA);
  }
}
```

# Vue Router

## 基本使用

### HTML

`<router-link/>` 就像 `<a>` 标签，来链接到不同的路由

```html
<router-link to="/foo">Go to Foo</router-link>
```

`<router-view/>` 用来装路由匹配出来的内容

```html
<router-view/>
```

### JavaScript

```js
// 0. 引入 VueRouter
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

// 1. 引入要被路由的组件
import Foo from '../Foo.vue'
import Bar from '../Bar.vue'

// 2. 定义路由，相当于定义 router 的配置选项
const routes = [
  {path: '/foo', component: Foo},      
  {path: '/bar', component: Bar}      
]

// 3. 创建 router 实例
const router = new VueRouter({
  routes  
})

// 4. 创建根实例，并使用 VueRouter
const App = new Vue({
    router
}).$mount('#app')
```

## 路由匹配

| 模式                          | 匹配路径            | $route.params                          |
| ----------------------------- | ------------------- | -------------------------------------- |
| /user/:username               | /user/evan          | `{ username: 'evan' }`                 |
| /user/:username/post/:post_id | /user/evan/post/123 | `{ username: 'evan', post_id: '123' }` |

{% note warning %}
当使用路由参数时，例如从 `/user/foo` 导航到 `/user/bar`，原来的组件实例会被复用，组件的生命周期钩子不会再被调用
{% endnote %}

如果想要对路由参数的变化做出响应，需要 watch $route 对象：

```js
const User = {
  template: '...',
  watch: {
    '$route' (to, from) {
      // 对路由变化作出响应...
    }
  }
}
```

或者使用 beforeRouterUpdate：

```js
const User = {
  template: '...',
  beforeRouteUpdate (to, from, next) {
    // react to route changes...
    // don't forget to call next()
  }
}
```

## 嵌套路由

一个被渲染组件同样可以包含自己的嵌套 `<router-view>`，此时要在 `VueRouter` 的参数中增加 `children` 的配置：

```js
const router = new VueRouter({
  routes: [
    { path: '/user/:id', component: User,
      children: [
        {
          // 当 /user/:id 匹配成功，
          // UserHome 会被渲染在 User 的 <router-view> 中
          path: '',
          component: UserHome
        },
        {
          // 当 /user/:id/profile 匹配成功，
          // UserProfile 会被渲染在 User 的 <router-view> 中
          path: 'profile',
          component: UserProfile
        },
        {
          // 当 /user/:id/posts 匹配成功
          // UserPosts 会被渲染在 User 的 <router-view> 中
          path: 'posts',
          component: UserPosts
        }
      ]
    }
  ]
})
```

{% note warning %}
以 / 开头的嵌套路径会被当作根路径
{% endnote %}

## 编程式导航

### `router.push(location, onComplete?, onAbort?)`

相当于 `<router-link :to="...">`，两者的语法规则相同

{% note warning %}
在 Vue 实例里面，可以通过 $router 访问路由实例 `router`。因此可以调用 this.$router.push
{% endnote %}

参数可以是字符串路径或者一个对象：

```js
// 字符串
router.push('home')

// 对象
router.push({ path: 'home' })

// 命名的路由
router.push({ name: 'user', params: { userId: '123' }})

// 带查询参数，变成 /register?plan=private
router.push({ path: 'register', query: { plan: 'private' }})
```

```js
const userId = '123'
router.push({ name: 'user', params: { userId }}) // -> /user/123
router.push({ path: `/user/${userId}` }) // -> /user/123
// 提供了 path 之后 params 不生效
router.push({ path: '/user', params: { userId }}) // -> /user
```

`onComplete` 和 `onAbort` 参数可以传入两个回调：将会在导航成功完成 (在所有的异步钩子被解析之后) 或终止 (导航到相同的路由、或在当前导航完成之前导航到另一个不同的路由) 的时候进行相应的调用。

{% note warning %}
如果目的地和当前路由相同，只有参数发生了改变 (比如从一个用户资料到另一个 /users/1 -> /users/2)，你需要使用 beforeRouteUpdate 来响应这个变化 (比如抓取用户信息)
{% endnote %}

### `router.replace(location, onComplete?, onAbort?`

相当于 `router-link :to="..." replace`，不会向 history 添加新的记录，而是替换掉当前的 history 目录

{% note warning %}
跟 `push` 的主要区别就是 `replace` 可以回去，`push` 则不能
{% endnote %}

### `router.go(n)`

类似 `window.history.go(n)` 意为在 history 记录中前进或后退多少步

```js
// 在浏览器记录中前进一步，等同于 history.forward()
router.go(1)

// 后退一步记录，等同于 history.back()
router.go(-1)

// 前进 3 步记录
router.go(3)

// 如果 history 记录不够用，那就默默地失败呗
router.go(-100)
router.go(100)
```

## 命名视图

```html
<router-view class="view one"></router-view>
<router-view class="view two" name="a"></router-view>
<router-view class="view three" name="b"></router-view>
```

```js
const router = new VueRouter({
  routes: [
    {
      path: '/',
      components: {
        default: Foo,
        a: Bar,
        b: Baz
      }    
    }
  ]  
})
```

## 导航守卫

### 全局前置导航守卫 `router.beforeEach`

```js
const router = new VueRouter({ ... })

router.beforeEach((to, from, next) => {
  // ...
})
```

{% note warning %}
守卫是异步解析执行，此时导航在所有守卫 resolve 完之前一直处于 等待中
确保要调用 next 方法，否则钩子就不会被 resolved
{% endnote %}

- `next()`: 进行管道中的下一个钩子。如果全部钩子执行完了，则导航的状态就是 confirmed (确认的)。

- `next(false)`: 中断当前的导航。如果浏览器的 URL 改变了 (可能是用户手动或者浏览器后退按钮)，那么 URL 地址会重置到 `from` 路由对应的地址。

- `next('/')` 或者 `next({ path: '/' })`: 跳转到一个不同的地址。当前的导航被中断，然后进行一个新的导航。你可以向 `next` 传递任意位置对象，且允许设置诸如 `replace: true`、`name: 'home'` 之类的选项以及任何用在 `router-link` 的 `to` prop 或 `router.push` 中的选项。

- `next(error)`: 如果传入 `next` 的参数是一个 `Error` 实例，则导航会被终止且该错误会被传递给 `router.onError()` 注册过的回调。

### 全局解析守卫 `router.beforeResolve`

类似 `router.beforeEach`，在导航被确认之前，同时在所有组件内守卫和异步路由组件被解析之后，解析守卫就被调用

### 全局后置钩子 `router.afterEach`

```js
router.afterEach((to, from) => {
  // ...
})
```

### 路由独享的守卫 `beforeEnter`

```js
const router = new VueRouter({
  routes: [
    {
      path: '/foo',
      component: Foo,
      beforeEnter: (to, from, next) => {
        // ...
      }
    }
  ]
})
```

### 组件内的守卫

```js
const Foo = {
  template: `...`,
  beforeRouteEnter (to, from, next) {
    // 在渲染该组件的对应路由被 confirm 前调用
    // 不！能！获取组件实例 `this`
    // 因为当守卫执行前，组件实例还没被创建
    // 但是可以通过传给 next 一个回调来访问组件实例          
    next(vm => {
      // 通过 `vm` 访问组件实例
    })
  },
  beforeRouteUpdate (to, from, next) {
    // 在当前路由改变，但是该组件被复用时调用
    // 举例来说，对于一个带有动态参数的路径 /foo/:id，在 /foo/1 和 /foo/2 之间跳转的时候，
    // 由于会渲染同样的 Foo 组件，因此组件实例会被复用。而这个钩子就会在这个情况下被调用。
    // 可以访问组件实例 `this`
  },
  beforeRouteLeave (to, from, next) {
    // 导航离开该组件的对应路由时调用
    // 可以访问组件实例 `this`
    // 通常用来禁止用户在还未保存修改前突然离开，通过 next(false) 来取消用户的导航    
  }
}
```

### 导航的解析流程

1. 导航被触发
2. 在失活的组件里调用离开守卫
3. 调用全局的 `beforeEach` 守卫
4. 在重用的组件里调用 `beforeRouteUpdate` 守卫 (2.2+)
5. 在路由配置里调用 `beforeEnter`
6. 解析异步路由组件
7. 在被激活的组件里调用 `beforeRouteEnter`
8. 调用全局的 `beforeResolve` 守卫
9. 导航被确认
10. 调用全局的 `afterEach` 钩子
11. 触发 DOM 更新
12. 用创建好的实例调用 `beforeRouteEnter` 守卫中传给 `next` 的回调函数

# Vue

可以先看一下 [这个例子](https://codesandbox.io/s/quizzical-kalam-303z8)

## 基本使用

Vuex 的核心就是一个状态仓库，与普通的全局对象有所不同：

1. Vuex 的状态存储是响应式的，store 变化 -> 组件响应式更新
2. 不能直接修改 store 中的状态，只能通过提交 mutation 的方式来改变状态

```js
// 1. 引入 Vuex
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

// 2. 创建一个 store
const store = new Vuex.Store({
  // 定义一个初始的状态
  state: {
    count: 0
  },
  // 定义一些 mutation 来操作状态的变更
  mutation: {
    increment (state) {
      state.count++
    }
  }
})

// 3. 可以通过 store.state 获取对象，或者通过 store.commit 来触发变更
store.commit('increment')

console.log(store.state.count) // -> 1
```

## State

Vuex 使用单一状态树，每个应用仅仅包含一个 store 实例

### 在 Vue 组件中获得 Vuex 状态

#### 方法一：通过计算属性

```js
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

这样每个需要使用 state 的组件中就都需要导入 store

#### 方法二：将状态从根组件注入每个子组件

```js
const app = new Vue({
  el: '#app',
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter/>
    </div>
  `
})
```

这样子组件就可以通过 `this.$store` 访问到

```js
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count() {
      return this.$store.state.count  
    }
  }
}
```

#### 方法三：`mapState`

```js
import { mapState } from 'vuex'

export default {
  computed: mapState({
    // 可以使用箭头函数
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',
    
    // 也可以使用普通函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

若计算属性的名称与子节点名称相同，的也可以给 mapState 传一个数组

```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```

#### 将 mapState 与局部计算属性混用

```js
computed: {
  localComputed () { /* ... */ },
  // 使用对象展开运算符将此对象混入到外部对象中
  ...mapState({
    // ...
  })
}
```

## Getter

从 store 的 state 中派生出一些状态，类似于 store 的计算属性

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    // 可以传入 state 作为第一个参数
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    },
    // 可以传入其他 getter 作为第二个参数
    doneTodosCount: (state, getters) => {
      return getters.doneTodos.length
    },
    // 可以让 getter 返回一个函数来实现给 getter 传参
    getTodoById: (state) => (id) => {
      return state.todo.find(todo => todo.id === id)
    }
    // 到时候通过 store.getters.getTodoById(2) 这样访问
    // 这样通过方法访问就不会有缓存
  }
})
```

可以像访问 State 访问这些 Getter

```js
store.getters.doneTodos // -> [{ id: 1, text: '...', done: true }]

// 或者在组件中
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}

// 或者通过 mapGetters
computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
  ...mapGetters([
  'doneTodosCount',
  'anotherGetter',
  // ...
  ])
}
```

## Mutation

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    // 将 state 作为第一个参数
    increment (state) {
      // 变更状态
      state.count++
    },
    // 将 payload 作为第二个参数，大多数情况下 payload 应该传一个对象
    decrement (state, payload) {
      state.count -= payload.amount
    }
  }
})
```

然后这样提交

```js
store.commit('increment')
store.commit('decrement', {
  amount: 10
})

// 或者写成含有 type 属性的对象
sotre.commit({
  type: 'increment',
  amount: 10
})

// 或者在组件中提交
methods: {
  increment () {
    this.$store.commit('increment')
  },
  // 或者使用 mapMutations
  ...mapMutations([
    'increment' // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
  ]),
  ...mapMutations({
    add: 'increment' // 将 `this.add()` 映射为 `this.$store.commit('increment')`
  })
}
```

{% note warning %}
Mutation 必须是同步函数，因为异步会使得很难调试，无法区分谁先回调
{% endnote %}

## Action

Action 类似于 mutation，不同在于：

- Action 提交的是 mutation，而不是直接变更状态
- Action 可以包含任意异步操作

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  // 接受一个与 store 实例具有相同方法和属性的 context 对象
  // 这个 context 对象也有 context.state context.getters 这种方法
  actions: {
    increment (context) {
      context.commit('increment')
    }
  },

  // 可以这样简化代码：
  action: {
    incrementAsync ({ commit }) {
      setTimeout(() => {
        commit('increment')
      }, 1000)
    }
  }
})
```

然后这样触发，同 Mutation 一样，也支持 payload 和含有 type 的对象的形式

```js
store.dispatch('incrementAsync')

// 或者在组件中触发
methods: {
  increment () {
    this.$store.dispatch('incrementAsync')
  },
  // 或者使用 mapActions
  ...mapActions([
    'incrementAsync' // 将 `this.increment()` 映射为 `this.$store.dispatch('incrementAsync')`
  ]),
  ...mapActions({
    add: 'incrementAsync' // 将 `this.add()` 映射为 `this.$store.dispatch('incrementAsync')`
  })
}
```

## Module

```js
const moduleA = {
  state: { ... },
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: { ... },
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> moduleA 的状态
store.state.b // -> moduleB 的状态
```

