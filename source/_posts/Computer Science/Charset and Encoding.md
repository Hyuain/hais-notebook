---
title: Charset and Encoding
date: 2020-02-01 20:12:49
categories:
  - - 前端
tags:
  - CX
---
> CSS 和 HTML 中的 charset 直译为 **字符集**，但 UTF-8 不是字符集（Charset），而是字符编码（Encoding），这是历史遗留问题。
 
- ASCII 字符集的编码形式就是 ASCII，这个字符集只有英文
- GB2312 字符集是中国人发明的简体中文字符集，他的编码形式也是 GB2312
- GBK 是微软发明的中日韩（CJK）字符集，他的编码形式也是 GBK。他使用 0000~FFFF，16 位，2 字节来表示
- Unicode 是支持所有字符的字符集
	- Unicode 已收录 13 万字符（大于 16 位），全世界通用，以后还会继续扩充，缺点是至少要三个字节
	- Unicode 的编码形式有 UTF-8/UTF-16/UTF-32
	- 其中 UTF-8 用变长的方法表示，每个字符（亦称为码点 Code Point）使用 1 ~ 4 个字节表示。因为 UTF-8 中最小的保存单位是字节，一个字节有 8 位，因此得名 UTF-8。与之相对应的 UTF-16 则以两个字节为最小单位，一个码点用 2 或 4 个字节表示