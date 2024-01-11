---
title: 长列表优化：小程序虚拟列表
date: 2020-05-13 11:25:32
categories:
  - [源码]
---

借助小程序的 createIntersectionObserver 来实现虚拟列表。

<!-- more -->

# 用例

```wxml
<block wx:for="{{dataList}}" wx:key="id">
    <!-- 通过 overRenderScreenCount 控制当前实际渲染在页面中的项数 -->
    <virtual-list-item overRenderScreenCount="{{2}}">
        <view>{{item}}</view>
        <view slot="skeleton">
            ...skeleton template
        </view>
    </virtual-list-item>
</block>
```

# 思路

微信小程序提供了 `createIntersectionObserver` API，这个 API 可以监听元素的相交情况。利用此 API，我们可以知道元素何时进入（需要被渲染出来的）可视区域，何时退出可视区域，从而控制其是否渲染。
此外，还需要记录每个元素的高度，并在元素不被渲染出来的时候，提供一个 minHeight，防止高度塌陷。

# 伪代码

```wxml
<!-- 当元素没有实际渲染出来的时候，需要提供最小高度，防止高度塌陷 -->
<view class=".virtual-item" style="minHeight: {{isShow ? 0 : virtualHeight}}px">
    <block wx:if="{{isShow}}">
        <slot></slot>
    </block>
    <block wx:else>
        <slot name="skeleton"></slot>
    </block>
</view>
```

```typescript
let intersectionObserver

const ready = () => {
  const margin = props.overRenderScreenCount * windowHeight
  intersectionObserver = wx.createIntersectionObserver({ initialRatio: 0 })
    .relativeToViewPort({ top: margin, bottom: margin })
    .observe(".virtual-item", (res) => {
      data.isShow = res.intersectionRatio > 0
      data.virtualHeight = res.boundingClientRect.height
    })
}

const detached = () => {
  intersectionObserver?.disonnect?.()
}
```
