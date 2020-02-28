---
title: CSS 基本概念
date: 2020-02-01 20:23:07
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, CSS]
---

主要是包括文档流、脱离文档流、盒模型、margin 合并等问题

<!-- more -->

# 文档流

> 这里的文档流指的是 **Normal Flow**，也就是 **正常布局流**
> 当我们没有使用过任何 CSS 规则来改变元素的展现方式的时候，他们会按照正常的布局流来组织——也就是默认情况下的元素布局——这也就是我们所说的 **Noraml Flow**，即 **文档流** 

| `display` | 流动方向 | 宽度 | 高度 |
| --- | --- | --- | --- |
| `inline` | 从左到右排列，**行尾会截断成两行** | 里面所有 inline 元素之和，不接受指定 `width` | **实际高度**由**行高（`line-height`）间接**（与字体有关，具体可以看 [这个](https://zhuanlan.zhihu.com/p/25808995?group_id=825729887779307520)）**确定**，`padding` 只能改变**看得见**的高度（而不是实际高度），如果没有内容，也有高度，为 `line-height` |
| `block` | 从上到下排列（每个占一行） | 默认为 `auto`（不是100%），可以指定 `width`，但永远不要写 `width: 100%` | 由里面**所有**的**文档流**元素决定，也可以设置 `height`，如果没有内容，高度为 `0`
| `inline-block` | 从左到右排列，**行尾不会分成两块** | 默认为里面所有 `inline` 元素之和，但接受指定 width | 由里面**所有**的**文档流**元素决定，也可以设置 `height` |

# 溢出

> 这里的溢出指的是 **Overflow**

| `overflow` | 特性 |
| --- | --- |
| `hidden` | 隐藏 |
| `scroll` | 滚动，就算没有溢出，依然有滚动条，**如果有横向滚动条，inline 元素也只会在第一屏** |
| `auto` | 没有溢出就没有滚动条，并且只显示必要的滚动条 |

可以分开设置 `overflow-x` 和 `overflow-y`

# 脱离文档流

> 也就是 **Out of flow**

{% note warning %}
block 不计算其高度
{% endnote %}

```css
position: absolute | fixed;
/* OR */
float: left | right;
```

# 盒模型

可以把网页中显示的 HTML 元素看成是一个个盒子，盒子具有四层：

- 内容（content）
- 内边距（padding）
- 边框（border）
- 外边距（margin）

| `box-sizing` | 特性 |
| --- | --- |
| `content-box` | 指定 `width` 或 `height` 时，说的是他的最里面的 `content` 的 宽或高 |
| `border-box` | 指定 `width` 或 `heihgt` 时，除了 `content` 的宽或高之外，还包括了 `padding` 和 `border` 的厚度 |

通过 `box-sizing` 控制，一般使用 `border-box`

# Margin 合并

跟 BFC 有关系，可以查看 [CSS BFC](/hais-notebook/2020/02/12/CSS-BFC/)

是说我们上下两个元素若都有 margin，那么他们的 margin 将不会同时生效。有以下几个注意点：

1. 只有上下才会发生 `margin` 合并，左右不会发生
2. 只有 `block` 会发生 `margin` 合并，`inline-block` 不会发生
3. `first-child` 和 `last-child` 也会和 `parent` 发生 margin 合并

取消 margin 合并的方法：

1. 使用 `overflow:hidden` 之后就不会再合并
2. 使用 `display:flex` 之后就不会再合并
3. 通过给 `parent` 上面加东西（`padding` `border` `overflow` 等）之后就不会再合并