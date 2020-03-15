---
title: JavaScript：Canvas
date: 2020-03-05 17:26:02
tags:
  - 入门
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

介绍 Canvas。

<!-- more -->

# 什么是 Canvas

> `<canvas>` 是 HTML 5 新增的元素，可用于通过使用 JavaScript 的脚本来绘制图形。例如，他可以用于绘制图形、制作照片、创建动画，甚至可以进行实时视频处理和渲染

简单来说，`<canvas>` 提供了一块画布，在上面可以使用 JavaScript 动态渲染图像

## Canvas 与 SVG

> SVG（Scalable Vector Graphics）是基于 XML，用于描述二维矢量图形的一种图片格式，由 W3C 指定，是一个开放标准

Canvas 在绘制结束之后不能被保存到内存中，每次都需要重新绘制，是实时绘制的；svg 则在绘制完之后会被保存在内存中，当需要修改这个对象信息的时候，直接修改就可以了

| Canvas | SVG |
| --- | --- |
| 依赖分辨率（位图） | 不依赖分辨率（矢量图） |
| 单个 HTML 元素 | 每个图形都是一个 DOM 元素 |
| 只能通过脚本语言绘制 | CSS、脚本语言都可以用来绘制 |
| 不支持事件处理程序 | 支持事件处理程序 |
| 弱的文本渲染恩能够力 | 最适合带有大型渲染区域的应用程序，比如谷歌地图 |
| 图面较小，对象数量较大时（>10k）性能最佳 | 对象数量较小（<10k）、图面更大时性能更佳 |

# Canvas 的应用场景

1. **绘制图表**，比如 EChats 之类的数据可视化库都是使用 Canvas 实现的
2. **小游戏**，基本上所有的 HTML 5 游戏都是基于 Canvas 开发的，因为不需要借助其他任何插件
3. **活动页面**，比如转转轮、刮奖
4. **特效、背景**

# Canvas API

## 设置宽高

可以通过 HTML 或 JS 来设置，不能通过 CSS 设置，这会导致不可预料的缩放

## 获取 Canvas 对象

```js
canvas.getContext(contextType, contextAttributes)
```

`contextType` 是上下文类型，有这样几种：
- `2d`：代表一个二维渲染上下文
- `webgl` 或 `experimental-webgl`：代表一个三维渲染上下文
- `webgl2` 或 `experimental-webgl2`：代表一个三维渲染上下文，并且只能在浏览器中实现 WebGL 版本 2（OpenGL ES 3.0）

## 绘制路径

| 方法 | 描述 |
| --- | --- |
| `fill()` | 填充路径 |
| `stroke()` | 描边 |
| `arc()` | 创建圆弧 |
| `rect()` | 创建矩形 |
| `fillRect()` | 绘制矩形路径区域 |
| `strokeRect()` | 绘制矩形路径描边 |
| `clearRect()` | 在给定的矩形内清除指定的像素 |
| `arcTo()` | 创建两切线之间的弧/曲线 |
| `beginPath()` | 起始一条路径，或重置当前路径 |
| `moveTo()` | 把路径移动到画布中的指定点，不创建线条 |
| `lineTo()` | 添加一个新点，然后画布中创建从该点到最后指定点的线条 |
| `closePath()` | 创建从当前点回到起始点的路径 |
| `clip()` | 从原始画布剪切任意形状和尺寸的区域 |
| `quadraticCurveTo()` | 创建二次方贝塞尔曲线 |
| `bezierCurveTo()` | 创建三次方贝塞尔曲线 |
| `isPointInPath()` | 如果指定的点位于当前路径中，则返回 true，否则返回 false |

### 绘制曲线

#### arc()

```js
context.arc(x, y, sAngle, eAangle, counterCloseWise)
```

`x` 和 `y` 指定了圆心坐标，`sAngle` 和 `eAngle` 分别为起始角和结束角，`counterCloseWise` 设置为 `true` 表示逆时针绘制

### 绘制直线

```js
moveTo(x, y) // 把路径移动到画布中的指定点，不创建线条
lineTo(x, y) // 添加一个新点，然后在画布中创建从该点到最后指定点的线条
```

{% note warning %}
如果没有 `moveTo`，那么第一次 `lineTo` 就视为 `moveTo`
{% endnote %}

可以给绘制的直线添加样式：

| 样式 | 描述 |
| --- | --- |
| `lineCap` | 设置或返回线条的结束端点样式 |
| `lineJoin` | 设置或返回两条线相交时，所创建的拐角类型 |
| `lineWidth` | 设置或返回当前的线条宽度 |
| `miterLimit` | 设置或返回最大斜接长度 |

### 绘制矩形

```js
fillRect(x, y, width, height) // 绘制一个实心矩形
strokeRect(x, y, width, height) // 绘制一个空心矩形
```

### 颜色、样式和阴影

创建路径之后，可以设置样式属性，再使用 `fill()` 和 `stroke()` 进行填充或描边

| 属性 | 描述 |
| --- | --- |
| fillStyle | 设置或返回用于填充绘画的颜色、渐变或模式 |
| strokeStyle | 设置或返回用于笔触的颜色、渐变或模式 |
| shadowColor | 设置或返回用于阴影的颜色 |
| shadowBlur | 设置或返回用于阴影的模糊级别 |
| shadowOffsetX | 设置或返回阴影距形状的水平距离 |
| shadowOffsetY | 设置或返回阴影距形状的垂直距离 |

### 设置渐变

有这样几个方法可以设置渐变：

| 方法 | 描述 |
| --- | --- |
| createLinearGradient() | 创建线性渐变（用在画布内容上） |
| createPattern() | 在指定的方向上重复指定的元素 |
| createRadialGradient() | 环形的渐变（用在画布内容上） |
| addColorStop()| 规定渐变对象中的颜色和停止位置 |

主要使用到下面这个方法

```js
const grd = context.createLinerGradient(x0, y0, x1, y1)
grd.addColorStop(stop, color) // stop 是介于 0.0 ~ 1.0 之间的值
```

### 图形转换

| 方法 | 描述 |
| --- | --- |
| scale() | 缩放当前绘图至更大或更小 |
| rotate() | 旋转当前绘图 |
| translate() | 重新映射画布上的 (0,0) 位置 |
| transform() | 替换绘图的当前转换矩阵 |
| setTransform() | 将当前转换重置为单位矩阵，然后运行 transform() |

注意旋转实际上是对画布旋转，旋转之后所有的都会旋转

### 图形绘制

```js
drawImage(img, sx, sy, swidth, sheight, x, y, width, height)
```

- `img`：规定要使用的图像、画布或视频
- `sx`：可选。开始剪切的 x 坐标位置
- `sy`：可选。开始剪切的 y 坐标位置
- `swidth`：可选。被剪切图像的宽度
- `sheight`：可选。被剪切图像的高度
- `x`：在画布上放置图像的 x 坐标位置
- `y`：在画布上放置图像的 y 坐标位置
- `width`：可选。要使用的图像的宽度（伸展或缩小图像）
- `height`：可选。要使用的图像的高度（伸展或缩小图像）