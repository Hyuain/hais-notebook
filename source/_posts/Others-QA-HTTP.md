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

# HTTP/2 和 HTTP/1 的区别

