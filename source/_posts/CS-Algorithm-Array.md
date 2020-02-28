---
title: 简单算法：数组
date: 2020-02-11 17:18:46
tags:
  - 入门
  - 慕课网
  - FAQ
categories:
  - [计算机, 算法]
---

一些数组中涉及到的算法的题解。

<!-- more -->

# 电话号码的组合（公式运算）

```text
给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。
给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

示例:
输入："23"
输出：["ad", "ae", "af", "bd", "be", "bf", "cd", "ce", "cf"].

说明:
尽管上面的答案是按字典序排列的，但是你可以任意选择答案输出的顺序。
```

[LeetCode 原题链接](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

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

# 卡牌分组 （归类运算）

```text
给定一副牌，每张牌上都写着一个整数。
此时，你需要选定一个数字 X，使我们可以将整副牌按下述规则分成 1 组或更多组：
每组都有 X 张牌。
组内所有的牌上都写着相同的整数。
仅当你可选的 X >= 2 时返回 true。

示例 1：
输入：[1,2,3,4,4,3,2,1]
输出：true
解释：可行的分组是 [1,1]，[2,2]，[3,3]，[4,4]

示例 2：
输入：[1,1,1,2,2,2,3,3]
输出：false
解释：没有满足要求的分组。

示例 3：
输入：[1]
输出：false
解释：没有满足要求的分组。

示例 4：
输入：[1,1]
输出：true
解释：可行的分组是 [1,1]

示例 5：
输入：[1,1,2,2,2,2]
输出：true
解释：可行的分组是 [1,1]，[2,2]，[2,2]

提示：
1 <= deck.length <= 10000
0 <= deck[i] < 10000
```

[LeetCode 原题链接](https://leetcode-cn.com/problems/x-of-a-kind-in-a-deck-of-cards)

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

# 种花问题（筛选运算）

# 格雷编码（二进制运算）

