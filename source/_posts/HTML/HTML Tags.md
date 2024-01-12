---
title: HTML Tags
date: 2020-02-02 13:33:25
categories:
  - - 前端
tags:
  - FX
---

HTML 常见标签及注意事项。

<!-- more -->

# 常用标签

- 表示书/文章的层级
    - 标题 `<h1>` ~ `<h6>`
    - 章节 `<section>`
    - 文章 `<article>`
    - 段落 `<p>`
    - 头部 `<header>`
    - 脚部 `<footer>`
    - 主要内容 `<main>`
    - 旁支内容 `<aside>`
    - 划分 `<div>`
- 内容标签
    - 有序列表 `<ol>` + `<li>`
    - 无序列表 `<ul>` + `<li>`
    - 描述 `<dl>` + `<dt>` + `<dd>`，其中 `<dt>` 表示描述词，`<dd>` 表示描述的内容（ `Emmet` 速写: `dl+`）
    - 保留空格的段落 `<pre>`
    - 分割线 `<hr>`
    - 换行 `<br>`
    - 定位符、超链接 `<a>`
    - 语气的强调 `<em>`
    - 本质的强调 `<strong>`
    - 代码 `<code>`，默认是内联元素，可以用 `<pre>` 包住 `<code>`
    - 引用 `<quote>`
    - 块级引用 `<blockquote>`
- 全局属性
    - `class` 类
    - `contenteditable` 用户可以直接编辑页面上的东西
    - `hidden` 隐藏
    - `id` 标记
    - `style` 样式
    - `tabindex` 控制 `Tab键` 激活元素的顺序，`tabindex=0` 是最后一个，`tabindex=-1` 代表永远不会访问
    - `title` 鼠标悬浮显示的内容

# `<a>` 标签

## `href` 属性

- 网址
    - `http://google.com`
    - `https://google.com`
    - `//google.com` 无协议网址
- 路径
    - `/a/b/c` 绝对路径，但是是基于 **HTTP 服务** 开启的根目录，不是整个计算机的根目录
    - `a/b/c` 相对路径，基于当前路径的目录
    - `index.html` 当前目录的文件
    - `./index.html` 当前目录的文件
- `#id`
- 伪协议
    - `javascript:alert(1);`
        - `javasrcript:;` 可以写一个什么都不做的a标签
    - `mailto:xxx@xxx.com` 会呼出邮件客户端
    - `tel:1300000000` 会呼出拨号界面

## `target` 属性

- 内置名字
    - `_blank` 新标签
    - `_top` 在最顶层打开（比如 `iframe` 的最外层页面）
    - `_parent` 在父级窗口打开，没有 `_top` 那么高层
    - `_self` 在当前页面打开（比如 `iframe` 的当前层）
- 其他自定义的新窗口的名字（`window.name`）或者 `iframe` 的名字

使用 `rel=noopener` 不打开新标签
    
## `download` 属性

下载而不是查看网页，但是大部分不支持

# `<iframe>` 标签

内嵌窗口，现在大都不用了

# `<table>` 标签

{% note warning %}
里面必须写`<thead>` `<tbody>` `<tfoot>`，否则浏览器也会自己加上，并且显示的顺序与实际写的这三个顺序无关，浏览器一定是按照`<thead>` `<tbody>` `<tfoot>`的顺序显示
{% endnote %}

相关的样式有：

- `table-layout`
    - `auto` 按照内容的多少来分配宽度权重
    - `fixed` 等宽
- `border-collapse: collapse` 表示两个单元格的边线合并
- `border-spacing` 两个单元格中间的空隙

# `<img>` 标签

> 发出一个GET请求，展示一张图片

## `src` 属性

可以是相对路径，也可以是绝对路径

## `alt` 属性

图片加载失败的时候显示的内容

## `height` 和 `width` 属性

若只写高度或宽度，图像比例保持不变

## 事件

- `onload`图片加载成功
- `onerror` 图片加载失败

## 响应式

`max-width: 100%`

# `<form>` 标签

> 发出一个 GET 或 POST 请求，然后刷新页面

## `action` 属性

请求到哪个页面

## `method` 属性

是用 GET 还是 POST

## `autocomplete` 属性

为 `on` 则打开自动填充，下面的 `text` 要写 `name`

## `target` 属性

把哪个页面（ 值可以为 `_blank` 等等）变成要请求到的那个页面（也就是说哪个页面需要刷新）

{% note warning %}
- 一般不监听 `input` 的 `click` 事件
- `form` 里面的 `input` 要有 `name`
- 一个 form 必须要有一个 `type="submit"`，如果 `button` 不写 `type`，默认 `submit`
{% endnote %}

{% note warning %}
**`<input type="submit">` 和 `<button type="submit">` 的区别？**
`input` 里面不能再有标签（但是文字可以用 `value=""` 进行更改）
`button` 里面可以有任何东西（包括图片等其他标签）
{% endnote %}

# `<input>` 标签

## `type` 属性

- `text`
- `color`
- `password`
- `radio` 所有的 `input` 要有同一个 `name`
- `checkbox` 所有的 `input` 要有同一个 `name`
- `file` 加上 `multiple` 属性可以同时选多个文件
- `hidden`
- `tel`
- `email`
- `search`

## 事件

- `onchange`
- `onfocus`
- `onblur`

# `<textarea>` 标签

`style="resize: none"` 让右下角不能拖动

# `<select>` 标签

`<option value="1">星期一</option>`
