---
title: 数据结构与算法
date: 2022-02-05 15:49:12
categories:
  - [计算机]
---

.

<!-- more -->

# 数组

## 排序

### 选择排序

每次从剩下的部分选择最小的排到前面来，O(n^2)

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
    if (i !== index) {
      let temp = numbers[index]
      numbers[index] = numbers[i]
      numbers[i] = temp
    }
  }  
}
```

### 快速排序

找出 Pivot，把小于 Pivot 的放左边，大于 Pivot 的放右边，最好 O(nlogn)，最坏 O(n^2)

```js
const quickSort = numbers => {
  if (numbers.length < 2) return numbers
  let pivotIndex = Math.floor(numbers.length / 2)
  let pivot = numbers.splice(pivotIndex, 1)[0]
  let left = []
  let right = []
  for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] <= pivot) {
      left.push(numbers[i])
    } else {
      right.push(numbers[i])
    }
  }
  return quickSort(left).concat([pivot], quickSort(right))
}
```

### 冒泡排序

两两比较，如果后者比前者小，则交换，最终每次循环使得最大数冒泡到最后去（O(n^2)，最好 O(n)）

```js
const bubbleSort = numbers => {
  let length = numbers.length
  while (length > 1) {
    let swapped = false
    for (let i = 0; i < numbers.length - 1; i++) {
      if (numbers[i + 1] < numbers[i]) {
        let temp = numbers[i + 1]
        numbers[i + 1] = numbers[i]
        numbers[i] = temp
        swapped = true
      }
    }
    if (!swapped) break
    length--
  }
}
```

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

