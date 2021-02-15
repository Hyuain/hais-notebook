---
title: CSS 动画
date: 2020-02-02 13:10:32
tags:
  - 入门
categories:
  - [前端, CSS]
---

用 `transform` 的性能会比直接修改 `left` 属性好，因为浏览器不会 repaint 那么多次

<!-- more -->

# 浏览器的渲染过程

根据 [Google 团队的文章](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction)分析，我们可以将网页的渲染过程大致分为以下几步：

1. 根据 HTML 构建 HTML 树（DOM）
2. 根据 CSS 构建 CSS 树（CSSOM）
3. 将两棵树合并为渲染树（Render Tree）
4. 布局 **Layout**（文档流、盒模型、计算大小和位置）
5. 绘制 **Paint**（边框颜色、文字颜色、阴影）
6. 合成 **Compose**（根据层叠关系展示画面）

![浏览器渲染过程](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Browser-Render-Process.png)

# 更新样式

{% note warning %}
一般用 JS 更新样式，比如
```js
div.style.background = 'red'
div.style.display = 'none'
div.classList.add('red')
div.remove()
```
{% endnote %}

## 三种更新样式的路径

在 [这里](https://csstriggers.com/) 可以看到不同的属性在不同的浏览器中的更新路径

- JS/CSS > Style > Layout > Paint > Composite: remove
- JS/CSS > Style > Paint > Composite: background-color
- JS/CSS > Style > Composite: transform

## CSS 动画优化

可以看看 [谷歌的这篇文档](https://developers.google.com/web/fundamentals/performance/rendering)

- JS 优化：使用 `requestAnimationFrame` 代替 `setTimeout` 或者 `setInterval`
- CSS 优化：使用 `will-change` 或者 `transform`

# `transform`

> 使用 `transform: none`，取消所有

## 位移

```css
transform: translateX(50px) | translateY(50%) | translateZ(50px) | translate(50px, 50px)
```

视点的确定（在父元素上）： `perspective: 1000px` 表示父元素的中心为坐标原点，距离屏幕为 1000px

可以这样做绝对定位的居中：

```css
top: 50%; left: 50%; transform: translate(-50%, -50%)
```

## 缩放

border 也会一起变

```css
transform: scaleX(<number>) | scaleY(<number>) | scale(<number>, <number>?)
```

## 旋转

一般用来做 360 度旋转的 loading 或者按钮的交互

```css
transform: rotate([<angle>|<zero>]) | rotateZ([<angle>|<zero>]) | rotateX([<angle>|<zero>])
```

## 扭曲

```css
transform: skewX(<angle>|<zero>) | skewY(<angle>|<zero>) | skew(<angle>|<zero>, <angle>|<zero>?)
```

# `transition`

```css
transition: 属性名(可以写all) 时长 过渡方式( linear | ease | ease-in | ease-in-out | cubic-bezier | step-start | step-end | steps ) 延迟
transition: width 2s linear 3s
```

{% note warning %}
不是所有属性都有过渡
`display: none => block`，一般改成 `visibility: hidden => visible`
{% endnote %}

# `animation`

```css
/* 声明关键帧 */
@keyframes xxx {
  0% {
    transform: none;
  }
  66.66% {
  transform: translateX(200px)
  }
}

/* 添加动画 */
animation: 时长 过渡方式 延迟 次数( infinite ) 方向( reverse | alternate | alternate-reverse ) 填充模式( none | forwards | backwards | both) 是否暂停( paused )  动画名
```