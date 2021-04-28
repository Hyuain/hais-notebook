---
title: CSS 定位
date: 2020-02-02 11:57:07
tags:
  - 入门
categories:
  - [前端, CSS]
---

定位和布局的区别：布局是平面上的，定位是垂直于屏幕的。

<!-- more -->

# 一个 div 的分层

![一个div的分层](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CSS-Position.png)

# `position`

## `static`

默认值，当前元素在文档流中

## `relative`

- 距离自己原来的位置的偏离（`top` / `bottom`），还是占据原来的空间
- 做位移
- 做 `absolute` 元素的爸爸
- 配合 `z-index`，默认是 `auto`，`auto` 计算出来的值为 `0`
- **经验**：不要写 `z-index: 9999`，要学会 `z-index` 的管理

## `absolute`

- 脱离原来的位置，另起一层；会相对于**祖先元素中最近的**一个**定位元素**（即非 `static` 元素）
- 可以用于悬浮显示提示内容
- `white-space: nowrap` 文字内容不换行
- 经验：
    - 某些浏览器如果不写 `top / left` 会引起混乱
    - 善用 `left: 100%`，善用 `left: 50% `加负 `margin`

## `fixed`

- 相对于视口定位，但是如果放到具有 `transform`  属性的元素里面，会有问题
- 做广告
- 做回到顶部按钮
- **经验**
    - 某些浏览器如果不写 `top / left` 会引起混乱
    - 手机上尽量不要使用 `fixed`
    
## `sticky`

兼容性很差