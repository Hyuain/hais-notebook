---
title: Margin Collapse
date: 2020-02-01 20:12:49
categories:
  - - 前端
tags:
  - FX
---

**Margin 合并** 是说我们上下两个元素若都有 `margin`，那么他们的 `margin` 将不会同时生效。有以下几个注意点：

1. 只有上下才会发生 `margin` 合并，左右不会发生
2. 只有 `block` 会发生 `margin` 合并，`inline-block` 不会发生（生成了 BFC）
3. `first-child` 和 `last-child` 也会和 `parent` 发生 `margin` 合并 ^632f55

取消 margin 合并的方法，主要是靠生成 [[BFC]]：

1. 使用 `overflow:hidden` 之后就不会再合并
2. 使用 `display:flex` 之后就不会再合并
3. 通过给 `parent` 上面加东西（`padding` `border` `overflow` 等）之后就不会再合并

