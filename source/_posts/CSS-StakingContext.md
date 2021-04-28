---
title: CSS 层叠上下文
date: 2020-02-02 12:02:59
tags:
  - 入门
categories:
  - [前端, CSS]
---

我们假定用户正面向（浏览器）视窗或网页，而 HTML 元素沿着其相对于用户的一条虚构的 z 轴排开，**层叠上下文** 就是对这些 HTML 元素的一个 **三维构想**。众 HTML 元素基于其元素属性 **按照优先级顺序** 占据这个空间。

<!-- more -->

![CSS 层叠上下文](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Staking-Context.png)

默认的元素的三维层叠顺序如上图所示，可以看到定位元素会超出原来的高度范畴，换句话说，**原来的 background、border、块级子元素、浮动元素、内联子元素均处在 z-index = -1 ~ 0 这个区间之内**。

每一个层叠上下文就好像一个小世界，只有这里面的 `z-index` 才能进行比较，具有不同 `z-index` 的父元素之中的子元素根本无法同台竞技。

这些常见的属性（或元素）可以创建一个层叠上下文：

- `HTML`（根元素）
- `z-index` 值不为 `auto` 的 `relative | absolute` 元素
- `z-index` 值不为 `auto` 的 `flex` `grid` 子项
- `transform` 值不为 `none` 的元素
- `opacity` 值小于 `1` 的元素

更多资料可以查看 [MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Understanding_z_index/The_stacking_context) 。