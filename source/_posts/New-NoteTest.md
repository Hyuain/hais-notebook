---
title: NEW_NOTE_TEST
date: 2020-01-30 09:45:22
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, 浏览器]
---

NEW_NOTE_TEST

<!-- more -->

# Cookie 防篡改

思路一：加密，但是有安全漏洞（加密后的内容可以无限期使用）

思路二：把用户信息隐藏在服务器，这样我就可以自由控制 Cookie 的失效时间

- 把用户信息放在服务器的 `session` 里，再给信息一个随机 `id`
- 把随机的 `id` 发送给浏览器
- 后端读取的时候，通过 `session[id]` 获取用户信息

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

LocalStorage 一般不过期，SessionStorage 一般在 Session 结束的时候过期（比如关闭浏览器的时候）