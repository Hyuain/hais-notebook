---
title: JavaScript Variable Declaration
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 中有 3 种变量声明的方式：`var` `let` 和 `const`。

<!-- more -->

# var

- 过时的，不好的
- 函数作用域
- 可以重复声明
- 可以先使用再声明
- 全局声明的 `var` 变量会变成 `window` 的属性

# let

- 新的，更合理的
- 遵循块作用域
- 不能重复声明
- 可以赋值，也可以不赋值
- 必须先声明再使用
- 全局声明的 `let` 变量不会再变成 `window` 的属性
- `let` 配合 `for` 循环有奇效

# const

跟 `let` 几乎一样，但声明时必须赋值，且不能再更改

{% note warning %}
变量声明指定值的时候同时也指定了类型，但是 **值和类型都可以随意变化**
{% endnote %}
