---
title: Cookie
date: 2020-01-30 09:45:22
tags:
  - 入门
categories:
  - [前端, 浏览器]
---

Cookie 是浏览器下发给浏览器的一段字符串，浏览器必须保存这个 Cookie，之后发起的相同二级域名的请求，浏览器必须附上 Cookie。

<!-- more -->

# Cookie

## Cookie 防篡改

思路一：加密，但是有安全漏洞（加密后的内容可以无限期使用）

思路二：把用户信息隐藏在服务器，这样我就可以自由控制 Cookie 的失效时间

- 把用户信息放在服务器的 `session` 里，再给信息一个随机 `id`
- 把随机的 `id` 发送给浏览器
- 后端读取的时候，通过 `session[id]` 获取用户信息

## Cookie、LocalStorage、SessionStorage 和 Session

### Cookie 和 Session 的区别

Cookie 是服务器发给浏览器的一段字符串，每次浏览器在访问对应域名的时候，都需要把这个字符串带上
Session 是会话，表示浏览器与服务器一段时间内的会话

- 一般 Cookie 在浏览器上，Session 在服务器上
- Session 一般是基于 Cookie 来实现的（把 SessionID 放到 Cookie 里面）

### Cookie 和 LocalStorage 的区别

- Cookie 上限一般为 4K，LocalStorage 为 5M
- Cookie 一般存储用户信息，LocalStorage 一般存储不重要的数据
- Cookie 需要发送到服务器上，LocalStorage 不发送到服务器上

### LocalStorage 和 SessionStorage 的区别

LocalStorage 一般不过期，SessionStorage 一般在 Session 结束的时候过期（比如关闭浏览器的时候）

# 同源策略

- 源：输入 `window.origin` 或者 `location.origin`，我们就可以看到 **源**，实际上他就是 **协议 + 域名 + 端口号**。
- 同源：当两个 url 的源（协议、域名、端口号）完全一致，则称之为 **同源**，比如 `https://baidu.com` 和 `https://www.baidu.com` 就不同源。
- 同源策略：**浏览器** 规定：如果 JS **运行在**源 A 里，那么就不能获取源 B 的数据，这就是 **同源策略** ——不允许不同源的资源 **跨域访问**。

{% note warning %}
要注意的是，同源策略限制的是 **数据的访问**，引用 CSS、JS 和图片的时候，其实并不知道其内容，只是在 **引用**，因此不受同源策略限制。
{% endnote %}

## CORS

> Cross-Origin Resource Sharing

只需要在响应头里面写 `Access-Control-Allow-Origin: http://foo.example` 就可以了，但是 IE 6、7、8、9 都不支持，得用 JSONP。

## JSONP

> JSONP 和 JSON 没有多大关系

让 `frank.com` 访问 `qq.com` 的方法：

1. `qq.com` 将 数据写到 `/friends.js`
2. `frank.com` 用 `script` 标签引用 `/friends.js`
3. `/friends.js` 执行，执行 `frank.com` 事先定义好的 `window.xxx` 函数（`window.xxx({friends:[...]})`）
4. 然后 `frank.com` 通过 `window.xxx` 获取到了数据，这也是一个回调

但是，JSONP 存在安全性问题：

- 因为每个人都可以引用 js，需要进行 `referer` 检查
- 仍然存在安全问题，如果 `frank.com` 被攻陷，则 `qq.com` 也被攻陷

如何自动生成 window.xxx（如何把 frank.com 定义好的函数传给后台）？

- 通过查询参数

> 什么是 JSONP？
> 背景：当前浏览器或者由于某些因素导致不支持跨域
> 方法：请求一个 JS 文件，文件会执行一个回调，回调里面有我们的数据，回调的名字可以随机生成，我们把名字用 callback 参数传给后台，后台再返回给我们再执行
> 优点：兼容 IE、可以跨域
> 缺点：由于是 `script` 标签，所以读不到 AJAX 那么精确的状态（比如 Status、响应头等等），并且只能发 `GET` 请求，不支持 `POST`

一个代码实现：

```js
function jsonp(settings) {
  const data = settings.data || {}
  const key = settings.key || 'callback'
  const callback = settings.callback || function() {}
  
  data[key] = '__onGetData__'
  window.__onGetData__ = function(response) {
    callback(response)
  }
  
  const query = []
  for (let key in data) {
    query.push(key + '=' + data.key)
  }

  const script = document.createElement('script')
  script.src = url + '?' + encodeURIComponent(query.join('&'))
  document.head.appendChild(script)
  document.head.removeChild(script)
}
```

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

> 关于他们与 CSS 动画的关系，看这篇笔记：{% post_link CSS %}

### 合成：Compositing

> 将 文档结构、元素的样式、元素的几何关系、绘画顺序 这些信息转化为显示器中的像素的过程，叫做 **光栅化(Rasterizing)**。

#### 什么是合成

##### 简单的绘制方式

最简单的绘制方法就是只光栅化视口（Viewport）内的内容，如果滚动了页面，就移动光栅帧（Rastered Frame）并且光栅化更多的内容：

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/15.gif)

Chrome 最开始就是使用的这种方式，他的缺点在于每当页面滚动，光栅线程都需要对新视图的内容进行光栅化——这会导致性能损耗。于是 Chrome 采用合成的方式来优化这种情况。

##### 采用合成的方式提高性能

合成就是 **将页面分成若干层**，然后分别对他们进行光栅化，最后在一个单独的线程——合成线程中合成一个页面。
当用户滚动页面的时候，由于各个层都已经光栅化了，浏览器只需要合成一个新的帧来展示滚动之后的效果。
页面动画也是类似，将页面上的层移动并构建出一个新的帧。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/16.gif)

#### 主线程与元素分层

**主线程** 需要遍历布局树来构建一棵层次树（Layer Tree）。对于有 [will-change](https://developer.mozilla.org/zh-CN/docs/Web/CSS/will-change) 属性的元素，会被看做是单独的一层；没有 `will-change` CSS 属性的元素，浏览器会根据情况决定是否把元素放在单独的层——如果层数过多，可能这个合成过程比光栅化还要慢。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/17.png)

#### 合成器线程、光栅化线程

一旦 Layer Tree 被创建，渲染顺序被确定，主线程会把这些信息通知给 **合成器线程**，**合成器线程** 开始对每一层进行光栅化。有的层可以达到整个页面的大小，因此合成器线程需要将他们切成小块的图块（tiles）。之后将这些小图块分别发送给一系列的 **光栅化线程** 进行光栅化，结束后 **光栅化线程** 会将每个图块的光栅化结果存在 **GPU 线程** 的内存中。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/18.png)

为了优化显示体验，**合成线程** 可以给不同的 **光栅化线程** 赋予不同的优先级，将那些在视口中或者视口附近的层优先光栅化。

当图层上面的图块都被光栅化后，**合成线程** 会收集图块上面的 **绘画四边形（Draw Quads）** 信息来构建一个 **合成帧（Compositor Frame）**

> 绘画四边形：包含图块在内存的位置、图块在页面的位置之类的信息
> 合成帧：代表一个帧的内容的绘画四边形的集合。

#### 提交合成帧

当以上步骤完成之后，合成线程会通过 IPC 向浏览器进程提交（Commit）一个渲染帧。这时浏览器线程的 UI 进程也可能提交合成帧来改变浏览器的UI。这些合成帧都会被发送给 GPU 然后展示在屏幕上。如果合成线程收到了页面滚动事件，他会再构建一个合成帧发送给 GPU 来更新页面。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/19.png)

由于合成的过程没有涉及到主线程，因此合成线程不需要等待样式的计算以及 Javascript 执行完成。因此 [合成相关的动画最流畅](https://www.html5rocks.com/en/tutorials/speed/high-performance-animations/)，比如 `transform` `opacity`。

# 浏览器处理事件

以点击事件为例，当鼠标点击页面的时候：

1. 浏览器进程 Browser Process 最先接收到事件信息，他知道了事件的类型和位置，然后把事件信息传递给渲染进程。
2. 渲染进程 Renderer Process 会根据事件发生的坐标，找到目标对象，并运行绑定的监听函数。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/20.png)

## 渲染进程中的合成器线程接收事件

比如对于页面滚动，如果我们没有绑定页面滚动事件，合成器线程可以独立于主线程创建合成帧；但如果页面绑定了滚动事件，合成器线程需要先等待主线程的事件处理完成之后才能创建合成帧。

当页面合成时，合成器线程会标记页面中有绑定事件处理器的区域为 **非快速滚动区域（Non-Fast Scrollable Region）**。如果这些区域发生事件，合成器线程就会先把事件发送给主线程，否则他就会直接合成新的帧。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/21.png)

因此，我们需要注意全局事件绑定，比如有时候我们使用事件委托，将目标元素的事件发送给根元素 body 来进行处理：

```javascript
document.body.addEventListener('touchstart', event => {
  if (event.target === area) {
    event.preventDefault()
  }
})
```

在这段代码下，整个 body 被绑定了事件监听器，也就意味着整个页面都视为非快速滚动区域，合成器每次都需要与主线程通信并且等待反馈。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/22.png)

我们为了提升性能可以这样改一下：

```javascript
document.body.addEventListener('touchstart', event => {
  if (event.target === area) {
    // 注意这时候就不能调用 preventDefault 了，因为合成器线程不会再等待主线程的反馈
  }
}, { passive: true })
```

## 查找事件对象 Event Target

当事件没有发生在非快速滚动区域的时候，合成器线程会向主线程发送事件信息，主线程获取到事件信息之后会通过命中测试（Hit Test）去找到事件的目标对象——命中测试会遍历在绘制阶段生成的绘画记录（Paint Record）来找到包含事件发生坐标上的元素对象。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/23.png)

## 浏览器对事件的优化

一般我们的屏幕是 60 fps，但某些事件触发频率可以超过这个数值（比如 `wheel` `mousewheel` `mousemove` `pointermove` `touchmove`，他们一般会每秒触发60~120次）。对于相对较低的屏幕刷新率来说，这样主线程就会触发过量的命中测试和 JavaScript 代码，浪费性能。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/24.png)

为了优化这个问题，浏览器会合并这些连续的事件，延迟到下一帧渲染时进行——也就是 `requestAnimationFrame` 之前。

![](https://raw.githubusercontent.com/yacan8/blog/master/images/%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/25.png)

