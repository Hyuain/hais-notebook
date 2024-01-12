---
title: JSON
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

> JavaScript Object Natation，JSON 不是对象，而是一种标记语言（就像 HTML、XML、Markdown），用来展示数据，[JSON 中文官网](http://json.org/json-zh.html)

<!-- more -->

# JSON 的数据类型

- string（但是只支持双引号）
- number（支持科学计数法）
- bool
- null
- object
- array

{% note warning %}
JSON 中，不支持函数和变量
{% endnote %}

# JSON.parse()

> 反序列化

- 将符合 JSON 语法的字符串转换为 JS 对应类型的数据
- JSON 字符串 ⇒ JS 数据
- 由于 JSON 只有六种类型，所以转换出的数据也只有六种
- 如果不符合 JSON 语法，则会抛出 Error，一般用 try catch 捕获错误，比如

```js
let object
try {
  object = JSON.parse(`{'name':'harvey'}`)
} catch (error) {
  console.log('出错了，错误详情是：')
  console.log(error)
  object = {'name': 'no name'}
}
console.log(object)
```

# JSON.stringify()

> 序列化

- 是 JSON.parse 的逆运算
- JS 数据 ⇒ JSON 字符串
- 由于 JS 的数据类型比 JSON 多，所以不一定能够成功
- 如果失败，则会抛出一个 Error 对象
