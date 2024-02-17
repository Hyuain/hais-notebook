---
title: React.createElement
date: 2020-02-02 16:25:06
categories:
  - - 前端
tags:
  - FX
---

`<div/>` 会被翻译成 `React.createElement('div')`

`<Welcome/>` 会被翻译成 `React.createElement(Welcome)`

`React.createElement` 做了下面这些事情：

- 如果传入一个字符串 `'div'` ，则会创建一个 `div`
- 如果传入一个函数，则会调用该函数，获取其返回值
- 如果传入一个类，则会在前面加类前面加 `new` （执行 constructor），获取一个组件的对象，然后调用对象的 render 方法，获取其返回值

