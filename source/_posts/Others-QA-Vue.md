---
title: FAQs：Vue
date: 2020-02-13 21:17:05
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [前端, 其他]
---

关于 Vue 的一些问题与答案。

<!-- more -->

# Vue 的生命周期

![](https://cn.vuejs.org/images/lifecycle.png)

在调用每个生命周期钩子的时候，已经完成了哪些事情：

1. **beforeCreate**：创建一个新的 Vue 实例，初始化事件与生命周期
2. **created**：初始化数据，进行数据的观测（比如将使用 `Object.defineProperty` 改造 data，并将 vm 作为代理）
3. **beforeMount**：将模板编译为 render 函数，将 v-if 变为 if、v-for 变为 map 等
4. **mounted**：给 vm 添加 $el 成员，并且替换掉挂载的 DOM 元素
5. **updated**：数据发生改变，触发组件更新，将会使用新的数据构造一份新的 DOM，替换掉原来的
6. **destroyed**：vm 指示的所有东西都会解绑，所有的事件监听器被移除，所有的子实例被销毁

# `v-if` 和 `v-show` 的区别？

- `v-if` 是真正的条件渲染，涉及事件监听器和子组件的销毁和重建过程；而且是惰性的，若初始条件为假，`v-if` 不会渲染
- `v-show` 仅仅是 `display` 属性的改变，不支持 `<template>` 元素

# `watch` 和 `computed` 的区别？

- `computed` 是用来计算一个值的，调用的时候不需要加括号，会根据依赖缓存
- `watch` 有两个比较常用的选项：`immediate` 和 `deep`，通常是在需要进行异步或者开销比较大的操作的时候使用

# Vue 组件间的通信？

1. 父子组件：`$emit` 和 `$on`
2. 爷孙组件、兄弟组件：eventBus
3. Vuex

# Vuex？

Vuex 是一个专为 Vue.js 应用程序开发的状态管理工具，分别有几个核心概念：State、Getter、Mutation、Action、Module。

# Vue 的数据响应式是如何做到的？

# VueRouter？

VueRouter 是 Vue.js 官方的路由管理器