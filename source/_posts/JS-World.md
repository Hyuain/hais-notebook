---
title: JavaScript 世界
date: 2020-01-29 19:48:35
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

介绍 JavaScript 这个世界从诞生开始的故事。

<!-- more -->

# 开机

## 固件

- 固件是固定在主板上的存储设备，里面有开机程序
- 开机程序会将文件里的 OS 加载到内存里运行

## 操作系统（以 Linux 为例）

- 首先加载操作系统内核
- 然后启动初始化进程，编号为 1，每个进程都有 PID
- 启动系统服务：文件、安全、联网
- 等待用户登录：输入密码登录 / ssh 登录
- 登录后，运行 shell，用户就可以和操作系统对话了
- bash 是一种 shell，图形化界面可认为是一种 shell

# 打开浏览器

## chrome.exe

- 运行 chrome.exe 文件
- 开启 chrome 进程，作为主进程
- 主进程会开启一些辅助进程，如网络服务、GPU 加速
- 每新建一个标签，就有可能开启一个子进程

## 浏览器的功能

- 发起请求、下载 HTML、解析 HTML、下载 CSS、解析 CSS、渲染界面、下载 JS、解析 JS、执行 JS
- 功能模块：用户界面、渲染引擎、JS 引擎、存储等，这些功能模块属于不同的线程
    - JS 通过**跨线程通信**，使渲染引擎重新渲染
    - JS 是单线程的
    - DOM 操作慢：跨线程通信慢

## JavaScript 引擎

- 浏览器用的什么 JS 引擎？
    - Chrome 用的是 V8 引擎，C++ 编写
    - 网景用的是 SpiderMonkey，后被 Firefox 使用，C++
    - Safari 用的是 JavaScriptCore
    - IE 用的是 Chakra（JScript 9）
    - Edge 用的是 Chakra (JavaScript)
    - Node.js 用的是 V8 引擎
- 主要功能
    - 编译：把 JS 代码翻译为机器能执行的字节码或机器码
    - 优化：改写代码，使其更加高效
    - 执行：执行上面的字节码或机器码
    - 垃圾回收：把 JS 用完的内存回收，方便之后再次使用
    
## 运行 JavaScript 代码

- 准备工作
    - 提供 API： window、document、setTimeout（这些功能成为运行环境 runtime env）
- 内存图
    - ![](/hais-notebook/images/JS-002.png)
    - 红色区域（存放数据，但不会存变量名）
        - Stack 栈：每个数据顺序存放，非对象都存在 Stack
        - Heap 堆：每个数据随机存放，对象都存在 Heap（数组、函数）
        
# JavaScript 世界

{% cq %}
**JS 公式：`对象.__proto__ === 其构造函数.prototype`**
**根公理：`Object.prototype` 是所有对象的（直接或间接）原型**
**函数公理：所有函数都是由 `Function` 构造的**
{% endcq %}

![](/hais-notebook/images/JS-003.png)
![](/hais-notebook/images/JS-004.png)
![](/hais-notebook/images/JS-005.png)
![](/hais-notebook/images/JS-006.png)
