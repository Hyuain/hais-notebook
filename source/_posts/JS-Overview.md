---
title: JavaScript 序章
date: 2020-01-29 14:12:35
tags:
  - 入门
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

JavaScript 的历史和评价的简单介绍。

<!-- more -->

# 历史

1994 年，网景公司（当时叫 Mosaic Communications）发布了一款名为 **Mosaic Netscape** 的网页浏览器，在四个月内，这款浏览器就占据了四分之三的浏览器市场，并成为 1990 年代互联网的主要浏览器。

> 因为世界最早流行的图形接口网页浏览器 **NCSA Mosaic** 是美国国家超级电脑应用中心（NCSA）与 1993 年发布的，网景公司为了避免版权纠纷，将浏览器改名为 **Netscape Navigator**，而公司则改名为 **Netscape Communications**。

这款浏览器发布之后，网景意识到，**光有静态的页面是不行的，需一种网页脚本语言，使得浏览器可以与网页互动。**

1995 年，昇阳（Sun）正式向市场推出 Java，网景公司看到 Java 的前景，决定与之结盟，并在浏览器中支持 Java，但如果直接将Java作为脚本语言嵌入网页，只是因为这样会使HTML网页过于复杂。

同年，网景招募了布兰登（Brendan Eich），授意其开发一款 **“未来的脚本语言”** ，这种语言需要：“看上去与Java足够相似，但是比Java简单，使得非专业的网页作者也能很快上手。”——这个决定就排除了 Perl、Python、Tcl 或 Scheme 这些选项，同时也促成了 JavaScript 的诞生。

由于对 Java 不感兴趣，布兰登只用了十天时间就设计出了这款语言的原型，并命名为 **Mocha**，后续又改名为 **LiveScript**，但在 1995 年 12 月，公司为了蹭 Java 的热度，改名为 **JavaScript**。而事实上，JavaScript 和 Java 关系并不大。

> 总的来说，布兰登的设计思路是这样的：
> 
> 1. 借鉴 C 的基本语法；
> 2. 借鉴 Java 的数据类型和内存管理；
> 3. 借鉴 Scheme，将函数提升到“第一等公民”（first class）的地位；
> 4. 借鉴 Self，使用基于原型（prototype）的继承机制。

由于 JavaScript 在浏览器上的大获成功，微软（Microsoft）在后续推出的 IE 3 上也使用了 **JScript** ——这与 JavaScript 是类似、但不同标准的语言。于是当年市场上出现了两者对峙的情况，网页设计者通常会在主页放上“用 Netscape可达到最佳效果 ”或“用 IE 可达到最佳效果”的标志。

1996 年 11 月，网景正式向 **欧洲计算机制造商协会（ECMA）** 提交语言标准；1997 年 6 月，ECMA 以 JavaScript 语言为基础制定了 ECMAScript 标准规范 ECMA-262。自然 JavaScript 也成为了 ECMAScript 最著名的实现之一。

由于只有短短十天的设计时间，而且世界上之前没有出现过结合了函数式编程和对象编程的语言，以及发展的迅速导致没有时间调整设计，JavaScript 成功成为了有着众多设计缺陷的语言，在这里不做细谈。

2001 年，微软发布 Windows XP，并捆绑了 IE 6。由于 Windows XP 迅速爆火以及长期的垄断，IE 6 也随之占据非常高的市场份额。前文已经说过，IE 6 对 JavaScript 支持并不好，同时 IE 6 对 CSS 标准的支持也不尽完善，导致前端技术的发展进入了漫长的蛰伏期。

2004 年，谷歌（Google）发布爆款应用 Gmail。这款应用在刚推出时，容量就比起其他受欢迎的电子邮箱服务如雅虎和微软的 Hotmail 多出过百倍，成为市场爆品，同时也让众多开发者看到了页面交互的巨大前景和可能性。

2005 年，Jesse 将谷歌用到的技术命名为 AJAX。

2006 年，至今为止最为长寿的 JavaScript 库—— jQuery，发布。

2008 年，谷歌发布 Chrome 浏览器；同年，Chrome 的使用率上升至 1%。其使用高性能 JavaScript 引擎 V8。

2009 年，Ryan 基于 V8 写了 Node.js。

2010 年，Isaac 基于 Node.js 写了 npm。

2010 年，TJ 受 Sinatra 启发，写了 Express.js。赶上了这几波顺风车的 JavaScript 迅速发展，并将触手伸向了后端。自此，JavaScript 也能胜任后端的一些工作了。

2012 年，Chrome 全球占有率达到 33%，超越 IE 跃居首位。

2015 年 12 月，Chrome 中国占有率达到 37%，超越 IE。

# 评价

- JS 是历史的选择，最开始浏览器支持 Java、Flash、VBScript，只有 JS 活到了最后
- JS 低开高走，一开始是一个玩具语言，但每次都走对了风口（ECMA 标准、Gmail、移动端、Node.js）
- 对于初学者，目前可以忽略与 IE 相关的知识