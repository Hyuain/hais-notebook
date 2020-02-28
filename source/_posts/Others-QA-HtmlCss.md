---
title: FAQs：HTML 与 CSS
date: 2020-02-04 09:53:08
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [前端, 其他]
---

关于 HTML 和 CSS 的一些问题与答案。

<!-- more -->

# HTML

## HTML 语义化？

### HTML 语义化的历史

最开始是后台用写 HTML，主要使用 table 来布局，维护起来非常麻烦；后来进入 DIV+CSS 的时代，问题是不够语义化；现在才是比较专业的，使用正确的标签的正确的写法。

具体：平时写的代码中用到的语义化标签。

## meta viewport？

## 用过哪些 HTML 5 标签？

## H5 是什么？

# CSS

## 回溯机制？

## flex 的常见属性？

## 页面导入样式时，`link` 和 `@import` 有什么区别？

| link | @import |
| --- | --- |
| HTML 标签，除了 CSS 之外还可以定义 RSS 等 | CSS 提供的 |
| 加载页面时同时加载 | 页面加载完之后再加载 |
| 没有兼容性问题 | 不兼容 IE 5 以下 |
| 可以通过 DOM 动态引入 | 不能动态引入 |

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

3. `sbsolute` + `translate`： 不需要知道自己的宽高，不需要有具体的宽高

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

## 移动端响应式方案

1. @media
2. PC 固定布局，移动端 rem
3. flex
4. vh/vw

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

## CSS 选择器优先级？

1. 越具体优先级越高
2. 写在后面的覆盖写在前面的
3. `important!` 最高，但是少用

## 清除浮动