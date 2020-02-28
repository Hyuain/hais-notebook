---
title: FAQs：HTTP
date: 2020-02-13 20:31:32
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [前端, 其他]
---

关于 HTTP 的一些问题与答案。

<!-- more -->

# HTTP 状态码

# HTTP 缓存

## ETag

> MD5

实际上还是会发请求，命中了一般就是 304 Not Modified

## Expire

> 日期

时间点，可以通过修改本地时间改变

## Cache-Control

> max-age=600

时间段，是相对时间；而且是无请求的

## PWA

浏览器缓存顺序

# GET 和 POST 的区别？

- GET 不安全，POST 不安全（其实都不安全）
- GET 是有长度限制的，POST 没有长度限制（但一般 GET 的长度不超过 1 KB，POST 上限 4M ~ 20M 左右）
- GET 参数在 URL 中，POST 消息在消息体中
- GET 只需要一个报文，POST 需要两个以上（因为有消息体）
- GET 幂等，POST 不幂等
- 语义不同，GET 是为了获取数据，POST 是为了提交数据

# Cookie、LocalStorage、SessionStorage 和 Session

## Cookie 和 Session 的区别

Cookie 是服务器发给浏览器的一段字符串，每次浏览器在访问对应域名的时候，都需要把这个字符串带上
Session 是会话，表示浏览器与服务器一段时间内的会话

- 一般 Cookie 在浏览器上，Session 在服务器上
- Session 一般是基于 Cookie 来实现的（把 SessionID 放到 Cookie 里面）

## Cookie 和 LocalStorage 的区别

- Cookie 上限一般为 4K，LocalStorage 为 5M
- Cookie 一般存储用户信息，LocalStorage 一般存储不重要的数据
- Cookie 需要发送到服务器上，LocalStorage 不发送到服务器上

## LocalStorage 和 SessionStorage 的区别

LocalStorage 一般不过期，SessionStorage 一般在 Session 结束的时候过期

# HTTP/2 和 HTTP/1 的区别

