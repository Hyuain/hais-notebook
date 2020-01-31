---
title: Vue 选项
date: 2020-01-30 10:35:47
tags:
  - 入门
  - 饥人谷
  - 文档
categories:
  - [前端, JavaScript, 框架, Vue]
---

记录 Vue Options 中的内容。

<!-- more -->

# Data

## data

- 接收内部属性
- 有两种写法：对象和函数，函数 return 一个对象
- 优先使用函数，避免两个组件共用一个 data，此外在 vue 组件中的 data 不能使用对象
- data 有 bug

关于更多 data 与 响应式的内容，可以看看 [我的一篇博客](https://hais-teatime.com/post/2019-12-25-vue-1/)

## props

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
  }
})
```



{% note warning %}
props 和 data 的区别：
如果需要传值，就放在 props 里面
{% endnote %}

## computed

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

## watch

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

> computed 和 watch 的区别？
> computed 是用来计算一个值的，调用的时候不需要加括号，会根据依赖缓存
> watch 有两个比较常用的选项：`immediate` 和 `deep`
  
  watch 有两个选项，immediate，deep

# DOM

## el

- 想要挂载到哪个节点，节点内容会被替换
- 在里面写了内容，基本上是不太可能被用户看见的，除非用户网速特别慢
- 可以用不用 el，替换为 $mount，效果一样：

```js
new Vue({
  render: h => h(demo)
}).$mount('#app')
```

## template

### 三种写法

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

### 模板语法

#### 展示内容

##### 表达式

```html
{{ object.a }} <!-- 表达式 -->
{{ n + 1 }} <!-- 运算 -->
{{ fn(n) }} <!-- 调用函数 -->
<div v-text="表达式"></div>
```

{% note warning %}
如果值为 undefined 或 null 就不显示
{% endnote %}

##### HTML 内容

```html
<p>Using v-html directive: <span v-html="rawHtml"></span></p>
```

`span` 将会被 `rawHtml` 的内容所替换，并且 `rawHtml` 里面的数据绑定将会失效，因此我们不能用 `v-html` 来在 `template` 里面使用 `template`，这个时候需要用 `component` 来进行组件的组合。

##### 纯文本

```html
<!-- v-pre 不会对模板进行编译 -->
<div v-pre>{{ n }}</div>
```

#### 绑定属性

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

##### 绑定 Class

###### 对象语法

```html
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>
```

通过设置 `isActive` 和 `hasError` 的值为 `true` 或者 `false` 来控制 `div` 是否有 `active` 和 `text-danger` 这两个 class

###### 数组语法

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

##### 绑定 Style

###### 对象语法

```html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```

可以为 `activeColor` 和 `fontSize` 赋值，来改变 Style

###### 数组语法

```html
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

数组中可以装下多个样式对象

#### 绑定事件
   
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

#### 指令
   
- v- 开头的就是指令： `v-指令名:参数=值`，比如 `v-on:click=add`
- 如果值里面没有特殊字符，可以不加引号
- 有的指令没有参数和值，比如 `v-pre`
- 有的指令没有值，比如 `v-on:click.prevent`，阻止默认动作

##### 动态参数

可以用方括号括起来的 JavaScript 表达式作为一个指令的参数：

```html
<a v-bind:[attributeName]="url"> ... </a>
<!-- attributeName 会被作为一个 JavaScript 表达式进行动态求值，求得的值将会作为最终的参数来使用 -->
```
   
##### 修饰符

有的指令支持修饰符，比如：

```
@click.stop="add" 阻止冒泡
@click.prevent="add" 阻止默认动作
@click.stop.prevent="add"
@keypress.13 当按下回车时执行，或者用 @keypress.enter，有很多都有别名
```

###### .sync 修饰符

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

#### 条件渲染

##### v-if

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

##### v-show

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

#### 列表渲染

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

#### 双向绑定

`v-model` 可以实现常用表单元素的双向绑定，包括文本、多行文本、复选框、单选按钮、选择框等

```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

# Lifecycle Hooks

> 出现的时机可以用 debugger 证明

- beforeCreate
- created：出现在内存中，没有出现在页面中
- beforeMount
- mounted：出现在页面中
- beforeUpdate
- updated：更新了
- activated
- deactivated
- beforeDestroy
- destroyed：消亡了再弄出来之后是新的组件，占用新的内存地址
- errorCaptured

# Assets

## directives

### 指令声明

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

### directiveOptions

```js
bind(el, info, vnode, oldVnode) // 类似 created
inserted(el, info, vnode, oldVnode) // 类似 mounted
update(el, info, vnode, oldVnode) // 类似 updated
componentUpdated(el, info, vnode, oldVnode)
unbind(el, info, vnode, oldVnode) // 类似 destroyed
```

### 指令的作用

- **用于 DOM 操作**
    - Vue 的实例/组件主要用于数据绑定、事件监听、DOM 更新；Vue 指令主要目的是原生 DOM 操作
- **减少重复**
    - 如果某个 DOM 操作经常使用/很复杂，可以封装成指令

## filters

尽量不用，用 methods

## components

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



# Composition

## mixins

目的：减少 data、methods、钩子的重复，可以查看在 [Codesandbox 上的这个例子](https://codesandbox.io/s/lingering-microservice-6cw8h)
选项会智能合并：data、钩子等都会合并，在冲突的时候优先使用组件自己的

## extends

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

## provide && inject

用于大范围的数据共用，简单数据类型不能直接更改，需要通过函数，传过去的函数的 this 会自动绑定到 Provider 上面，可以查看在 [Codesandbox 上的这个例子](https://codesandbox.io/s/affectionate-jennings-v4y1t)

# Misc