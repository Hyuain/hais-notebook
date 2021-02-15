---
title: CSS 序章
date: 2020-02-01 20:12:49
tags:
  - 入门
categories:
  - [前端, CSS]
---

记录 CSS 的含义及基本语法。

<!-- more -->

# 层叠的含义

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

# charset

- charset 是字符集的意思，但 UTF-8 不是字符集，而是字符编码(encoding)，这是历史遗留问题
- ASCII 字符集的编码形式就是 ASCII，这个字符集只有英文
- GB2312 字符集是中国人发明的简体中文字符集，他的编码形式也是 GB2312
- GBK 是微软发明的中日韩（CJK）字符集，他的编码形式也是 GBK
- unicode 是支持所有字符的字符集，他的编码形式有 UTF-8/UTF-16/UTF-32

# 找资料

- 查资料：[MDN](https://developer.mozilla.org/zh-CN/docs/Web/CSS)、[CSS Tricks](https://css-tricks.com/)、[张鑫旭的博客](https://www.zhangxinxu.com/wordpress/)
- 找素材（可以搜 PSD WEB）：[Freepik](https://www.freepik.com/)、[Dribbble](http://dribbble.com/)、[365 PSD](https://cn.365psd.com/)