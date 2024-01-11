---
title: Box Model
date: 2020-02-01 20:12:49
categories:
  - - 前端
---

CSS 的 **盒模型** 指的是，可以把网页中显示的 HTML 元素看成是一个个盒子，盒子具有四层：

- 内容（content）
- 内边距（padding）
- 边框（border）
- 外边距（margin）

| `box-sizing` | 特性 |
| --- | --- |
| `content-box` | 指定 `width` 或 `height` 时，说的是他的最里面的 `content` 的 宽或高 |
| `border-box` | 指定 `width` 或 `heihgt` 时，除了 `content` 的宽或高之外，还包括了 `padding` 和 `border` 的厚度 |

通过 `box-sizing` 控制，一般使用 `border-box`

