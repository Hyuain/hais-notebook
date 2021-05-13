---
title: JavaScript 世界
date: 2020-01-29 19:48:35
tags:
  - 入门
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

介绍 JavaScript 这个世界从诞生开始的故事。

<!-- more -->

# 开机

## 固件

- 固件是固定在主板上的存储设备，里面有开机程序
- 开机程序会将文件里的 OS 加载到内存里运行

## 操作系统（以 Linux 为例）

- 首先加载操作系统内核
- 然后启动初始化进程，编号为 1，每个进程都有 PID
- 启动系统服务：文件、安全、联网
- 等待用户登录：输入密码登录 / ssh 登录
- 登录后，运行 shell，用户就可以和操作系统对话了
- bash 是一种 shell，图形化界面可认为是一种 shell

# 打开浏览器

## chrome.exe

- 运行 chrome.exe 文件
- 开启 chrome 进程，作为主进程
- 主进程会开启一些辅助进程，如网络服务、GPU 加速
- 每新建一个标签，就有可能开启一个子进程

## 浏览器的功能

- 发起请求、下载 HTML、解析 HTML、下载 CSS、解析 CSS、渲染界面、下载 JS、解析 JS、执行 JS
- 功能模块：用户界面、渲染引擎、JS 引擎、存储等，这些功能模块属于不同的线程
    - JS 通过**跨线程通信**，使渲染引擎重新渲染
    - JS 是单线程的
    - DOM 操作慢：跨线程通信慢

## JavaScript 引擎

- 浏览器用的什么 JS 引擎？
    - Chrome 用的是 V8 引擎，C++ 编写
    - 网景用的是 SpiderMonkey，后被 Firefox 使用，C++
    - Safari 用的是 JavaScriptCore
    - IE 用的是 Chakra（JScript 9）
    - Edge 用的是 Chakra (JavaScript)
    - Node.js 用的是 V8 引擎
- 主要功能
    - 编译：把 JS 代码翻译为机器能执行的字节码或机器码
    - 优化：改写代码，使其更加高效
    - 执行：执行上面的字节码或机器码
    - 垃圾回收：把 JS 用完的内存回收，方便之后再次使用
    
## 运行 JavaScript 代码

- 准备工作
    - 提供 API： window、document、setTimeout（这些功能成为运行环境 runtime env）
- 内存图
    - ![内存图](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/RAM-Graph.png)
    - 红色区域（存放数据，但不会存变量名）
        - Stack 栈：每个数据顺序存放，非对象都存在 Stack
        - Heap 堆：每个数据随机存放，对象都存在 Heap（数组、函数）

## 堆内存与栈内存

JS 的内存空间分为

- 栈（Stack）：存放简单类型，LIFO
- 堆（Heap）：存放引用类型，是一种经过排序的树形结构，每个节点都有一个值，通常说的堆是指二叉堆，存取随意
- 池（一般也被归类为栈中）：存放常量，故又称常量池

### 堆与栈的优缺点

- 栈内存系统效率高，因为堆内存需要分配空间和地址，还要把地址存在栈内存中，因此效率低
- 引用类型变量大小不确定，如果放到栈内存中就无法轻易改变自己的大小，同时也浪费空间

闭包中的变量保存在堆内存中，而不是栈内存中，因此函数调用之后闭包仍然能够引用到函数内的变量

### 闭包与堆内存

```js
function A() {
  let a = 1
  function B() {
    console.log(a)
  }
  return B
}
let res = A()
```

函数 A 弹出调用栈之后，函数 A 中的变量这个时候是存储在堆上的，现在的 JS 引擎可以通过逃逸分析辨别出哪些变量需要存储在堆上，哪些需要存储在栈上

## V8 执行一段代码的过程

1. 首先通过词法分析和语法分析生成 **AST**
2. **解释器** 将 AST 转换为字节码
3. 由 **解释器** 逐行执行字节码，遇到热点代码启动 **编译器** 进行编译，生成对应的机器码，优化执行效率

### 生成 AST

#### 词法分析

主要是把一行行代码分解成一个个 token：

```js
let name = 'harvey'
```

会把句子拆分成四个部分（四个 token）：关键字 let、变量名 name、赋值 =、字符串 'harvey'

#### 语法分析

将生成的这些 token 数据，根据一定的语法规则转化为 AST，之后会生成执行上下文。
babel 实际上就是将 ES6 代码解析生成  ES6 的 AST，然后再转换为 ES5 的 AST，然后再转换为 ES5 的代码。

### 生成字节码

生成 AST 之后，直接通过 V8 解释器（Ignition） 生成字节码，但是字节码并不能让机器直接运行——之所以不直接转化为机器码，是因为机器码空间占用太大了。

字节码比机器码轻量很多，是介于 AST 和机器码之间的一种代码，但是与特定类型的机器码无关，字节码需要通过解释器将其转换为机器码然后执行。但和原来不同的是，现在不用一次性将全部的字节码都转换成机器码，而是通过解释器来逐行执行字节码，省去了生成二进制文件的操作，这样就大大降低了内存的压力。

### 执行代码

热点代码：在执行字节码的过程中，如果有一段代码反复出现，则记为 **热点代码**（HotSpot），这段代码会被编译成机器码保存起来。在这样的机制下，代码执行的时间越久，那么执行效率会越来越高，因为有越来越多的字节码被标记为热点代码。

JS 实际上并不是一门完全的解释型语言，因为字节码不仅配合了 **解释器**，还要配合 **编译器**。编译器和解释器的根本区别在于前者会编译生成二进制文件而后者不会。

# JavaScript 世界

{% cq %}
**JS 公式：`对象.__proto__ === 其构造函数.prototype`**
**根公理：`Object.prototype` 是所有对象的（直接或间接）原型**
**函数公理：所有函数都是由 `Function` 构造的**
{% endcq %}

![JSWorld-1](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-1.png)
![JSWorld-2](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-2.png)
![JSWorld-3](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-3.png)
![JSWorld-4](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/JS-World-4.png)
