---
title: JavaScript Expressions & Statements
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

# Expressions & Statements

- **表达式（Expressions）** 一般都有值，语句可能有也可能没有
- **语句（Statements）** 一般会改变环境（声明、赋值），逗号表示语句没完

```js
// 表达式
1 + 2          // 值为 3
add(1, 2)      // 值为函数的返回值
console.log    // 值为函数本身
console.log(3) // 值为函数的返回值：undefined

// 语句
var a = 1
```

{% note warning %}
`return` 后面不能接回车，否则相当于返回 `undefined`
{% endnote %}

# Identifier

在 JavaScript 中，**标识符（Identifier）** 只能包含字母或数字或下划线（`_`）或美元符号（`$`），且不能以数字开头（有时候也可以用其他的 Unicode 字符，比如中文、Emoji 等）

# Block

**代码块（Block）** 简单来说就是把代码用大括号包在一起

# For

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

# Label

```js
{ a:1 } // a 是一个 label，值是 1
```

