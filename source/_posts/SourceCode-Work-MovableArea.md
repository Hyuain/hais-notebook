---
title: 小程序拖拽排序
date: 2021-08-18 09:25:32
tags:
  - 源码
  - 抄袭
  - 优化
  - 长列表
categories:
  - [源码, 工作]
---

借助小程序的 movableArea 和 movableView 实现拖拽排序。

<!-- more -->

# 官方文档

[movable-area](https://developers.weixin.qq.com/miniprogram/dev/component/movable-area.html)

[movable-view](https://developers.weixin.qq.com/miniprogram/dev/component/movable-view.html)

# 整体思路

借助 Move 类实现，每个 movable-area 对应一个 Move 类的实例。

- 长按
  - 用一个对象 `moveItemInfo` 记录正在移动的元素的信息：记录准备移动的元素开始时的 `index`
- 移动
  - 更新 `moveItemInfo`：更新实时的 `x` `y`
  - *计算出移动中元素实时的 `index`：初始化的时候传入可移动元素的宽、高、间距，以及列的数量，然后结合当前 `moveItemInfo` 中的 `x` `y` 信息，计算拖动到的 `index`
  - *将元素移动导新的 `index`：遍历当前的所有可移动元素，找到他们的 `index`，根据当前移动中的元素是向前还是向后移动，来确定其他的元素是否需要移动、需要向哪个方向移动
  - 更新其他元素的 `movable-view` 中的 `x` 和 `y` 属性，达到移动的效果
- 松手：计算出可移动对象最终的 index，并更新整个列表的位置信息
