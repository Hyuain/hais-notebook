---
title: CSS Positioning
date: 2020-02-01 20:12:49
categories:
  - - 前端
---
名为 CSS 的魔法可以给网页添加色彩与各种丰富的样式。

<!-- more -->

# 找资料

- 查资料：[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS)、[CSS Tricks](https://css-tricks.com/)、[张鑫旭的博客](https://www.zhangxinxu.com/wordpress/)
- 找素材（可以搜 PSD WEB）：[Freepik](https://www.freepik.com/)、[Dribbble](http://dribbble.com/)、[365 PSD](https://cn.365psd.com/)

# 层叠的含义

CSS 全称是 **层叠样式表（Cascading Style Sheets）**，可以从以下几个方面解释 **层叠**：

- **样式层叠**：可以对同一个选择器进行样式声明
- **选择器层叠**：可以用不同的选择器对同一个元素进行样式声明
- **文件层叠**：可以用多个文件进行层叠

# 样式语法

```css
选择器 {
  属性名: 属性值;
  /* 注释 */
}
```

# @ 语法

```css
@charset "UTF-8";   /* 必须放在第一行 */
@import url(2.css); 
@media (min-width: 100px) and (max-width: 200px) { }
```

其中 `@charset` 可以指定字符编码，具体可查看 [[Charset]]。

# Concepts

CSS 中包含一些概念，比如：

- 文档流 [[Document Flow]]
- 盒模型 [[Box Model]]
- CSS 布局 [[CSS Layout]]
- 
# Positioning

定位和布局的区别：布局是平面上的，定位是垂直于屏幕的。

## 一个 div 的分层

![一个div的分层](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CSS-Position.png)

## `position`

### `static`

默认值，当前元素在文档流中

### `relative`

- 距离自己原来的位置的偏离（`top` / `bottom`），还是占据原来的空间
- 做位移
- 做 `absolute` 元素的爸爸
- 配合 `z-index`，默认是 `auto`，`auto` 计算出来的值为 `0`
- **经验**：不要写 `z-index: 9999`，要学会 `z-index` 的管理

### `absolute`

- 脱离原来的位置，另起一层；会相对于**祖先元素中最近的**一个**定位元素**（即非 `static` 元素）
- 可以用于悬浮显示提示内容
- `white-space: nowrap` 文字内容不换行
- 经验：
  - 某些浏览器如果不写 `top / left` 会引起混乱
  - 善用 `left: 100%`，善用 `left: 50% `加负 `margin`

### `fixed`

- 相对于视口定位，但是如果放到具有 `transform`  属性的元素里面，会有问题
- 做广告
- 做回到顶部按钮
- **经验**
  - 某些浏览器如果不写 `top / left` 会引起混乱
  - 手机上尽量不要使用 `fixed`

### `sticky`

兼容性很差

# Staking Context

我们假定用户正面向（浏览器）视窗或网页，而 HTML 元素沿着其相对于用户的一条虚构的 z 轴排开，**层叠上下文** 就是对这些 HTML 元素的一个 **三维构想**。众 HTML 元素基于其元素属性 **按照优先级顺序** 占据这个空间。

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

# Animation

用 `transform` 的性能会比直接修改 `left` 属性好，因为浏览器不会 repaint 那么多次

## 浏览器的渲染过程

根据 [Google 团队的文章](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction)分析，我们可以将网页的渲染过程大致分为以下几步：

1. 根据 HTML 构建 HTML 树（DOM）
2. 根据 CSS 构建 CSS 树（CSSOM）
3. 将两棵树合并为渲染树（Render Tree）
4. 布局 **Layout**（文档流、盒模型、计算大小和位置）
5. 绘制 **Paint**（边框颜色、文字颜色、阴影）
6. 合成 **Compose**（根据层叠关系展示画面）

![浏览器渲染过程](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Browser-Render-Process.png)

## 更新样式

{% note warning %}
一般用 JS 更新样式，比如
```js
div.style.background = 'red'
div.style.display = 'none'
div.classList.add('red')
div.remove()
```
{% endnote %}

### 三种更新样式的路径

在 [这里](https://csstriggers.com/) 可以看到不同的属性在不同的浏览器中的更新路径

- JS/CSS > Style > Layout > Paint > Composite: remove
- JS/CSS > Style > Paint > Composite: background-color
- JS/CSS > Style > Composite: transform

### CSS 动画优化

可以看看 [谷歌的这篇文档](https://developers.google.com/web/fundamentals/performance/rendering)

- JS 优化：使用 `requestAnimationFrame` 代替 `setTimeout` 或者 `setInterval`
- CSS 优化：使用 `will-change` 或者 `transform`

## `transform`

> 使用 `transform: none`，取消所有

### 位移

```css
transform: translateX(50px) | translateY(50%) | translateZ(50px) | translate(50px, 50px)
```

视点的确定（在父元素上）： `perspective: 1000px` 表示父元素的中心为坐标原点，距离屏幕为 1000px

可以这样做绝对定位的居中：

```css
top: 50%; left: 50%; transform: translate(-50%, -50%)
```

### 缩放

border 也会一起变

```css
transform: scaleX(<number>) | scaleY(<number>) | scale(<number>, <number>?)
```

### 旋转

一般用来做 360 度旋转的 loading 或者按钮的交互

```css
transform: rotate([<angle>|<zero>]) | rotateZ([<angle>|<zero>]) | rotateX([<angle>|<zero>])
```

### 扭曲

```css
transform: skewX(<angle>|<zero>) | skewY(<angle>|<zero>) | skew(<angle>|<zero>, <angle>|<zero>?)
```

## `transition`

```css
transition: 属性名(可以写all) 时长 过渡方式( linear | ease | ease-in | ease-in-out | cubic-bezier | step-start | step-end | steps ) 延迟
transition: width 2s linear 3s
```

{% note warning %}
不是所有属性都有过渡
`display: none => block`，一般改成 `visibility: hidden => visible`
{% endnote %}

## `animation`

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
animation: 时长 过渡方式 延迟 次数( infinite ) 方向( reverse | alternate | alternate-reverse ) 填充模式( none | forwards | backwards | both) 是否暂停( paused )  动画名
```

# CSS Reset

[[Default Styles & CSS Reset]]