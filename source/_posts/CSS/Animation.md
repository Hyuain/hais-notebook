---
title: Animation
date: 2020-02-01 20:12:49
categories:
  - - 前端
tags:
  - FX
---


用 `transform` 的性能会比直接修改 `left` 属性好，因为浏览器不会 repaint 那么多次

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