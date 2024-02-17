---
title: React UI
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

不同的前端框架为开发者提供了不同的描述 UI 的方式，比如 [[Vue]] 和 [[Angular]] 的模板语法，以及 [[React]] 的 [[JSX]]。

形如 `<div>{user.name}</div>` 的 JSX 代码会在编译阶段被 Babel 转换为对 [[React.createElement]] 的调用，而该函数会在运行时返回 [[React Elements & Components|React Elements]]，这些 React Elements 会被进一步转换为 DOM 节点，最终呈现在页面上。

在上述代码中，倘若 `user` 或 `user.name` 由于用户操作或网络请求等发生了变化，那么页面上 DOM 节点也需要进行对应的变化。这个从状态更新到 UI 变化的过程，被 React 称为 [[React Reconciliation|Reconciliation]]。

此外，React 基于 JSX 提供了[类似于 Vue 中 slot 的组合方式](https://react.dev/learn/passing-props-to-a-component#passing-jsx-as-children)，也提供了 [[React Fragments|Fragments]] 等组件来优化代码结构。