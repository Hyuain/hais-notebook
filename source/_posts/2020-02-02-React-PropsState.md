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

Vue：
- 一个对象，对应一个虚拟 DOM，当对象的属性改变时，把属性相关的 DOM 节点全部更新
- JS in HTML

React：
- 一个对象，对应一个虚拟 DOM，另一个对象对应另一个虚拟 DOM，对比两个更新，用 DOM Diff 算法找不同，然后局部更新 DOM
- HTML in JS
