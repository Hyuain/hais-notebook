---
title: React Props & State
date: 2020-02-02 16:54:25
tags:
  - 入门
  - 饥人谷
  - 文档
categories:
  - [前端, JavaScript, 框架, React]
---

主要记录了 React 的 Props 和 State。

<!-- more -->

# Props

可以查看[CodeSandbox 上的这个例子](https://codesandbox.io/s/billowing-wind-d8kzw)

# State

可以查看[CodeSandbox 上的这个例子](https://codesandbox.io/s/silly-diffie-9yk38)

## setState 的注意事项

- `this.state.n += 1` 无效，UI 不会自动更新，需要用 `setState`
- `setState` 不会马上改变 `state`，是异步更新的，推荐使用 `setState(函数)`
- 不推荐 `this.setState(this.state)`，因为 React 不推荐我们修改旧的 `state`（不可变数据）

## 复杂 state

- 类组件的 `setState` 会自动合并第一层，建议使用 `Object.assign` 或者 `...sate`
- 函数组件不会自动合并，建议分开写

## Vue 和 React 的区别

共同点：
- 都是对视图的封装，React 是用类和函数表示一个组件，Vue 是用构造选项表示一个组件
- 都提供了 `creatElement` 的 XML 简写，React 是 JSX，Vue 是 template

不同点：
1. 监听数据变化的实现原理不同
    - Vue 通过 getter/setter 和代理等方式进行劫持，能够精确知道数据的变化，不需要特别的优化
    - React 默认是通过比较引用的方式来进行的，可能会造成不必要的 vDOM 重新渲染，需要用诸如 PureComponent/shouldComponentUpdate 等方式来进行优化
2. 数据流不同
    - Vue 在 1.0 的时候 **父子组件** props 可以双向绑定，还有 v-model 可以实现 **组件与 DOM** 双向绑定
    - Vue 2 的时候 **父子组件** 不能双向绑定了（虽然仍然提供了 .sync 语法糖），并且还有 v-model
    - React 中是单向数据流，并且组件与 DOM 之间也需要使用 onChange/setState。
3. HoC 和 mixins
    - Vue 中使用不同功能的组合是通过 mixins 实现的
    - React 中则使用 HoC
4. 组件间的通信
    - 都有 props，也都可以跨层级通信，比如 Vue 的 provide/inject、React 的 context
    - React 本身不支持自定义事件，因此 Vue 里面一般使用事件，React 中一般使用父组件传来的回调函数
5. 更新视图
    - Vue 中一个对象，对应一个虚拟 DOM，当对象的属性改变时，把属性相关的 DOM 节点全部更新
    - React 一个对象，对应一个虚拟 DOM，另一个对象对应另一个虚拟 DOM，对比两个更新，用 DOM Diff 算法找不同，然后局部更新 DOM
6. 写法不同
    - Vue 是 JS in HTML
    - React 是 HTML in JS
7. Vuex 和 Redux 的区别
    - Vue 中的 $store 直接注入到了组件实例中，可以直接用 dispatch、commit、mapState、this.$store 等；React 中需要用 connect 把 props 和 dispatch、state 连接起来
    - Vue 中可以用 dispatch action、commit mutation；React 中只能使用 dispatch，不能直接操作 reduce
    - Redux 中的使用的是不可变数据，每次都要用新的 State 替换旧的，而 Vue 中是直接修改的