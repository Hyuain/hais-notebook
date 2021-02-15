---
title: HTML 序章
date: 2020-02-02 13:29:55
tags:
  - 入门
categories:
  - [前端, HTML]
---

WWW 和 HTML 的历史。

<!-- more -->

# Word Wide Web

- 互联网，基于IP之间通信，但是无法输入地址得到一个网页
- WWW = URL + HTTP + HTML（李爵士发明的，基于互联网实现的，输入地址得到一个网页的网络）

# HTML 历史

诞生于李爵士的一篇文章，最开始只有 18 个元素，这些元素如今还有 11 个健在，现在最新版的HTML大概有 110 个标签

## HTML 5

- 狭义HTML 5：最新版本 HTML 语言，多了 32 个新标签
- 广义HTML 5：包括 CSS3 等朋友们

### HTML 5 技术集合

- 新标签、新属性
- 新的通信技术：WebSockets、WebRTC
- 离线存储技术：LocalStorage、断网检测
- 多媒体技术：视频、音频
- 图像技术：Canvas、SVG、WebGL
- Web增强技术：History API、全屏
- 设备相关技术：摄像头、触摸屏
- 新的样式技术：CSS3新的 Flex、Grid 的布局方式

# HTML 起手式

用 `Emmet` 所提供的速写法可以很快地写出你在写 `HTML` 所需要写的一个骨架。
你只需要在安装了插件的编辑器（某些编辑器默认具有此功能）中输入 `!` 再敲击 `Tab`，便可以很方便地输入以下内容。
```html
<!DOCTYPE html> <!-- 表示文档类型是 HTML 5 -->
<html lang="en"> <!-- html 标签，可以在这里设置语言，比如 lang="zh-CN" -->
<head>  <!-- 这里的东西不会显示到页面上 -->
    <meta charset="UTF-8">  <!-- 文件的字符编码 -->
    <!-- 设置视口大小为设备宽度（以兼容手机），并设置初始缩放为 1.0 并禁用缩放 -->
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no"> 
    <meta http-equiv="X-UA-Compatible" content="ie=edge"> <!-- 让 IE 浏览器使用最新的内核 -->
    <title>Document</title> <!-- 标题 -->
</head> 
<body> <!-- 内容 -->
    
</body>
</html>
```