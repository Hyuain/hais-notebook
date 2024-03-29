---
title: JavaScript Arrays
date: 2020-01-29 17:22:52
categories:
  - - 前端
tags:
  - FX
---

JavaScript 的数组不是典型的数组，而是一个对象；**元素的数据类型可以不同，内存不一定连续，是通过字符串下标（而不是数字下标）获取元素**。

<!-- more -->


# 获得新数组

## 将类数组转化为数组

```js
args = Array.prototype.slice.call(arguments)
args = Array.from(arguments)
args = [...arguments]
args = Array.prototype.concat.apply([], arguments)
```

## 新建数组

```javascript
// 数组的简便定义
let arr = [1, 2, 3]

// 数组的标准写法
let arr = new Array(1, 2, 3)

// 或者只传入一个参数，表示数组的长度
let arr = new Array(3)
```

## 转化为数组

```javascript
// 以','分隔的字符串
let arr = '1,2,3'.split(',')

// 没有间隔的字符串
let arr = '123'.split('')
Array.from('123')

// 由对象转换
let arr = new Array(3)

// 数组转化为字符串
let str = [1, 2, 3].join(',')
```

数组对象除了 `__proto__` 之外，还包括 **索引** 和 **长度（`length`）** 这两个自身属性。

{% note warning %}

**伪数组**：伪数组的原型链中没有数组的原型

比如 `let divList = document.querySelector('div')` 将得到一个伪数组；一个普通的对象只是加上 `length` 属性，也将得到一个伪数组。

通常我们需要把它转化为数组来使用

`let divArray = Array.from(divList)`

{% endnote %}

## 合并数组

```js
let arr3 = arr1.concat(arr2) // 生成新数组，原数组不变
```

## 截取数组

```js
let arr2 = arr1.slice(2) // 生成新数组，原数组不变
```

## 复制数组

```js
let arr2 = arr1.slice(0) // 生成新数组，原数组不变
```

# 删除元素

```js
arr = ['a', 'b', 'c'];  delete arr['0']
// arr 为 [ empty, 'b', 'c']，如果3个都是 empty，称为稀疏数组，不推荐

// 改 length 也可以删除数组的元素，不推荐

arr.shift()
// 删除最开始的元素，并返回他，arr 被修改

arr.pop()
// 删除最后一个元素，并返回他，arr 被修改
 
arr.splice(2, 3)
// 从 2 开始，删除 3 个，并返回删除的部分，arr 被修改

arr.splice(2, 3, 'x', 'y')
// 从 2 开始，删除 3 个，增加 'x' 和 'y'
// 并返回删除的部分，arr 被修改
```

# 查找元素

## 遍历元素

```js
Object.keys(arr) // 不推荐
Object.values(arr) // 不推荐
for in // 不推荐

for(let i = 0; i < arr.length; i++) {
  console.log(`${i} : ${arr[i]}`)
}

arr.forEach(function(item, index, array) {
  console.log(`$(index) : $(item)`)
})

// 以上两种基本没有区别
// 但 for 关键字有 continue 和 break，forEach 没有
// for 是块级作用域，forEach 是函数作用域
```

## 查找单个元素

```js
arr.indexOf(item) // 有就会返回 index，没有就会返回 -1

arr.find(item => item % 2 === 0) 
// 会返回第一个符合条件的元素
arr.findIndex(item => item % 2 === 0)
// 会返回第一个符合条件的元素对应的索引
```

{% note warning %}

**索引越界**

```js
arr[arr.length] === undefined
a[-1] === undefined
```

{% endnote %}

# 增加元素

```js
arr.push()
// 在尾部添加，返回数组长度，arr 被修改

arr.unshift()
// 在头部添加，返回数组长度，arr 被修改

arr.splice(8, 0, 'x', 'y')
// 增加 'x' 和 'y'，并返回删除的部分（[]），arr 被修改
```

# 修改元素

```js
arr.reverse() // arr 被修改
arr.sort() // arr 被修改

arr.sort( function(a, b) {
  if ( a > b ) return 1
  else if ( a === b ) return 0
  else return -1
})
// 返回 1，a 在 b 之后
// 返回 0，不变
// 返回 -1，b 在 a 之后

arr.sort((a, b) => a.score - b.score) // 按 score 从小到大排序
```

# 数组变换

得到新数组，原数组不变

```js
// map： n 变 n
arr.map(item => item * item)

// filter：n 变少
arr.filter(item => item % 2 === 0)

// reduce：n 变 1
arr.reduce((sum, item) => sum + item, 0)
arr.reduce((result, item) => result.concat(item * item), [])
arr.reduce((result, item) =>
  result.concat(item % 2 === 1 ? [] : item), [])
```
