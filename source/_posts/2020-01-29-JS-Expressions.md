---
title: JavaScript 表达式与语句
date: 2020-01-29 14:22:45
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

JavaScript 的表达式与语句，包括标识符、block 等概念。

<!-- more -->

# 表达式与语句

## 直观区别

- 表达式一般都有值，语句可能有也可能没有
- 语句一般会改变环境（声明、赋值），逗号表示语句没完

## 表达式

- `1 + 2`：值为 `3`
- `add(1, 2)`：值为函数的返回值
- `console.log`：值为函数本身
- `console.log(3)`：值为函数的返回值—— `undefined`

## 语句

- `var a = 1`

{% note warning %}
**注意**
`retrun` 后面不能接回车，否则相当于返回 `undefined`
{% endnote %}

## 标识符

第一个字符可以是 unicode 字符、$、_、中文，后面可以加数字

## 代码区块 block

简单来说就是把代码用大括号包在一起

## &&

如果前面是 **真的**，就执行后面的（若前面是假的，表达式的值为前面；若前面是真的，表达式的值为后面）

```js
window.f1 && console.log('f1 存在')
console && console.log && console.log('hi') // 因为 IE 没有 console.log，所以可以这样写防止出错
```

## ||

如果前面是 **假的**，就执行后面的

```js
a = a || 100 // 可以用于设置保底值
```

## for

```js
for(var i = 0; i < 5; i++){
	setTimeout(() => console.log(i))
}
// 5 5 5 5 5
for(let i = 0; i < 5; i++){
	setTimeout(() => console.log(i))
}
// 0 1 2 3 4
```

## label

```js
{ a:1 } // a 是一个 label，值是 1
```