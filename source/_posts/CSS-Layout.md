---
title: CSS 布局
date: 2020-02-02 10:57:39
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, CSS]
---

如何选择布局。

<!-- more -->

# 两种布局方式

- **固定宽度布局**：960、1000、1024
- **不固定宽度布局**：一般在手机上使用，主要靠文档流原理来布局
- **响应式布局**：PC上固定宽度，手机上不固定宽度，也就是一种**混合布局**

# 两种布局思路

- 从大到小：先定下大局，然后再完善每个部分的小布局
- 从小到大：先完成小布局，然后再组合成大布局

![两种布局思路](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CSS-Layout.png)

# Float

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

# Flex

这里是 [CSS Tricks 上的 Flex 教程](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

## Container

- 让一个元素变成 flex 容器：`display: flex | inline-flex`
- 改变 items 的流动方向（主轴）：`flex-direction: row | row-reverse | column | column-reverse`
- 改变折行：`flex-wrap: nowrap | wrap | wrap-revers`
- 主轴的对齐方式：`justify-content: flex-start | flex-end | center | space-between | space-around | space-evenly`
- 次轴对齐：`align-items: flex-start | flex-end | center | stretch(默认)`
- 多行对齐（基本不用）：`align-content: flex-start | flex-end | center | strech | space-between | space-around`

## Items

- `order`：改变显示顺序
- `flex-grow`：控制自己如何长胖（占多余所有空间的权重），默认是0（尽可能窄）
- 导航栏常用左边 logo 和右边用户头像都是 0，中间是 1
- `flex-shrink`：控制如何变瘦，（在空间不够用的时候，压缩的比例）默认是1（大家一起变小），可以写0（防止变小）
- `flex-basis`：控制基准宽度，一般用得比较少，默认是 auto（跟宽度一样）

以上可以缩写为：`flex: grow shrink basis`

- `align-self: flex-start | flex-end`

经验：

- 永远不要把 `width` 和 `height` 写死
- 尽量使用 `min-width` | `max-width` 等来写
- `margin-xxx: auto`，有奇效，类似于 `between`

# Grid

这里是 [CSS Tricks 上的 Grid 教程](https://css-tricks.com/snippets/css/complete-guide-grid/)

![Grid布局](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/CSS-Grid.png)

## Container

- 让一个元素变成 grid 容器：`display: grid | inline-grid`
- 行和列：
    - `grid-template-columns`: 每一列的宽度，可以写 `auto`
    - `grid-template-rows`: 每一行的高度，可以写 `auto`
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

## Items

- grid-row-start
- grid-row-end
- grid-column-start
- grid-column-end
- fr：free space，类似于flex-grow
- **grid-area: header;**
- **grid-area: main;**
- **....**

# 水平垂直居中的方案

## 定位：三种

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

## `flex`

```scss
.father{
  display: flex;
  justify-content: center;
  align-items: center;
  .child{ }
}
```

## JavaScript

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

## `table-cell`

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

# 左右固定、中间自适应布局

## `float` + 负 `margin`

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

## `calc`

兼容到 IE 9，渲染会比较慢

```scss
.center{
  width: calc(100% - 400px);
}
```

## `flex`

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

## 定位

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
