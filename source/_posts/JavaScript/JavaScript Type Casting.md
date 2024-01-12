---
title: JavaScript Type Casting
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 中的运算符会自带隐式类型转换，规则如下：

- `-` `* ` `/` `%` 会转换成数值后计算
- `+`
  - 数字 + 字符串 = 字符串，从左到右运算；+ 字符串 = 数字
  - 数字 + 对象，`toPrimitive` `valueOf` `toString`
  - 数字 + Boolean / `null` = 数字
  - 数字 + `undefined` = `NaN`
