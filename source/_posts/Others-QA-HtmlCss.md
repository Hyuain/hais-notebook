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

# 移动端响应式方案

## @media

## PC 固定布局，移动端 rem

## flex

## vh/vw