---
title: Array
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

> 数组是存放在连续内存空间上的相同类型数据的集合

- 数组的下标从 0 开始
- 数组的内存空间地址是连续的

# Operations of Array

| 操作 | 时间复杂度 |
| ---- | ---------- |
| 访问 | O(1)       |
| 查找 | O(n)       |
| 插入 | O(n)       |
| 追加 | O(1)       |
| 删除 | O(n)       |

# Searching in Array

## Binary Search

[LeetCode.204](https://leetcode.cn/problems/binary-search/submissions/)

前提

- 数组是有序的
- 数组中无重复元素

时间复杂度 O(log(n))

{% note Code %}
```typescript
// 左闭右闭写法
const binarySearch = (nums: number[], target: number) => {
  if (!nums.length) { return -1 }
  let left = 0
  let right = nums.length - 1
  // 当 left === right 时，区间 [left, right] 依旧有效
  while (l <= r) {
    let mid = Math.floor((left + right) / 2)
    if (nums[mid] === target) {
      return mid
    } else if (nums[mid] > target) {
      // 因为是右闭区间，新的 right 需要在下一次循环被考虑
      right = mid - 1
    } else {
      // 因为是左闭区间，新的 left 需要在下一次循环被考虑
      left = mid + 1
    }
  }
  return -1
}

// 左闭右开写法
const binarySearch = (nums: number[], target: number) => {
  if (!nums.length) { return -1 }
  let left = 0
  let right = nums.length
  // 当 left === right 时，区间 [left, right) 无效了
  while (l < r) {
    let mid = Math.floor((left + right) / 2)
    if (nums[mid] === target) {
      return mid
    } else if (nums[mid] > target) {
      // 因为是右开区间，新的 right 不需要被循环考虑
      right = mid
    } else {
      // 因为是左闭区间，新的 left 需要在下一次循环被考虑
      left = mid + 1
    }
  }
  return -1
}
```
{% endnote %}

# Sorting

## Selection Sort

每次从剩下的部分选择最小的排到前面来，O(n^2)

- 不稳定（交换操作通常都不稳定）
  - `8* 3 5 4 8 3 1 9` -> `1 3 5 4 8 8* 9`
- 原地

```js
const minIndex = numbers => {
  let index = 0
  for (let i = 1; i < numbers.length; i++) {
    if (numbers[i] < numbers[index]) {
      index = i
    }
  }
  return index
}

const selectionSort = numbers => {
  for (let i = 0; i < numbers.length - 1; i++){
    let index = minIndex(numbers.slice(i))
    if (index) {
      [numbers[index + i], numbers[i]] = [numbers[i], numbers[index + i]]
    }
  }  
  return numbers
}
```

## Insertion Sort

从后面未排序的数中取一个，插入前面已经排序好的数组中正确的位置，最坏 O(n^2)，最好 O(n)

- 稳定
- 原地

```js
const insertionSort = numbers => {
  for (let i = 0; i < numbers.length - 1; i++) {
    let j = i - 1
    const current = numbers[i]
    while (j >= 0 && current < numbers[j]) {
      numbers[j + 1] = numbers[j]
      j--
    }
    numbers[j + 1] = current
  }
  return numbers
}
```

## Bubble Sort

两两比较，如果后者比前者小，则交换，最终每次循环使得最大数冒泡到最后去，最坏 O(n^2)，最好 O(n)

- 稳定
- 原地

```js
const bubbleSort = numbers => {
  let length = numbers.length
  while (length > 1) {
    let swapped = false
    for (let i = 0; i < length; i++) {
      if (numbers[i + 1] < numbers[i]) {
        [numbers[i + 1], numbers[i]] = [numbers[i], numbers[i + 1]]
        swapped = true
      }
    }
    if (!swapped) break
    length--
  }
  return numbers
}
```

## Merge Sort

拆成左右两个数组，分别对左右数组进行排序（当长度为 1 的时候，自然是排好序的数组），再合并起来（排序实际上发生在合并的那一步），O(nlogn)

- 稳定
- 非原地

```js
const merge = (left, right) => {
  let i = 0
  let j = 0
  const merged = []
  while (i < left.length && j < right.length) {
    if (left[i] <= right[j]) {
      merged.push(left[i])
      i++
    } else {
      merged.push(right[j])
      j++
    }
  }
  if (i === left.length) {
    // left 已经合并完了，将 right 剩下的部分放进去
    merged.push(...right.slice(j))
  } else {
    // right 已经合并完了，将 left 剩下的部分放进去
    merged.push(...left.slice(i))
  }
  return merged
}

const mergeSort = numbers => {
  // Conquer
  if (numbers.length <= 1) { return numbers }
  // Devide
  const divisionIndex = Math.floor(numbers.length / 2)
  // Combine
  return merge(
    // Conquer
    mergeSort(numbers.slice(0, divisionIndex)),
    mergeSort(numbers.slice(divisionIndex))
  )
}
```

## Quick Sort

找出 Pivot，把小于 Pivot 的放左边，大于 Pivot 的放右边，最好 O(nlogn)，最坏 O(n^2)

- 不稳定
- 原地

```js
// 非原地写法
const quickSort = numbers => {
  if (numbers.length < 2) return numbers
  // Devide
  let pivotIndex = Math.floor(numbers.length / 2)
  let pivot = numbers[pivotIndex]
  let left = []
  let right = []
  // Conquer
  for (let i = 0; i < numbers.length; i++) {
    if (i === pivotIndex) { continue }
    if (numbers[i] <= pivot) {
      left.push(numbers[i])
    } else {
      right.push(numbers[i])
    }
  }
  // Combine
  return quickSort(left).concat([pivot], quickSort(right))
}
```

```js
// 原地写法
const partition = (numbers, l, r) => {
  // 选择最后一个数作为 pivot
  const pivot = numbers[r]
  // 相当于整个数组被分为
  // |左边部分|右边部分|未遍历数组|pivot|
  // i 表示 pivot 左边部分的末尾的下一项（即右边数组的第一项）
  let i = l
  // j 表示未遍历数组的第一项
  for (let j = l; j < r; j++) {
    if (numbers[j] < pivot) {
      // 如果该项小于 pivot，应该放到左边部分中，于是 j 和 i 互换
      // 因为 i 是左边部分最后一项的下一位
      [numbers[i], numbers[j]] = [numbers[j], numbers[i]]
      i++
    }
  }
  // 右边部分的第一项与 pivot 互换，使得 pivot 在左右部分的中间
  [numbers[i], numbers[r]] = [numbers[r], numbers[i]]
  // 返回 pivot 的下标
  return i
}

const quickSort = (numbers, l = 0, r = numbers.length - 1) => {
  // l 表示需要考虑的切片的开始的坐标
  // r 表示需要考虑的切片的结束的坐标（l r 都需要被考虑）
  // l == r 时，只有一项，不需要再算了
  if (l < r) {
    const pivotIndex = partition(numbers, l, r)
    quickSort(numbers, l, pivotIndex - 1)
    quickSort(numbers, pivotIndex + 1, r)
  }
  return numbers
}
```

## Heap Sort

见 [[Heap#Heap Sort|堆排序]]。

## Sorting in JavaScript

早年的 ECMAScript Spec 并没有规定 `Array#sort` 方法的稳定性，甚至有的 JavaScript Engine 在短数组时是稳定的（插入排序），长数组中不稳定（快速排序）。但 V8 向 ECMAScript 发起了 [数组稳定性的提案](https://v8.dev/features/stable-sort)，并且已经被采纳了，现在的 JavaScript Engine （V8 v7.0 / Chrome 70）均已支持稳定的数组排序，可以参考 [V8 中的排序](https://v8.dev/blog/array-sort) 博客。

但具体采用什么样的算法来实现数组排序，在不同的 JavaScript Engine 中有着不同的实现。比如 V8 采用的是 [Timsort](https://v8.dev/blog/array-sort#timsort)。

### Sorting in V8

- 在 JavaScript 这样的动态类型语言中，比较比访问内存更昂贵，因为两个数的比较通常涉及到用户代码的调用。
- 在 JavaScript 中，用户可以做很多奇怪的事情：
  - 在比较函数中改变数组；
  - 数组中可能有对象，其有 `toString` 方法，而我们默认的比较函数就会调用 `toString` 方法去比较；
  - 给数组的某些项增加访问器（getter 和 setter）；
  - 构造出原型链很奇怪的对象，并且要给这个对象数组排序。

#### Sorting Process

> 收集所有的非 `undefined` 值，放入一个临时列表，为这个临时列表排序，然后将其写回原来的数组或对象中。这使得 V8 可以在真正的排序中不用再关心访问器和原型链。

规范要求 `Array#sort` 排序好的列表可以分为三个部分：

1. 所有的非 `undefined` 值根据比较函数排序；
2. 所有的 `undefined`；
3. 所有的空洞（稀疏数组）。

实际上排序算法只需要应用给第一部分就可以了。

#### Before Sorting

V8 的排序预处理步骤如下：

1. 将 `length` 赋值为数组或对象的 `length` 属性；
2. 将 `numberOfUndefineds` 赋值为 `0`；
3. 对于所有在 `[0, length)` 区间内的 `value`：
   1. 如果 `value` 是空洞，什么也不做；
   2. 如果 `value` 是 `undefined`，`numberOfUndefineds++`；
   3. 否则将 `value` 加入临时列表 `elements`。

这样处理之后，所有的非 `undefined` 值被存在了 `elements` 中，所有的 `undefined` 仅仅只是被计数了（`undefiend` 不会被传给用户提供的比较函数）。

#### After Sorting

排序完成后，排序好的值将会被写回原来的数组或对象中。V8 的排序后处理步骤如下：

1. 将 `elements` 写回原来数组的 `[0, elements.length)` 区间内；
2. 将 `[elements.length, elements.length + numberOfUndefineds)` 区间设置为 `undefined`；
3. 删除区间 `[elements.length + numberOfUndefineds, length)` 的值。

#### During Sorting: Timsort

- 快排的好处是原地排序，可以减少内存占用。缺点是不稳定，并且最坏的情况下时间复杂度是 O(n^2)。
- Timsort 是 2002 年 Tim Peters 为 Python 开发的。
- Timsort 是迭代调用的（而归并排序通常是递归调用）。

> To Be Completed.

# Deduplication

> 恰当地利用对象和 Map key 的唯一性可以实现对 `'1'` 和 `1` 的区分、对象的去重，以及 `undefined` `null` `NaN` 的良好识别。

使用数组 `[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]` 输出进行实验，其中：

```js
const x = { b: 1 }
const y = { b: 1 }
const z = { a: 1 }
```

## 双重循环

```js
Array.prototype.unique = function() {
  const newArray = []
  let isRepeat
  for (let i = 0; i < this.length; i++) {
    isRepeat = false
    for (let j = 0; j < newArray.length; j++) {
      if(this[i] === newArray[j]){
        isRepeat = true
        break
      }
    }
    if (!isRepeat) {
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN, NaN]
```

不能区分：对象，`NaN`

## `indexOf`

```js
Array.prototype.unique = function() {
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    const current = this[i]
    if (newArray.indexOf(current) < 0) {
      newArray.push(current)
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN, NaN]
```

不能区分：对象，`NaN`

## 相邻元素去重（先排序）

```js
Array.prototype.unique = function() {
  const newArray = []
  this.sort()
  for (let i = 0; i < this.length; i++) {
    if(this[i] !== this[i+1]) {
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 0, 1, '1', NaN, NaN, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, null]
```

注意：原来的数组会被更改！

不能区分：对象，`NaN`，`undefined`（会丢失）

## `filter`

```js
Array.prototype.unique = function() {
  return newArray = this.filter((item, index, array) => {
    return array.indexOf(item) === index
  })
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null]
```

不能区分：对象，`NaN`（会丢失）

## `reduce`（先排序）

```js
Array.prototype.unique = function() {
  return this.sort().reduce((result, current) => {
    if (result.length === 0 || result[result.length - 1] !== current) {
      result.push(current)
    }
    return result
  }, [])
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 0, 1, '1', NaN, NaN, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, null, undefined]
```

注意：原来的数组会被更改！

不能区分：对象，`NaN`

## `includes`

```js
Array.prototype.unique = function() {
  const newArray = []
  this.forEach(item => {
    if (!newArray.includes(item)) {
      newArray.push(item)
    }
  })
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

注意：原来的数组会被更改！

不能区分：对象

## 对象键值对

利用对象中 key 的不可重复性去重

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if (!obj[this[i]]) {
      obj[this[i]] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, 0, { a: 1 }, undefined, null, NaN]
```

不能区分：对象（存进去的时候所有对象都变成了 `[object Object]`）、`'1'` 和 `1`

### 改进版本 1

可以区分 `'1'` 和 `1`

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if (!obj[typeof this[i] + this[i]]) {
      obj[typeof this[i] + this[i]] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

### 改进版本 2

可以区分对象

```js
Array.prototype.unique = function() {
  const obj = {}
  const newArray = []
  for (let i = 0; i < this.length; i++) {
    if(!obj[typeof this[i] + JSON.stringify(this[i])]) {
      obj[typeof this[i] + JSON.stringify(this[i])] = true
      newArray.push(this[i])
    }
  }
  return newArray
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { b: 1 }, undefined, null, NaN]
```

## Map

利用 Map 的 key 唯一

```js
Array.prototype.unique = function() {
  const newArray = []
  const map = new Map()
  for (let i = 0; i < this.length; i++) {
    if(!map.get(this[i])) {
      map.set(this[i], true)
      newArray.push(this[i])
    }
  }
  return newArray
}

// 简化版
Array.prototype.unique = function() {
  const map = new Map()
  return this.filter((item) => (!map.has(item) && map.set(item, true)))
}
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

不能去重：对象，可以像上面的对象键值对方法一样处理

## Set

Set 类似于数组，但成员的值都是唯一的

```js
Array.prototype.unique = function() {
  return Array.from(new Set(this))
}

// 可以简化为
Array.prototype.unique = function() {
  return [...new Set(this)]
}

// 还可以简化为
```

```js
[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]
    |
    ↓
[ 1, '1', 0, { a: 1 }, { a: 1 }, { b: 1 }, { b: 1 }, { a: 1 }, undefined, null, NaN]
```

不能去重：对象

# Examples

## 电话号码的组合

[LeetCode.17](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

Given a string containing digits from 2-9 inclusive, return all possible letter combinations that the number could represent. Return the answer in any order.

A mapping of digits to letters (just like on the telephone buttons) is given below. Note that 1 does not map to any letters.

{% note Code %} 

#回溯

  ```typescript
  function letterCombinations(digits: string): string[] {
    const letters = [
      "", "", "abc", "def", "ghi", "jkl", "mno", "pqrs", "tuv", "wxyz"
    ]
    const result = []
    const currentChosenLetters = []
    function trackingback() {
      if (currentChosenLetters.length === digits.length) {
        if (currentChosenLetters.length) {
          result.push(currentChosenLetters.join(""))
        }
        return
      }
      const currentAvaliableLetters = letters[digits[currentChosenLetters.length]]
      for (let i = 0; i < currentAvaliableLetters.length; i++) {
        currentChosenLetters.push(currentAvaliableLetters[i])
        trackingback()
        currentChosenLetters.pop()
      }
    }
    trackingback()
    return result
  };
  ```

{% endnote %}

## 卡牌分组 （归类运算）

[LeetCode.914](https://leetcode.cn/problems/x-of-a-kind-in-a-deck-of-cards/)

{% note Code %}

问题的核心是求最大公约数：

```text
辗转相除： gcd(a, b) = gcd(b, a % b)
更相减损： gcd(a, b) = a > b ? gcd(a - b, b) : gcd(a, b - a)
```

```js
const cardGroup = (arr) => {
  // 先排序
  const str = arr.sort().join('')
  // 再将相同的进行分组（多张或者单张为一组）
  const group = str.match(/(\d)\1+|\d/g)
  // 求最大公约数
  const gcd = (a, b) => {
    if (b === 0) {
      return a
    } else {
      return gcd(b, a % b)
    }
  }
  // 进入处理之前检查，如果 group 为空或者 group.length === 1，就不找公约数
  while (group.length > 1) {
    const a = group.shift().length
    const b = group.shift().length
    const v = gcd(a, b)
    if (v === 1) {
      return false
    } else {
      group.unshift('0'.repeat(v))
    }
  }
  // 如果 group 为空，或者每组的数量小于 1，则为 false
  return group.length ? group[0].length > 1 : false
}
```

{% endnote %}
