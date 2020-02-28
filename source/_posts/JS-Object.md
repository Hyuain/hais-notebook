---
title: JavaScript Object
date: 2020-01-29 16:00:51
tags:
  - 入门
  - 饥人谷
categories:
  - [前端, JavaScript, 原生 JavaScript]
---

简单来说，对象实际上就是无序的数据集合，或者说是 **键/值对（Key/Value Paris）**的集合.

<!-- more -->

# 创建对象

对象有以下这样几种写法：

```js
// 对象的简便声明
let obj = {
    'name': 'harvey',
    'age': 18
    }
// 对象的标准写法
let obj = new Object({
    'name': 'harvey',
    'age': 18
})
// 对象也可以作为参数直接传入函数
console.log({
    'name': 'harvey',
    'age': 18
})
```

在这里我们需要注意以下几个细节：

- 键名是字符串，不是标识符，可以是任意字符
- 引号可以省略，省略之后就只能写标识符或者以数字开头
- **就算引号省略了，键名也还是字符串**（不一定，键名也可能是函数、日期、正则等）
- 奇怪的属性名： `1e2` 会变成 `'100'`， `.234` 会变成 `'0.234'`， `0xFF` 会变成 `'255'`
- 变量也可以作为属性名，比如：`let obj = { [p1]: 'harvey' }`，这样就会用 `p1` 里面的值了，**中括号里面的东西都会先求值**

# 对象的属性

## 隐藏的属性

**每一个** 对象都有一个隐藏属性 `__proto__`，这个属性存着 **一个对象的地址**，这个对象包含了这类对象（普通对象、数组、函数等）的 **共有属性**

`__proto__` 里面存的实际上就是 **原型的地址**

因此，**每一个对象都有原型**

比如 `obj = {}`，他的原型的地址就存储在 `obj.__proto__` 中，而`obj.__proto__` 也是一个对象，因此他也有原型，但我们规定，他的原型值为 `null`

## 属性的删除

有以下两种删除对象属性的方法：

1. `obj.name = undefined`，这样做只会删除属性的值，不会把属性完全删除
2. `delete obj.name`，同时删除属性名和属性值，或者用 `delete obj['name']` 也是可以的

删除完成后可以对删除的结果进行检查：

1. `'name' in obj`，检查 `'name'` 是不是 `obj` 的属性名，如果是用上面的第一种方法删除，检查的结果将是 `true`；如果是第二种方法删除，则会返回 `false`；注意属性名有引号（因为属性名实际上是字符串）
2. `'name' in obj && obj.name === undefined`，检查是否含有属性名且值为 `undefined`，如果是上面第一种方法删除，则会返回 `true`

## 属性的查看

- `Object.keys(obj)`，查看 `obj` 的 **自身** 属性名
- `Object.values(obj)`，查看 `obj` 的  **自身** 属性值
- `Object.entries(obj)`，返回结果包含两个数组，第一个数组是 `obj` 的 **自身** 属性名，第二个数组是 `obj` 的 **自身** 属性值
- `console.dir(obj)`，查看 `obj` 的 **自身属性 + 共有属性**
- `in`，查看是不是 **自身属性 + 共有属性** （相当于所有属性）
- `obj.hasOwnProperty('toString')`，查看 `'toString'` 是不是 `'obj'` **自身的** 属性

{% note warning %}
`obj.name` 等价于 `obj['name']`，不等价于 `obj[name]`
{% endnote %}

## 添加或修改属性

直接赋值：

```javascript
obj.name = 'harvey'
obj['name'] = 'harvey'
obj['na'+'me'] = 'harvey' // 因为属性名本质是字符串，上面三句话实际上是一样的
```

批量赋值：

```javascript
Object.assign(obj, {p1:1, p2:2, p3:3})
```

但是，不能直接修改共有属性（原型上的属性）：比如，不能通过 `obj.toString` 来修改原型上的`'toString'`，这样只会为 `obj` 增添一个本身的`'toString'` 属性，而不会修改原型，除非这样写代码（但是这是不推荐的）：

```js
// 修改原型的属性
obj.__proto__.toString =
Object.prototype.toString =

// 修改原型
let person = Object.create(common)
// 相当于
person.__proto__ = common // 原型链增加一个环节
```

# 对象的原型

{% note warning %}
所有的函数自带 prototype
prototype 中自带 constructor
constructor 里面的东西就是函数的内容
{% endnote %}

## new X()

1. 自动创建空对象
2. 自动为空对象关联原型，原型的地址为 `X.prototype`
3. 自动将空对象作为 `this` 关键字运行构造函数
4. 自动 `return this`（也就是说可以接着写 `new X().getName()`）

```js
// 自己实现一个类似 new 的函数
function X(a, b) {
    this.a = a + 1
    this.b = b + 1
}

const y = new X(1, 2)

function NEW(fun, ...args){
    const newObj = Object.create(obj.prototype)
    // 相当于
    // let newObj = {}
    // newObj.__proto__ = fun.prototype
    const result = fun.apply(newObj, args)
    // 原版的 new 中，如果构造函数返回一个对象，则 new 也返回一个对象；如果构造函数返回一个简单类型，则 new 返回刚刚创建的新对象
    return typeof result === 'object' ? result : newObj
}

const z = NEW(X, 3, 4)
```

## 构造函数 `X`

- `X` 自身用于添加新对象的**自身的属性**
- `X.prototype` 负责保存对象的**共用属性**

一般来讲有这样的代码规范：

- 所有构造函数首字母大写，被构造出来的对象首字母小写
- `new` 后面的函数使用名词；其他的函数一般用动词开头

## 如何确定一个对象的原型？

```js
对象.__proto__ === 其构造函数.prototype
```

## 数据类型与对象的分类

>  数据类型是 JavaScript 数据的类型，一共有 7 种；
>  对象的分类则有无数种，常见的有 Array、Function、Date、RegExp 等

我们常见的有这几种类型的对象：

- 数组对象
  - 自身属性：`'0'` `'1'` `'2'` `'length'`
  - 共有属性：`'push'` `'pop'` `'shift'` `'unshift'` `'join'`

- 函数对象
  - 自身属性：`'name'` `'length'`
  - 共有属性：`'call'` `'apply'` `'bind'`
  
# 其他问题

> **Q: window 是谁构造出来的？**
> A: 可以通过 constructor 属性看出构造者

> **Q: window.Object 是谁构造的？**
> A: window.Function，所有的函数都是 window.Function 构造的

> **Q: window.Function 是谁构造的？**
> A: window.Function，所有的函数都是 window.Function 构造的，浏览器构造了 Function，然后指定它的构造者是自己