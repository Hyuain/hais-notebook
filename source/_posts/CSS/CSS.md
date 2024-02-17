---
title: Charset
date: 2020-02-01 20:12:49
categories:
  - - 前端
tags:
  - F1
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

其中 `@charset` 可以指定字符编码，具体可查看 [[Charset and Encoding]]。

# Concepts

CSS 中包含一些概念，比如：

- [[Document Flow|文档流]]
- [[Box Model|盒模型]]
- [[BFC]] 与 [[Margin Collapse|Margin 塌陷]]
- [[CSS Layout|CSS 布局]]
- [[CSS Positioning|CSS 定位]]
- [[Staking Context|层叠上下文]]
- [[Animation|动画]]
- [[Default Styles & CSS Reset|默认样式与 CSS Rest]]