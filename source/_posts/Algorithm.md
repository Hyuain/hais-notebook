---
title: 数据结构与算法
date: 2022-02-05 15:49:12
categories:
  - [计算机]
---

.

<!-- more -->

# 数组

## 查找

### 二分查找

[LeetCode.204](https://leetcode.cn/problems/binary-search/submissions/)

前提

- 数组是有序的
- 数组中无重复元素

时间复杂度 O(log(n))

```typescript
const binarySearch = (nums: number[], target: number) => {
  if (!nums.length) { return -1 }
  let left = 0
  let right = nums.length - 1
  while (l <= r) {
    let mid = Math.floor((left + right) / 2)
    if (nums[mid] === target) {
      return mid
    } else if (nums[mid] > target) {
      // 如果写成这样，意味着新的 right 需要在下一次循环被考虑
      // 所以当 left === right 的时候，循环还不能终止
      right = mid - 1
    } else {
      // 新的 left 总是需要被考虑的，所以下面这行一般没有第二种写法
      left = mid + 1
    }
  }
  return -1
}
```

## 排序

### 选择排序

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

### 插入排序

从后面未排序的数中取一个，插入前面已经排序好的数组中正确的位置，O(n^2)，最好 O(n)

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

### 冒泡排序

两两比较，如果后者比前者小，则交换，最终每次循环使得最大数冒泡到最后去，O(n^2)，最好 O(n)

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

### 归并排序

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

### 快速排序

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
  // j 表示未遍历数组的最后一项
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

### Sorting in JavaScript

早年的 ECMAScript Spec 并没有规定 `Array#sort` 方法的稳定性，甚至有的 JavaScript Engine 在短数组时是稳定的（插入排序），长数组中不稳定（快速排序）。但 V8 向 ECMAScript 发起了 [数组稳定性的提案](https://v8.dev/features/stable-sort)，并且已经被采纳了，现在的 JavaScript Engine （V8 v7.0 / Chrome 70）均已支持稳定的数组排序。

但具体采用什么样的算法来实现数组排序，在不同的 JavaScript Engine 中有着不同的实现。比如 V8 采用的是 [Timsort](https://v8.dev/blog/array-sort#timsort)。

#### Sorting in V8

- 在 JavaScript 这样的动态类型语言中，比较比访问内存更昂贵，因为两个数的比较通常涉及到用户代码的调用。
- 在 JavaScript 中，用户可以做很多奇怪的事情：
  - 在比较函数中改变数组；
  - 数组中可能有对象，其有 `toString` 方法，而我们默认的比较函数就会调用 `toString` 方法去比较；
  - 给数组的某些项增加访问器（getter 和 setter）；
  - 构造出原型链很奇怪的对象，并且要给这个对象数组排序。

##### Sorting Process

> 收集所有的非 `undefined` 值，放入一个临时列表，为这个临时列表排序，然后将其写回原来的数组或对象中。这使得 V8 可以在真正的排序中不用再关心访问器和原型链。

规范要求 `Array#sort` 排序好的列表可以分为三个部分：

1. 所有的非 `undefined` 值根据比较函数排序；
2. 所有的 `undefined`；
3. 所有的空洞（稀疏数组）。

实际上排序算法只需要应用给第一部分就可以了。

##### Before Sorting

V8 的排序预处理步骤如下：

1. 将 `length` 赋值为数组或对象的 `length` 属性；
2. 将 `numberOfUndefineds` 赋值为 `0`；
3. 对于所有在 `[0, length)` 区间内的 `value`：
   1. 如果 `value` 是空洞，什么也不做；
   2. 如果 `value` 是 `undefined`，`numberOfUndefineds++`；
   3. 否则将 `value` 加入临时列表 `elements`。

这样处理之后，所有的非 `undefined` 值被存在了 `elements` 中，所有的 `undefined` 仅仅只是被计数了（`undefiend` 不会被传给用户提供的比较函数）。

##### After Sorting

排序完成后，排序好的值将会被写回原来的数组或对象中。V8 的排序后处理步骤如下：

1. 将 `elements` 写回原来数组的 `[0, elements.length)` 区间内；
2. 将 `[elements.length, elements.length + numberOfUndefineds)` 区间设置为 `undefined`；
3. 删除区间 `[elements.length + numberOfUndefineds, length)` 的值。

##### During Sorting: Timsort

- 快排的好处是原地排序，可以减少内存占用。缺点是不稳定，并且最坏的情况下时间复杂度是 O(n^2)。

- Timsort 是 2002 年 Tim Peters 为 Python 开发的。
- Timsort 是迭代调用的（而归并排序通常是递归调用）。

> To Be Completed.

## 数组去重

> 恰当地利用对象和 Map key 的唯一性可以实现对 `'1'` 和 `1` 的区分、对象的去重，以及 `undefined` `null` `NaN` 的良好识别。

使用数组 `[ 1, 1, '1', '1', 0, 0, { a: 1 }, { a: 1 }, x, y, z, undefined, undefined, null, null, NaN, NaN ]` 输出进行实验，其中：

```js
const x = { b: 1 }
const y = { b: 1 }
const z = { a: 1 }
```

### 双重循环

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

### `indexOf`

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

### 相邻元素去重（先排序）

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

### `filter`

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

### `reduce`（先排序）

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

### `includes`

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

### 对象键值对

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

#### 改进版本 1

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

#### 改进版本 2

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

### Map

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

### Set

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

## 例题

### 电话号码的组合（公式运算）

[LeetCode.17](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/)

```js
const telComb = (str) => {
  const map = ['', 1, 'abc', 'def', 'ghi', 'jkl', 'mno', 'pqrs', 'tuv', 'wxyz']
  const num = str.split('')
  // 保存键盘映射后的字母内容，比如 23 => ['abc', 'def']
  const code = []
  num.forEach(item => {
    if (map[item]) {
      code.push(map[item])
    }
  })
  const combine = (arr) => {
    const result = []
    for (let i = 0; i < arr[0].length; i++) {
      for (let j = 0; j < arr[1].length; j++) {
        result.push(`${arr[0][i]}${arr[1][j]}`)
      }
    }
    // 递归的精髓，用临时结果 result 替换掉前面两项
    arr.splice(0, 2, result)
    if (arr.length > 1) {
      combine(arr)
    } else {
      return result
    }
    return arr[0]
  }
  return combine(code)
}
```

### 卡牌分组 （归类运算）

[LeetCode.914](https://leetcode.cn/problems/x-of-a-kind-in-a-deck-of-cards/)

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

# 字符串

## 例题

### 反转字符串中的单词

[LeetCode.557](https://leetcode.cn/problems/reverse-words-in-a-string-iii/)

```js
const reverse = (str) => {
  const array = str.split(' ')
  const result = array.map(item => {
    return item.split('').reverse().join('')
  })
  return result.join(' ')
}

// 更优雅的写法
const reverse = (str) => {
  return str
          .split(' ')
          .map(item => item.split('').reverse().join(''))
          .join(' ')
}

// 使用正则表达式
const reverse = (str) => {
  return str
          .split(/\s/g)
          .map(item => item.split('').reverse().join(''))
          .join(' ')
}

// 使用 match API
const reverse = (str) => {
  return str
          .match(/[\w']+/g)
          .map(item => item.split('').reverse().join(''))
          .join(' ')
}
```

### 计数二进制子串

[LeetCode.696](https://leetcode.cn/problems/count-binary-substrings/)

```js
const count = (str) => {
  // 建立数据结构，堆栈、保存数据
  const result = []
  // 给定任意子输入都返回第一个符合条件的字符串
  const match = (str) => {
    const j = str.match(/^(0+|1+)/)[0]
    const o = (j[0] ^ 1).toString().repeat(j.length)
    const reg = new RegExp(`${j}${o}`)
    if (reg.test(str)) {
      return RegExp.$1
    } else {
      return ''
    }   
  }
  // 通过 for 循环控制程序运行流程
  for (let i = 0, len = str.length - 1; i < len; i++) {
    const sub = match(str.slice(i))
    if(sub){
      result.push(sub)
    }
  }
  return result
}
```

# 栈

## 单调栈

一维数组，要寻找任一元素的右边或左边第一个比自己大或自己小的位置。

比如 `[73, 74, 75, 71, 71, 72, 76, 73]` 为例，输出下一个 **大于** 自己的位置，输出应该是 `[1, 2, 6, 5, 5, 1, -1, -1]`。

这个时候需要维持一个 **栈底到栈顶单调递减** 的单调栈。这里我们单调栈里面可以存放元素的下标（存放什么可以根据题目的需要定义，重要的是单调栈的核心思想，这个看下面模拟会有标注）。

遍历一遍数组，每次将遍历到的元素与栈顶元素相比较，有两种操作：

- **如果当前遍历的元素大于栈顶的元素**，说明栈顶元素找到了下一位比自己大的元素，栈顶出栈并记录结果；同时新的栈顶也要与当前遍历的元素比较，直到小于等于当前元素
- **如果当前遍历的元素小于等于栈顶元素**，说明还需要继续找，将其入栈（这样效果上就达成了一个栈底到栈顶递减的单调栈）

模拟步骤如下：

```text
const stack = []
const result = [-1, -1, -1, -1, -1, -1, -1, -1]

--i == 0--
!stack.size
stack.push(0)
result: [-1, -1, -1, -1, -1, -1, -1, -1]
stack: [0]

--i == 1--
74 > stack.top(73), 需要在 73 的位置标记已经找到了 74；73 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[0] = 1
stack.push(1)
result: [1, -1, -1, -1, -1, -1, -1, -1]
stack: [1]

--i == 2--
75 > stack.top(74), 需要在 74 的位置标记已经找到了 75；74 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[1] = 2
stack.push(2)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2]

--i == 3--
71 < stack.top(75), 将 71 入栈，待查找
stack.push(3)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2, 3]

--i == 4--
71 == stack.top(71), 将 71 入栈，待查找
stack.push(4)
result: [1, 2, -1, -1, -1, -1, -1, -1]
stack: [2, 3, 4]

--i == 5--
72 > stack.top(71), 需要在 71 的位置标记已经找到了 72；71 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[4] = 5
  出栈后 72 仍然大于 stack.top(71)，继续出栈，直到小于等于 stack.top 或栈为空
  result[3] = 5
stack.push(5)
result: [1, 2, -1, 5, 5, -1, -1, -1]
stack: [2, 5]

--i == 6--
76 > stack.top(72), 需要在 72 的位置标记已经找到了 76；72 的下一个大的已经找到，可以出栈
  result[stack.pop()] = i, result[5] = 6
  出栈后 76 仍然大于 stack.top(75)，继续出栈，直到小于等于 stack.top 或栈为空
  result[2] = 6
stack.push(6)
result: [1, 2, 6, 5, 5, 6, -1, -1]
stack: [6]

--i == 7--
73 < stack.top(76), 将 73 入栈，待查找
stack.push(7)
result: [1, 2, 6, 5, 5, 6, -1, -1]
stack: [6, 7]

-- End --
```

计算器中将中缀表达转为后缀表达其实也类似于单调栈的思想，有三种方式可以解读：

- 遍历中缀表达式，遇到运算符的时候，如果栈顶的运算符优先级更高或相同，将其出栈（保证其先被运算），直到遇到优先级更低的运算符或栈空时，将当前运算符入栈
- 栈维持了从 **栈底到栈顶运算符优先级递增**
- 转为后缀表达式其实是要找 **下一个优先级相同或更低的运算符的位置**，因为 **下一个优先级相同或更低的运算符的位置** 就是 **当前运算符覆盖范围截止的位置**

```text
1 + 2 * 3 - 4  => 1 2 3 * + 4 -

const stack = []
const result = []

--i == 0--
数字 1 直接放入结果
result: [1]

--i == 1--
!stack.size:
  stack.push('+')
stack: ['+']
result: [1]

--i == 2--
数字 2 直接放入结果
stack: ['+']
result: [1, 2]

--i == 3--
* 比 stack.top('+') 优先级更高，stack.push('*')
stack: ['+', '*']
result: [1, 2]

--i == 4--
数字 3 直接放入结果
stack: ['+', '*']
result: [1, 2, 3]

--i == 5--
- 比 stack.top('*') 优先级低，result.push(stack.pop())，说明 * 负责的范围已经结束了（2 和 3）
- 跟 stack.top('+') 优先级相同，result.push(stack.pop())，说明 + 负责的范围已经结束了（1 和 2 * 3）
stack.push('-')
stack: ['-']
result: [1, 2, '*', '+']

--i == 6--
数字 4 直接放入结果
stack: ['-']
result: [1, 2, '*', '+', 4]

-- End --
清空 stack，按照出栈顺序加入结果
result: [1, 2, '*', '+', 4, '-']
```

## 例题

### 实现一个简单的计算器

利用后缀表达和栈可以实现一个简单的计算器

#### 算式的后缀表达

我们平时使用的是中缀表达（Infix Notation），形式是 **操作数 运算符 操作数**，比如 1 + 1

后缀表达（Postfix Notation, Reverse Polish Notation）的形式是 **操作数 操作数 运算符**，比如 1 1 +

要得到后缀表达，只需要：

- 操作数顺序不变
- 操作符根据优先级加在操作数后面

比如：

```text
 10 + (44 - 1) * 3 + 9 / 2
 10       44       1       3       9       2
 10      (44       1 -)    3      (9       2 /)
 10     ((44       1 -)    3 *)   (9       2 /)
(10     ((44       1 -)    3 *) +)(9       2 /)
(10     ((44       1 -)    3 *) +)(9       2 /) +
 10 44 1 - 3 * + 9 2 / +
 
 1 - 2 - 5 + 6 / 5
 1 2 - 5 - 6 5 / +
 
 (22 / 7 + 4) * (6 - 2)
 22 7 / 4 + 6 2 - *
```

如何从中缀表达转换为后缀表达，可以看单调栈部分，以及后面的 Code

#### 计算器的例子

1. 先得到后缀表达

2. 遇到操作数入栈，遇到运算符则将将前两个出栈并进行运算
3. 将结果入栈

计算 3 - 2 - 1：

1. 先得到后缀表达 3 2 - 1

2. push 3
3. push 2
4. 遇到运算符 -
   1. pop 出 2
   2. pop 出 3
   3. 3 - 2 = 1
   4. push 1
5. push 1
6. 遇到运算符 -
   1. pop 出 1
   2. pop 出 1
   3. 1 - 1 = 0
   4. push 0
7. 结果是栈顶的 0

#### Code

```js
class Stack {
  array = []
  constructor() {}
  push(value) {
    return this.array.push(value)
  }
  pop() {
    return this.array.pop()
  }
  get size() {
    return this.array.length
  }
  get top() {
    return this.array[this.array.length - 1]
  }
}
```

```js
const priorityMap = new Map([
  ['(', 0],
  ['+', 1],
  ['-', 1],
  ['*', 2],
  ['/', 2],
  [')', 3],
])

// '1+2' -> ['1', '+', '2']
const splitString = (s) => {
  return s.split(/(\(|\)|\+|-|\*|\/)/).reduce((acc, cur) => {
    if (!cur.trim()) {
      return acc
    }
    return acc.concat(cur.trim())
  }, [])
}

// ['1', '+', '2'] -> [1, 2, '+']
const infix2Postfix = (infix) => {
  const postfix = []
  const stack = new Stack()
  // -1 -> 0-1
  if (infix[0] === '-') {
    infix.unshift('0')
  }
  infix.forEach((item) => {
    if (!priorityMap.has(item)) {
      // 数字直接加进结果
      postfix.push(Number(item))
    } else if (item === '(') {
      // 左括号入栈，待处理
      stack.push(item)
    } else if (item === ')') {
      // 遇到右括号，一直pop，直到对应的左括号
      // 相当于一段的完结，后面低优先级的运算符放在一段的最末位
      while (stack.top !== '(') {
        postfix.push(stack.pop())
      }
      // pop掉多余的左括号
      stack.pop()
    } else {
      // 遇到 + - * /，如果stack.top优先级大于等于当前位，就把stack.top给pop出来
      // 注意这里的等于，因为当优先级相同的时候，stack.top在前面出现，也应该pop出来
      while (stack.size && priorityMap.get(stack.top) >= priorityMap.get(item)) {
        postfix.push(stack.pop())
      }
      // 当前位入栈
      stack.push(item)
    }
  })
  // 整体的结束，与括号结束类似，剩下的低优先级的运算符放在末尾
  while (stack.size) {
    postfix.push(stack.pop())
  }
  return postfix
}

// [1, 2, '+'] -> 3
const calcPostfix = (postfix) => {
  const stack = new Stack()
  postfix.forEach((item) => {
    if (typeof item === 'number') {
      // 数字直接入栈
      stack.push(item)
    } else {
      // 遇到运算符，出栈两个数组，与运算符一起做运算
      const r = stack.pop()
      const l = stack.pop()
      if (item === '+') {
        stack.push(l + r)
      } else if (item === '-') {
        stack.push(l - r)
      } else if (item === '*') {
        stack.push(l * r)
      } else if (item === '/') {
        stack.push(l / r)
      }
    }
  })
  return stack.top
} 

const calc = (s) => {
  const splited = splitString(s)
  const postfix = infix2Postfix(splited)
  return calcPostfix(postfix)
}
```

# 队列

## 队列的实现

用环形队列可以节约内存，并且入队和出队都能得到 O(1) 的时间复杂度：

![环形队列](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithmn-Queue-Implementation.jpg)

```typescript
// 这里使用 TypeScript 仅供参考，提供此种实现方案的思路
class Queue {
  
  // 该队列最多 9 个元素
  private q = new Array(10)
  private front = 0
  private rear = 0
  
  constructor() {}
  

  enqueue(x) {
    if (this.isFull()) {
      throw new Error('Full')
    }
    this.q[this.rear] = x
    if (this.rear === this.q.length - 1) {
      this.rear = 0
    } else {
      this.rear++
    }
  }
  
  dequeue() {
    if (this.isEmpty()) {
      return
    }
    const x = this.q[this.front]
    if (this.front === this.q.length - 1) {
      this.front = 0
    } else {
      this.front ++
    }
  }
  
  
  isFull() {
    // rear 在 front 左边，表示套圈了
    // 另外注意 rear 指向的是队尾的最后一位，是一个空元素
    // 所以队列的最大容量是数组的长度 - 1
    return this.rear + 1 === this.front || this.front === 0 && this.rear === this.q.length - 1
  }
  
  isEmpty() {
    // rear 和 front 相等时，定义为空
    return this.rear === this.front
  }
}
```

# 二叉树

## 定义

```typescript
class TreeNode {
  public val: number
  public left: TreeNode | null
  public right: TreeNode | null
  constructor(val: number = 0, left: TreeNode | null = null, right: TreeNode | null = null) {
    this.val = val
    this.left = left
    this.right = right
  }
}
```

## 遍历

### 深度优先

#### 递归写法

递归三步：

1. 确定参数和返回值
2. 确定终止条件
3. 确定每次需要做的事情

##### 前序（中左右）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) {	return }
  cb?.(current)							// 中
  traversal(current.left)   // 左
  traversal(current.right)  // 右
}
```

##### 中序（左中右）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) { return }
  traversal(current.left)   // 左
  cb?.(current)             // 中
  traversal(current.right)  // 右
}
```

##### 后序（左右中）

```typescript
const traversal = (current: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!current) { return }
  traversal(current.left)   // 左
	traversal(current.right)  // 右
  cb?.(current)							// 中
}
```

#### 迭代写法

##### 前序（中左右）

![二叉树的前序遍历（迭代法）](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Tree-1.gif)

入栈顺序为 中、**右、左**，这样才能保证出栈顺序为 中、**左、右**

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]          // 中入栈
  while (stack.length) {
    const current = stack.pop() // 出栈，可以看出中最先出栈，然后是左，然后是右
    cb?.(current)								// 处理
    if (current.right) {
      stack.push(current.right) // 右入栈
    }
    if (current.left) {
      stack.push(current.left)  // 左入栈
    }
  }
}
```

##### 中序（左中右）

![二叉树的中序遍历（迭代法）](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Tree-2.gif)

需要借助指针来跟踪当前访问的节点

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const current = root
  const stack = []
  while (current || stack.length) {
    if (current) {
      stack.push(current)        // 入栈
      current = current.left     // 准备左入栈（第一次会一直入栈直到最左下角的元素，那才是处理的起点）
    } else {
      current = stack.pop()      // 出栈，可以看出左最先出栈，然后是中，然后是右
      cb?.(current)							 // 处理
      current = current.right    // 准备右入栈
    }
  }
}
```

##### 后序（左右中）

先写前序遍历（中左右），写的时候把左右入栈顺序颠倒，就是中右左，然后反转 result 数组就是后序遍历

```typescript
const traversal = (root: TreeNode | null): TreeNode[] => {
  if (!root) { return }
  const result = []
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    result.push(current)
    if (current.left) {
      stack.push(current.left)
    }
    if (current.right) {
      stack.push(current.right)
    }
  }
  return result.reverse()
}
```

#### 统一风格的迭代写法

使用标记法，在需要处理的元素入栈之后推入 `null`，这样栈里面就有两种元素：

1. 需要处理的元素，他后面会接一个 `null`
2. 需要进一步访问其子结点的元素，他后面不会接 `null`

##### 中序（左中右）

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    if (!current) {					   	// 出栈时遇到 null，说明下一个元素需要处理
      cb?.(stack.pop())
      continue
    }
    if (current.right) { stack.push(current.right) }	// 右入栈，待访问
    stack.push(current, null)													// 中入栈，待处理
    if (current.left) { stack.push(current.left) }    // 左入栈，待访问
  }
}
```

![二叉树的中序遍历（统一迭代法）](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Tree-3.gif)

##### 前序（中左右）

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    if (!current) {
      cb?.(stack.pop())
      continue
    }
    if (current.right) { stack.push(current.right) }
    if (current.left) { stack.push(current.left) }
    stack.push(current, null)
  }
}
```

##### 后序（左右中）

```typescript
const traversal = (root: TreeNode | null, cb?: (node: TreeNode) => any) => {
  if (!root) { return }
  const stack = [root]
  while (stack.length) {
    const current = stack.pop()
    if (!current) {
      cb?.(stack.pop())
      continue
    }
    stack.push(current, null)
    if (current.right) { stack.push(current.right) }
    if (current.left) { stack.push(current.left) }
  }
}
```

### 广度优先

> 即层序遍历

#### 迭代法

使用队列和一个缓存当前层的数组

```typescript
function traversal(root: TreeNode | null): number[][] {
  if (!root) { return [] }
  const queue: TreeNode[] = [root]
  const res = []
  while (queue.length) {
    const currentLevel = []
    const currentLevelSize = queue.length
    for (let i = 0; i < currentLevelSize; i++) {
      const current = queue.shift()
      currentLevel.push(current.val)
      if (current.left) { queue.push(current.left) }
      if (current.right) { queue.push(current.right) }
    }
    res.push(currentLevel)
  }
  return res
}
```

#### 递归法

传入 deep 可以不管当前递归到哪里了，自己属于哪一层

```typescript
function levelOrder(root: TreeNode | null): number[][] {
    function traversal(current: TreeNode | null, result: number[][], deep: number) {
        if (!current) { return }
        if (!result[deep]) { result[deep] = [] }
        result[deep].push(current.val)
        traversal(current.left, result, deep + 1)
        traversal(current.right, result, deep + 1)
    }
    const result = []
    traversal(root, result, 0)
    return result
}
```

## 二叉搜索树

### Search

```typescript
const search = (root: TreeNode | null, target: any) => {
  let current = root
  while (current) {
    if (current.value === target) {
      return current
    }
    current = current.value < target ? current.right : current.left
  }
  return null
}
```

### Insertion

```typescript
const insert = (root: TreeNode | null, node: TreeNode) => {
  if (!root) {
    return node
  }
  let current = root
  let prev
  // find a node to insert
  while (current) {
    prev = current
    if (node.value < current.value) {
      current = current.left
    } else if (node.value > current.value) {
      current = current.right
    } else {
      // already exist
      return root
    }
  }
  if (node.value < prev) {
    prev.left = node.value
  } else {
    prev.right = node.value
  }
}
```

### Deletion

- 如果没有子节点：直接删除
- 如果有一个子节点：用子结点替代该节点
- 如果有两个子结点：删除右子树最小节点 x，然后用 x 替代该节点
  - 称 x 为继任者，x 是比该节点大一位的节点
  - 也可以用比 x 节点小一位的节点（左子树的最大节点）来替代，但是这样得到的树高度差会更大（？）





# 动态规划

## 背包问题

![背包问题分类](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Bag-1.png)

### 01背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，得到的价值是 value[i] 。**每件物品只能用一次**，求解将哪些物品装入背包里物品价值总和最大。

假设物品如下：

|        | 重量 | 价值 |
| ------ | ---- | ---- |
| 物品 0 | 1    | 15   |
| 物品 1 | 3    | 20   |
| 物品 2 | 4    | 30   |

#### 暴力解法

每个物品有两个状态，选或不选，可以用回溯法暴力求解，比如树的每一层就对应着一个物品，左节点是加了的情况，右节点是不加的情况。

时间复杂度是 O(2^n)。

#### 二位数组动态规划

##### 1. 确定 dp 数组及下标的含义

`dp[i][j]` 表示从物品 `0 ~ i` 中任意取，放进容量为 `j` 的背包，其总价值是多少。

![dp[i][j]的含义](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Algorithm-Bag-2.png)

##### 2. 确定递推公式

- 不放物品 `i`，那么 `dp[i][j]` 就等于 `dp[i-1][j]`

- 放物品 `i`，背包容量为 `j-weight[i]` 时的价值为 `dp[i-1][j-weight[i]]`，在此基础上再加上 `value[i]` 即可
  - 注意 `j - weight[i]` 可能为负，这种情况背包容量为 j 时，只装物品 i 也装不下，`dp[i][j] = dp[i-1][j]`


```text
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
```

##### 3. dp 数组的初始化

- 背包容量 j 为 0 时，总价值为 0：`dp[i][0] = 0`
- 在放第 0 个物品（i 为 0）时：
  - 如果放不下（`j < weight[0]`），则 `dp[0][j] = 0`
  - 如果能放下（`j >= weight[0]`），则 `dp[0][j] = value[0]`

```javascript
// 放不下的情况
if (let j = 0; j < weight[0]; j++) {
  dp[0][j] = 0
}
// 能放下的情况
if (let j = weight[0]; j < bagweight; j++) {
  dp[0][j] = value[0]
}
```

其他的值初始化为什么都不影响（因为会被覆盖），我们可以先生成一个全是 0 的二维数组，然后初始化 `value[0]` （能放下物品 0 的情况）

##### 4. 确定遍历的顺序

根据递推公式，先遍历 i 还是 j 都不影响

##### 最终代码

```typescript
const weightBagProblem = (
  weight: number[], value: number[], size: number,
): number => {
  const goodsNum = weight.length
  const dp = new Array(goodsNum)
  	.fill(0)
  	.map((item) => new Array(size + 1).fill(0))
  // j = 0 的时候全部为 0
  // i = 0 的时候能放下时为 value[0]，否则为 0
  for (let j = weight[0]; j <= size; j++) {
    dp[0][j] = value[0]
  }
  for (let i = 1; i < goodsNum; i++) {
    for (let j = 1; j <= size; j++) {
      if (j < weight[i]) {
        dp[i][j] = dp[i - 1][j]
      } else {
        dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
      }
    }
  }
  return dp[goodsNum - 1][size]
}
```

#### 一维数组动态规划（滚动数组）

在二维数组中，递推公式如下：

```text
dp[i][j] = max(dp[i - 1][j], dp[i - 1][j - weight[i]] + value[i])
```

如果每次把第 `i - 1` 行复制到第 `i` 行，那么公式变为这样：

```text
dp[i][j] = max(dp[i][j], dp[i][j - weight[i]] + value[i])
```

 看出来都是同一行的数据，可以考虑用一维数组来解这个问题。

##### 1. 确定 dp 数组及下标的含义

`dp[j]` 表示容量为 `j` 的背包的最大价值。

##### 2. 确定递推公式

```text
dp[j] = max(dp[j], dp[j - weight[i]] + value[i])
```

也是两种情况，要么放 `i`，要么不放 `i`：

- 不放 `i`：`dp[j] = dp[j]`
- 放 `i`：`dp[j] = dp[j - weight[i]] + value[i]`

相当于就是上面二维数组的公式中把 `[i]` 去掉了。

##### 3. dp 数组的初始化

- 背包容量为 0 时，`dp[0] = 0`
- 根据递推公式，剩下的可以随意初始化，如果题目给的价值都是正整数，那么其他项初始化为 `0` 就可以了

##### 4. 确定遍历的顺序

注意：

- 根据递推公式，必须 `i` 在外层，`j` 在内层（相当于二维数组的逐行遍历），因为 `dp[i - 1][j - weight[i]]` 相当于要依赖上一行前几位的运算结果
- 遍历 `j` 的时候，一定要倒序遍历，因为顺序的话物品可能会被加入多次：

```javascript
weight[0] = 1; value[0] = 15;
dp[1] = dp[1 - weight[0]] + value[0] = 15
dp[2] = dp[2 - weight[0]] + value[0] = 30 // value[0] 又加入了一次
```

这个加入多次的问题在二维数组中不存在，因为：

- 二维数组第 0 行是通过初始化给的，我们遍历是从 1 开始的
- 之后每一行都是借助 **上一行** 的数据算出来的（看递推公式），所以顺序遍历本行并不会有影响

一维数组中，相当于 **把上一行的数据原位重复利用**，如果顺序遍历的话，相当于后面的数据用了 **本行** 的数据，就可能会重复放入物品

##### 最终代码

```typescript
const weightBagProblem = (
	weight: number[], value: number[], size: number,
) => {
  const goodsNum = weight.length
  const dp = new Array(size + 1).fill(0)
  for (let i = 0; i < goodsNum; i++) {
    // 这里必须从大遍历到小
    for (let j = size; j >= weight[i]; j--) {
      dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
    }
  }
  return dp[size]
}
```

### 完全背包

有 n 件物品和一个最多能背重量为 w 的背包。第 i 件物品的重量是 weight[i]，其价值是 value[i] 。**每件物品都有无限个（也就是可以放入背包多次）**，求解将哪些物品装入背包里物品价值总和最大。

假设物品如下：

|        | 重量 | 价值 |
| ------ | ---- | ---- |
| 物品 0 | 1    | 15   |
| 物品 1 | 3    | 20   |
| 物品 2 | 4    | 30   |

#### 遍历顺序问题

完全背包最核心的就是遍历顺序问题，在完全背包中需要从小到大进行遍历，因为一样物品可以被添加多次：

```js
  for (let i = 0; i < goodsNum; i++) {
    for (let j = weight[i]; j <= size; j++) {
      dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
    }
  }
```

##### 求最大价值时的遍历顺序

另外之前 0 1 背包中，要求物品在外层，重量在内层，因为下一行需要 `dp[i - 1][j - weight[i]]`，会依赖上一行前几位的运算结果

而在完全背包中，是可以重量在外层，物品在内层的。因为 `dp[j - weight[i]]`，依赖的是本行的运算结果，只有 `dp[0]` 是继承的上一行的数据，如果先遍历列，也可以拿到上一行 `dp[i - 1][0]`（因为是同一列的） 

```js
// 先遍历背包容量（列），再遍历物品（行）
for (let j = 0; j <= size; j++) {
  for (let i = 0; i < weight.length; i++) {
    if (j - weight[i] >= 0) {
      dp[j] = Math.max(dp[j], dp[j - weight[i]] + value[i])
    }
  }
}
```

##### 求装满方式时的遍历顺序

如果是问最大价值，dp 数组里面存的是最大价值，最大价值与怎样凑成最大价值无关，因此物品、重量谁在内层谁在外层并不关键。

但如果问背包的装满方式，遍历顺序就会影响很大了，得看题目问的是组合问题还是排列问题。

拿零钱兑换问题（[LeetCode.518 Coin Change II](https://leetcode.cn/problems/coin-change-ii/)）举例，给定一个总额和一堆无限多的币值，问总额有多少种组合方式。组合问题不关心顺序。

1. 确定 dp 数组及下标的含义：`dp[j]` 表示凑成总额为 `j` 时，有多少种钱币组合方式
2. 确定递推公式：`dp[j] += dp[j - coins[i]]`
3. 初始化：`dp[0] = 1`，否则后面推导出来的都是 0（但是这其实并没有什么真正的含义）。其他的初始化为 `0`
4. 确定遍历顺序

如果外层钱币（物品），内层总额（重量）：

```js
for (let i = 0; i <= coins.length; i++) {    // 物品
  for (let i = coins[i]; j <= amount; j++) { // 重量
    dp[j] += dp[j - coins[i]]
  }
}
```

假设 `coins = [1, 5]`，那么这种情况就是先把 1 带进去看，再把 5 带进去看，得到的方法数只会有 {1, 5}，而不会有 {5, 1}，因此 `dp[j]` 里面存的是组合数。

如果交换遍历顺序：

```js
for (let j = 0; j <= amount; j++) {        // 重量
  for (let i = 0; i < coins.length; i++) { // 物品
    if (j - coins[i] >= 0) {
      dp[j] += dp[j - coins[i]]
    }
  }
}
```

背包容量的每一个值，都会重新计算一遍 1 和 5，所以包含了 {1, 5} 和 {5, 1} 两种情况。，`dp[j]` 里面存的就是排列数。
