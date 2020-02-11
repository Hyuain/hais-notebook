---
title: 同源策略与跨域
date: 2020-01-30 09:38:05
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, 浏览器]
---

关于同源策略、CORS 和 JSONP 等的笔记。

<!-- more -->

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

