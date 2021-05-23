---
title: 浏览器的工作原理
date: 2021-05-23 11:10:48
tags:
  - 入门
  - 这一秒是你的，下一秒就是我的
  - FAQ
categories:
  - [前端, 收集]
---

看一些文章做的笔记，备忘。
原文：[浏览器的工作原理](https://github.com/yacan8/blog/issues/28) 

<!-- more -->

# 浏览器架构

## 浏览器的多进程架构

### Chrome 的多进程架构

浏览器可以是单进程的，也可以是多进程的（不同的进程间可以通过IPC通信），不同的浏览器使用不同的架构，比如对于 Chrome：

- **浏览器进程 Browser Process**：负责浏览器 Tab 的前进、后退、地址栏、书签栏、网络请求、文件访问等
- **渲染进程 Renderer Process**：负责一个 Tab 内的显示相关工作
- **插件进程 Plugin Process**：负责控制插件
- **GPU 进程 GPU Process**：负责处理 GPU 任务

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/2.png)

当我们在浏览器地址栏输入 URL 后：

1. **Browser Process** 会向 URL 发请求，获取 HTML 内容，将 HTML 交给 **Renderer Process**
2. **Renderer Process** 解析 HTML 内容，如果遇到网络请求就又返回给 **Browser Process** 进行加载；同时通知 **Browser Process** 需要 **Plugin Process** 加载插件资源
3. 解析完成后，**Renderer Process** 计算得到图像帧，并将其交给 **GPU Process**
4. **GPU Process** 将图像帧转化为图像显示在屏幕上

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/3.png)

### 多进程架构的好处

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/4.png)

1. 容错性。防止崩溃传染
2. 安全性。浏览器针对不同的进程限制了不同的权限，并为其提供沙盒运行环境，使其更加安全可靠
3. 更高的响应速度。避免各个任务抢占 CPU 资源

### 多进程架构优化

一个 Tab 对应一个 **Renderer Process**，进程之间的内存无法共享，而不同的进程内存常常有相同的内容，此时需要优化。

#### 浏览器的进程模式

Chrome 提供了四种进程模式（Process Models），不同的进程会对 Tab 进程做不同的处理。

- **Process-per-site-instance** (default)：同一个 site-instance 使用一个进程
- **Process-per-site**：同一个 site 使用一个进程
- **Process-per-tab**：同一个 tab 使用一个进程
- **Single Process**：所有 tab 共用一个进程

其中：

- **site** 指的是相同的 domain name(比如 google.com) 和 scheme(比如 https://)。比如 a.baidu.com 和 b.baidu.com 就可以理解为一个 site（这与同源策略不同，同源策略还涉及子域名和端口）
- **site-instance** 指的是一组 connected pages from the same site。connected 即 can obtain references to each other in script code。比如满足下面两种情况，并打开的新页面和旧页面就属于同一个 site，那么他们就属于一个 site-instance：
  - 用户通过 `<a>` 标签打开的页面
  - 通过 Javascript 代码打开的新页面（比如 `window.open`）
  
举个栗子：若使用 **Process-per-site-instance** 模式，当打开一个 tab 访问 a.baidu.com，再打开一个 tab 访问 b.baidu.com，就是两个进程；若在 a.baidu.com 中，通过 Javascript 代码打开了 b.baidu.com，就会使用同一个进程。

因此实际上这是介于 **Process-per-site** 和 **Process-per-tab** 之间的一种模式。

# 导航过程

## 网页加载过程

**Browser Process** 又划分为不同的工作线程：

- **UI Thread**：控制浏览器上的按钮和输入框
- **Network Thread**：处理网络请求（Chrome 72以后，已将network thread单独摘成network service process，当然也可以通过 chrome://flags/#network-service-in-process修改配置，将其其作为线程运行在Browser Process中 的提出）
- **Storage Thread**：控制文件的访问

### 第一步：处理输入

在浏览器地址栏输入内容，按下回车之后 **UI Thread** 会判断输入的内容是搜索关键词还是 URL，然后跳转至搜索引擎或请求 URL。

### 第二步：开始导航

**UI Thread** 会将 URL 交给网络进程，并显示一个 Loading 图标。然后网络进程会进行诸如 DNS 寻址、建立 TLS 等连接进行资源请求。如果收到服务器的301重定向响应，它就会告知UI线程进行重定向然后它会再次发起一个新的网络请求。

### 第三步：读取响应

网络进程收到响应之后会开始解析 HTTP 响应报文，然后根据 `Content-Type` 字段确定媒体类型（MIME Type）：如果媒体类型是 HTML 文件，则会交给渲染进程进行下一步工作；如果是 Zip 或者其他文件，会把相关数据交给下载管理器。

与此同时，浏览器会进行 [Safe Browsing](https://safebrowsing.google.com/) 安全检查，有可能会展示一个警告页。除此之外，还会做 [CORB](https://www.chromium.org/Home/chromium-security/corb-for-developers) 检查来确定敏感的跨站数据不会发送到渲染进程。

### 第四步：查找渲染进程

检查完毕之后，网络线程会通知UI线程，说数据已经准备好了。UI线程会查找到一个渲染进程进行网页渲染。

### 第五步：提交导航

此时，数据和渲染进程都已经准备好了，浏览器进程会向渲染进程发送 IPC 消息来确认导航。渲染进程接收到数据后，会告诉浏览器进程导航已经提交了，页面开始加载。

这时导航栏会更新，安全指示符更新，访问历史列表更新。

### 第六步：初始化加载完成

渲染进程渲染完毕之后（页面及内部的 iframe 都触发了 onload 事件），会向浏览器进程发送 IPC 消息，告知浏览器进程。UI 线程就会取消 Loading 状态。

## 网页渲染原理

渲染进程其实主要就是渲染 HTML + CSS + JS，将其转化为用户可交互的 Web 页面。他包含这几个线程：

- **Main Thread**：一个主线程
- **Work Thread**：多个工作线程
- **Compositor Thread**：一个合成器线程
- **Raster Thread**：多个光栅化线程

### 构建 DOM

当渲染进程收到导航的确认信息之后，开始接受来自浏览器进程的数据。此时，主线程会转化为 DOM 对象。

### 子资源加载

在构建 DOM 的过程中，会解析到图片、CSS、Javascript 等资源，这些资源需要从网络或者缓存中获取，主线程在构建 DOM 过程中如果遇到了这些资源，逐一发起请求去获取。而为了提升效率，浏览器也会运行预加载扫描程序，如果 HTML 中存在 `img` `link` 等，预加载扫描程序会把这些请求传递给网络线程进行资源下载。

### Javascript 的加载与执行

构建 DOM 的过程中，如果遇到 `<script>` 标签，会停止对 HTML 的解析，去加载执行 Javascript 代码，因为 Javascript 代码可能会改变 DOM 的结构。

但我们也可以在 `<script>` 上添加 `async` `defer` 等属性，让浏览器不要阻塞。

> defer 仅适用于外部脚本。并且 defer 会保证脚本之间的**执行**的相对顺序，前面的脚本先执行，且一定会在 DOMContentLoaded 之前执行。
> async 不会保证脚本的相对顺序，谁先下载好谁先执行，且与 DOMContentLoaded 执行顺序无关。

### 样式计算：Style Calculation

主线程在解析页面时，遇到 `<style>` 或 `<link>` 标签的 CSS 资源，会加载 CSS 代码，并以此确定每个 DOM 节点的 **计算样式（Computed Style）**。

### 布局：Layout

主线程会遍历 **DOM** 及相关元素的 **计算样式**，构建出包含每个元素的页面 **坐标信息** 及 **盒子模型大小** 的布局树（Render Tree）。遍历过程中，会跳过隐藏的元素（display: none）；另外，伪元素虽然在 DOM 上不可见，但是在布局树上是可见的。

### 绘制：Paint

主线程会遍历布局树（Layout Tree），生成一系列的绘画记录（Paint Records）。绘画记录可以看做是记录各元素绘制先后顺序的笔记。

> 关于他们与 CSS 动画的关系，看这篇笔记：{% post_link CSS-Animation %}

### 合成：Compositing

将 文档结构、元素的样式、元素的几何关系、绘画顺序 这些信息转化为显示器中的像素的过程，叫做 **光栅化(Rasterizing)**。

