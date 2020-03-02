---
title: 算法小问题：排序
date: 2020-02-28 21:37:30
tags:
  - 入门
  - 收集
  - FAQ
categories:
  - [计算机, 算法]
---

介绍几种简单的排序方法。

可以看 [这个网站](https://visualgo.net/en/sorting) 的动图

<!-- more -->

# 选择排序

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

# 快速排序

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

# 冒泡排序

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