---
title: CSS
date: 2020-02-01 20:12:49
tags:
  - 入门
categories:
  - [前端, CSS]
---

整合所有 CSS 相关笔记。

<!-- more -->

# Introduction

## 层叠的含义

- **样式层叠**：可以对同一个选择器进行样式声明
- **选择器层叠**：可以用不同的选择器对同一个元素进行样式声明
- **文件层叠**：可以用多个文件进行层叠

## 样式语法

```css
选择器 {
  属性名: 属性值;
  /* 注释 */
}
```

## @ 语法

```css
@charset "UTF-8";   /* 必须放在第一行 */
@import url(2.css); 
@media (min-width: 100px) and (max-width: 200px) { }
```

## charset

- charset 是字符集的意思，但 UTF-8 不是字符集，而是字符编码(encoding)，这是历史遗留问题
- ASCII 字符集的编码形式就是 ASCII，这个字符集只有英文
- GB2312 字符集是中国人发明的简体中文字符集，他的编码形式也是 GB2312
- GBK 是微软发明的中日韩（CJK）字符集，他的编码形式也是 GBK
- unicode 是支持所有字符的字符集，他的编码形式有 UTF-8/UTF-16/UTF-32

## 找资料

- 查资料：[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS)、[CSS Tricks](https://css-tricks.com/)、[张鑫旭的博客](https://www.zhangxinxu.com/wordpress/)
- 找素材（可以搜 PSD WEB）：[Freepik](https://www.freepik.com/)、[Dribbble](http://dribbble.com/)、[365 PSD](https://cn.365psd.com/)

# Concepts

## 文档流

> 这里的文档流指的是 **Normal Flow**，也就是 **正常布局流**
> 当我们没有使用过任何 CSS 规则来改变元素的展现方式的时候，他们会按照正常的布局流来组织——也就是默认情况下的元素布局——这也就是我们所说的 **Noraml Flow**，即 **文档流**

| `display` | 流动方向 | 宽度 | 高度 |
| --- | --- | --- | --- |
| `inline` | 从左到右排列，**行尾会截断成两行** | 里面所有 inline 元素之和，不接受指定 `width` | **实际高度**由**行高（`line-height`）间接**（与字体有关，具体可以看 [这个](https://zhuanlan.zhihu.com/p/25808995?group_id=825729887779307520)）**确定**，`padding` 只能改变**看得见**的高度（而不是实际高度），如果没有内容，也有高度，为 `line-height` |
| `block` | 从上到下排列（每个占一行） | 默认为 `auto`（不是100%），可以指定 `width`，但永远不要写 `width: 100%` | 由里面**所有**的**文档流**元素决定，也可以设置 `height`，如果没有内容，高度为 `0`
| `inline-block` | 从左到右排列，**行尾不会分成两块** | 默认为里面所有 `inline` 元素之和，但接受指定 `width` | 由里面**所有**的**文档流**元素决定，也可以设置 `height` |

## 溢出

> 这里的溢出指的是 **Overflow**

| `overflow` | 特性 |
| --- | --- |
| `hidden` | 隐藏 |
| `scroll` | 滚动，就算没有溢出，依然有滚动条，**如果有横向滚动条，inline 元素也只会在第一屏** |
| `auto` | 没有溢出就没有滚动条，并且只显示必要的滚动条 |

可以分开设置 `overflow-x` 和 `overflow-y`

## 脱离文档流

> 也就是 **Out of flow**

{% note warning %}
block 不计算其高度
{% endnote %}

```css
position: absolute | fixed;
/* OR */
float: left | right;
```

## 盒模型

可以把网页中显示的 HTML 元素看成是一个个盒子，盒子具有四层：

- 内容（content）
- 内边距（padding）
- 边框（border）
- 外边距（margin）

| `box-sizing` | 特性 |
| --- | --- |
| `content-box` | 指定 `width` 或 `height` 时，说的是他的最里面的 `content` 的 宽或高 |
| `border-box` | 指定 `width` 或 `heihgt` 时，除了 `content` 的宽或高之外，还包括了 `padding` 和 `border` 的厚度 |

通过 `box-sizing` 控制，一般使用 `border-box`

## Margin 合并

> 跟 BFC 有关系

是说我们上下两个元素若都有 `margin`，那么他们的 `margin` 将不会同时生效。有以下几个注意点：

1. 只有上下才会发生 `margin` 合并，左右不会发生
2. 只有 `block` 会发生 `margin` 合并，`inline-block` 不会发生（生成了 BFC）
3. `first-child` 和 `last-child` 也会和 `parent` 发生 `margin` 合并

取消 margin 合并的方法，主要是靠生成 BFC：

1. 使用 `overflow:hidden` 之后就不会再合并
2. 使用 `display:flex` 之后就不会再合并
3. 通过给 `parent` 上面加东西（`padding` `border` `overflow` 等）之后就不会再合并

## BFC

> BFC （块级格式化上下文）就是一块小区域，一块独立的布局环境，在这个区域之内的任何布局和定位都不会影响到外面，外面也不能影响到里面。

### BFC 的作用

1. 同一个 BFC 内会发生垂直方向的 Margin 合并，如果是不同 BFC 则不会
2. 形成了 BFC 的区域不会与 float 元素重叠，可以与浮动元素形成左右自适应布局
3. 清除浮动，因为浮动元素的高度也会被 BFC 纳入计算，因此可以解决高度塌陷的问题

### 如何形成 BFC

1. `float: left`
2. `position: absolute`
3. `overflow: hidden`，经常使用
4. `display: inline-block`
5. `display: table-cell`

# Layout

## 两种布局方式

- **固定宽度布局**：960、1000、1024
- **不固定宽度布局**：一般在手机上使用，主要靠文档流原理来布局
- **响应式布局**：PC上固定宽度，手机上不固定宽度，也就是一种**混合布局**

## 两种布局思路

- 从大到小：先定下大局，然后再完善每个部分的小布局
- 从小到大：先完成小布局，然后再组合成大布局

![两种布局思路](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CSS-Layout.png)

## Float

1. 在子元素上加 `float: left` 和 `width`
2. 在父元素加上 `class="clearfix"`

```css
.clearfix:after{
  content: '';
  display: block;
  clear: both;
}
```

经验：

- 有经验者会留一些空间或最后一个不设置 `width`（可以设置一个 `max-width`）
- 不需要考虑响应式，因为手机上没有 IE，这个布局是专门为 IE 准备的
- 在 IE 上有 BUG：最左边浮动元素的 `margin-left` 会变成双倍
  - 可以加一句兼容性写法：`_margin-left: ?px`
  - 也可以加上 `display: inline-block`
- 如果图片下面有空隙，加上 `vertical-align: middle | top` 就可以消除
- **`float` 元素外边距不合并**

{% note warning %}
必要时可以采用负 margin
{% endnote %}

比如说做平均布局，在所有的小块外面加一个 `div` ，然后再给这个 `div` 一个负 `margin`

![负Margin](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CSS-Layout-Minus-Margin.png)

## Flex

这里是 [CSS Tricks 上的 Flex 教程](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

### Container

- 让一个元素变成 flex 容器：`display: flex | inline-flex`
- 改变 items 的流动方向（主轴）：`flex-direction: row | row-reverse | column | column-reverse`
- 改变折行：`flex-wrap: nowrap | wrap | wrap-revers`
- 主轴的对齐方式：`justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly`
- 次轴对齐：`align-items: flex-start | flex-end | center | stretch(默认)`
- 多行对齐（基本不用）：`align-content: flex-start | flex-end | center | strech | space-between | space-around`

### Items

- `order`：改变显示顺序
- `flex-grow`：控制自己如何长胖（占多余所有空间的权重），默认是0（尽可能窄）
- 导航栏常用左边 logo 和右边用户头像都是 0，中间是 1
- `flex-shrink`：控制如何变瘦，（在空间不够用的时候，压缩的比例）默认是1（大家一起变小），可以写0（防止变小）
- `flex-basis`：控制基准宽度，一般用得比较少，默认是 auto（跟宽度一样）

以上可以缩写为：`flex: grow shrink basis`

- `align-self: flex-start | flex-end`

经验：

- 永远不要把 `width` 和 `height` 写死
- 尽量使用 `min-width` | `max-width` 等来写
- `margin-xxx: auto`，有奇效，类似于 `between`

## Grid

这里是 [CSS Tricks 上的 Grid 教程](https://css-tricks.com/snippets/css/complete-guide-grid/)

![Grid布局](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CSS-Grid.png)

### Container

- 让一个元素变成 grid 容器：`display: grid | inline-grid`
- 行和列：
  - `grid-template-columns`: 每一列的宽度，可以写 `auto`
  - `grid-template-rows`: 每一行的高度，可以写 `auto`
  -
```css
grid-template-areas:
  "header header header"
  "aside main ad"
  "footer fotter ."
```

    - `grid-gap`
    - `grid-column-gap`
    - `grid-row-gap`

### Items

- grid-row-start
- grid-row-end
- grid-column-start
- grid-column-end
- fr：free space，类似于flex-grow
- **grid-area: header;**
- **grid-area: main;**
- **....**

## 水平垂直居中的方案

### 定位：三种

1. `absolute` + `margin`：需要知道自己的宽高

```scss
.father{
  position: relative;
  .child{
    position: absolute;
    height: 50px;
    width: 100px;
    left: 50%;
    top: 50%;
    margin-top: -25px;
    margin-left: -50px;
  }
}
```

2. `absolute` + `margin`：不需要知道自己的宽高，但是得有宽高

```scss
.father{
  position: relative;
  .child{
    position: absolute;
    height: 50px;
    width: 100px;
    left: 0;
    right: 0;
    top: 0;
    bottom: 0;
    margin: auto;
  }
}
```

3. `absolute` + `translate`： 不需要知道自己的宽高，不需要有具体的宽高

```scss
.father{
  position: relative;
  .child{
    position: absolute;
    height: 50px;
    width: 100px;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
  }
}
```

### `flex`

```scss
.father{
  display: flex;
  justify-content: center;
  align-items: center;
  .child{ }
}
```

### JavaScript

```scss
body{
  position: relative;
  #box{ }
}
```
```js
const HTML = document.documentElement,
      winW = HTML.clientWidth,
      winH = HTML.clientHeight,
      boxW = box.offsetWidth, // 带边框的宽，也可以用 getBoundingClientReact
      boxH = box.offsetHeight // 带边框的高
box.style.position = 'absolute'
box.style.left = (winW - boxW) / 2 + 'px'
box.style.top = (winH - boxH) / 2 + 'px'
```

### `table-cell`

要求父级有固定宽高，相当于给文本居中

```scss
.father{
  display: table-cell;
  width: 500px;
  height: 500px;
  vertical-align: middle;
  text-align: center;
  .child{
    display: inline-block;
  }
}
```

## 左右固定、中间自适应布局

### `float` + 负 `margin`

圣杯布局

```scss
.container{
  padding: 0 200px;
  .left, .right{
    width: 200px;
  }
  .center{
    width: 100%;
  }
  .center, .left, .right{
    float: left;
  }
  .left{
    margin-left: -100%;
    position: relative;
    left: -200px;
  }
  .right{
    margin-right: -200px;
  }
}
```

双飞翼布局

```scss
.container{
  width: 100%;
  padding: 0 200px;
  .center{
    width: 100%;
  }
}
.left, .right{
  width: 200px;
}
.left, .right, .center{
  float: left;
}
.left{
  margin-left: -100%;
}
.right{
  margin-right: -200px;
}
```

### `calc`

兼容到 IE 9，渲染会比较慢

```scss
.center{
  width: calc(100% - 400px);
}
```

### `flex`

```scss
.container{
  display: flex;
  justify-content: space-between;
  .left, .right{
    flex: 0 0 200px;
  }
  .center{
    flex: 1;
  }
}
```

### 定位

```scss
.container{
  position: relative;
  .left, .right{
    position: absolute;
    width: 200px;
    top: 0;
  }
  .left{
    left: 0;
  }
  .right{
    right: 0;
  }
  .center{
    margin: 0 200px;
  }
}
```

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

