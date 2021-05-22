---
title: 翻译：Angular 如何检查变化
date: 2021-05-17 18:56:06
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, Angular]
---

翻译文章 [Angular Change Detection - How Does It Really Work?](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/)。

<!-- more -->

# 目录

- 变化检查是如何进行的？
- Angular 变化检查器是什么样的？
- 默认的变化检查机制是怎样进行的
- 开启/关闭变化检查，手动触发他
- 避开变化检查循环：生产模式和开发模式
- `OnPush` 变化检查模式究竟做了什么？
- 使用 Immutable.js 来简化 Angular 应用的构建
- 总结

如果想要了解更多关于 OnPush 变化检查机制，请查阅 [这篇文章](https://blog.angular-university.io/onpush-change-detection-how-it-works)

# 变化检查是如何实现的

当组件的数据变化的时候，Angular 可以探测到，并重新渲染视图来相应这次变化。为了理解工作原理，我们需要首先意识到 JavaScript 中整个运行时都是可重写的。我们可以重写类似于 `String` 或 `Number` 等方法。
