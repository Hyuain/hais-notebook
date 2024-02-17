---
title: React
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - F1
---

React 是一个前端框架，本笔记并不会包含 React 的全解，因为官方文档已经足够详细，并且在不断更新。本笔记主要包括以下内容：

- [[React UI|React 中如何描述 UI]]
	- [[JSX]]
	- [[React Elements & Components|React 中的 Elements 和 Components]]
	- [[React UI|React 对 UI 的变更的处理]]
- [[React State Management|React 如何处理状态变更]]
- [[React Refs]]
- [[React Effects]]
- [[React Hooks]]
- [[React Class Components|DEPRECATED: React Class Components]]

# React Source Code Debugging

Clone React 项目之后，在 README 中找到调试 React 的方法，具体可参照 [How to Contribute](https://legacy.reactjs.org/docs/how-to-contribute.html)。然后按照文中安装依赖并 Build 之后，将 `fixture/packaging/babel-standalone/dev.html` 作为我们的 Playground，对 React 源码进行调试。

