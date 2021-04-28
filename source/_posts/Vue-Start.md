---
title: Vue 开始
date: 2020-01-30 10:31:34
tags:
  - 入门
  - 文档
categories:
  - [前端, JavaScript, 框架, Vue]
---

Vue 笔记的开始，主要记录了 Vue 的安装与历史。

<!-- more -->

# 历史

- 2015 年，1.0 版，是 MVVM 框架
- 2016 年，2.0 版，没有完全遵循 MVVM 模型
- 2019 年，2.6 版
- 2020 年，3.0 版，完全不是 MVVM

# 创建一个 vue 项目

- 使用 @vue-cli
- 或者使用 webpack 或者 rollup 从零开始

# 完整版和运行时版本

- **完整版**：同时包含编译器和运行时的版本，也就是 CDN 里面的 `vue.js`，可以直接在页面里面写 ，相当于把视图放在 html 里面写
- **运行时**：用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码。基本上就是除去编译器的其它一切，也就是 CDN 里面的 `vue.runtime.js`，不支持从 html 里面获取视图，也不支持在 template 里面写，没有编译器（compiler），代码体积小 30%
- webpack 中的 vue-loader 可以在最后打包的时候将 template 里面的东西编译成 JS，因此我们不需要使用完整版

|  | 完整版 | 运行时版 | 评价 |
| --- | --- | --- | --- |
特点 | 有 compiler | 没有 compiler | compiler 占 40% 的体积
视图 | 写在 HTML 里，或者写在 template 选项中 | 写在 render 函数里，用 h 来创建标签	h 是 Vue 写好传给 render 的
CDN 引入 | vue.js | vue.runtime.js | 文件名不同，生产环境后缀为 .min.js
webpack 引入 | 需要配置 alias | 默认使用此版 | 	 | 
@vue/cli 引入 | 需要额外配置 | 默认使用此版	 |   | 