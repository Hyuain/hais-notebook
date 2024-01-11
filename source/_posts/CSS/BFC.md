---
title: BFC
date: 2020-02-01 20:12:49
categories:
  - - 前端
---

**块级格式化上下文（Block Formatting Context, BFC）** 就是一块小区域，一块独立的布局环境，在这个区域之内的任何布局和定位都不会影响到外面，外面也不能影响到里面。

# BFC 的作用

1. 同一个 BFC 内会发生垂直方向的 [[Margin Collapse]]，如果是不同 BFC 则不会
2. 形成了 BFC 的区域不会与 float 元素重叠，可以与浮动元素形成左右自适应布局
3. 清除浮动，因为浮动元素的高度也会被 BFC 纳入计算，因此可以解决高度塌陷的问题

# 如何形成 BFC

1. 根元素
2. `float: left`
3. `position: absolute / fixed`
4. `overflow !== visible`，经常使用
5. `display: inline-block / table`

