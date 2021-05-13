---
title: 读书笔记：《计算机科学精粹》
date: 2021-05-09 19:48:23
tags:
  - 入门
  - 慕课网
  - FAQ
categories:
  - [读书笔记, 计算机]
mathjax: true
---

有幸得贵人赠书《计算机科学精粹》，将纸质版读书笔记提炼誊抄至此。

<!-- more -->

# 第一章：预备知识

## 逻辑

### 运算符

> 我们称 `->` 为条件运算符，可读作「若……，则……」

一个示例：

```text
A: 泳池暖和
B: 我去游泳
A -> B: 如果泳池暖和，我就去游泳
```

#### 换质位法

指的是这两句话含义相同（即逆否命题与原命题含义相同）：

```text
A -> B: 如果泳池暖和，我就去游泳
!B -> !A: 如果我不去游泳，那么泳池就不暖和
```

{% note warning %}
`A -> B` 和 `B -> A` 含义并不相同！
{% endnote %}

#### 双条件

若要使 `A -> B` 和 `B -> A` 都成立，则需要使用双条件：

```text
A <-> B: 当且仅当泳池暖和，我才去游泳
```

### 布尔代数

#### 结合律

```text
A and ( B and C ) = ( A and B ) and C
A or ( B or C ) = ( A or B ) or C
```

#### 分配律

```text
A and ( B or C ) = ( A and B ) or ( A and C )
A or ( B and C ) = ( A or B ) and ( A or C )
```

#### 德摩根定律

```text
!( A and B ) = !A or !B
!( A or B ) = !A and !B 
```

## 计数

### 乘法

两个不相关事件的可能性用乘法计算：

$$
{m}\cdot{n}
$$

### 排列

n 个项的排列方式种数：

$$
n!
$$

n 个元素中选 m 个排成一列：

$$
A_{n}^{m} = \frac{n!}{(n-m)!}
$$

n 个元素中，r 个相同排成一列：

$$
\frac{n!}{r!}
$$

### 组合

n 个不同元素，选 m 个并成一组（与顺序无关）：

$$
\tbinom{n}{m} = C_{n}^{m} = \frac{n!}{m!(n-m)!}
$$

### 概率

- 独立事件：两个事件的结果不会相互影响，概率相乘
- 互斥事件：两个事件结果不能同时发生，概率相加
- 对立事件：两个互斥事件概率涵盖所有可能的结果，概率之和为 1

# 第二章：复杂度

## 时间复杂度的计算

例：选择排序

```javascript
function selectionSort(list) {
  // 外层循环执行 n-1 次，每次执行赋值、交换两项操作，共 2n-2 次
  for (current = 1 ... list.length -1) {
    smallest = current
    // 内层循环执行 n-1（外层第一次）、n-2（外层第二次）、n-3 次，共计 (n*n-n)/2 次赋值与交换操作，也就是 n*n-n 次操作 
    for (i = current + 1 ... list.length) {
      if (list[i] < list[current]) {
        smallest = i
      }
    }
    list.swap_items(current, smallest)
  }
}
```

因此上述选择排序算法总计进行了 $T(n) = n^2 + n - 2$ 次操作

## 大 O 符号

表示最坏情况下算法成本函数的主项，比如选择排序的时间复杂度可记为 $O(n^2)$

## 空间复杂度的计算

比如对于选择排序，只需要一组固定的变量作为存储空间，因此空间复杂度可以记为 $O(1)$

# 第三章：策略

本章主要讲解这几种策略：

- 通过 **迭代** 处理 *重复性* 任务
- 通过 **递归** 进行优雅地 *迭代*
- 资源允许时使用 **蛮力法**
- 测试不可行的选择并 **回溯**
- 采用 **启发法** 合理缩短求解的时间
- 采用 **分治法** 求解难题
- **动态地** 识别重复问题，避免浪费资源
- 对问题 **定界** 缩小求解范围

## 迭代

```text
     ┌──────┐循环(每一步操作就被称为迭代)
     │      │                           满足条件 
  求解过程 ────────────────────────────────────────────> 目标
     ↑      │
     └──────┘
```

例：合并两个排好序的数组

比如将 `[1, 3, 5, 7]` 与 `[2, 4, 6, 8]` 合并为 `[1, 2, 3, 4, 5, 6, 7, 8]`

```javascript
function merge(list1, list2) {
  result = []
  while (list1.has_item || list2.has_item) {
    if (list1.top_item > list2.top_item) {
      temp = list1.remove_top_item
    } else {
      temp = list2.remove_top_item
    }
    result.push(fish)
  }
  return result
}
```

例：嵌套循环与幂集

> 对于给定对象 S 的集合，其幂集是 S 的全部子集构成的集合

比如有几种花，花可以调配香水，我们需要求出能调配的所有可能的香水。

使用迭代的思维：
- 若没有花，一种无味的香水，记做0。
- 若有一种花：0、A
- 若有两种花：0、A、0+B、A+B
- 若有三种花：0、A、0+B、A+B、0+C、A+C、0+B+C、A+B+C

不难发现，多加一种花，就是把之前的复制一遍，再给复制之后的每种香水加上新的花。

```javascript
function powerSet(flowers) {
  fragrances = Set.new
  fragrances.add(Set.new)
  for (flower in flowers) {
    // 复制已有的香水
    new_fragrances = copy(fragrances)
    for (fragrance in new_fragrances) {
      // 给复制之后的香水加上新的花
      fragrance.add(flower)
    }
    fragrances = fragrances + new_fragrances
  }
  return fragrances
}
```

这种构建幂集的方式时间复杂度为 $O(n^2)$，每增加一次循环，时间将会增加一倍。这种方法也可以用于构建真值表，每种花是否加入用布尔值进行表示。

## 递归

> 函数自己调用自己

例：判断回文字符串

```javascript
function isPalindrome(word) {
  if (word.length <= 1) return true
  if (word.first_char != word.last_char) return false
  w = word.remove_first_and_last_char
  return isPalindrome(w)
}
```

递归与迭代：递归算法通常更短，但由于其执行时大量调用自身，从而引入计算开销。

```text
迭代：
START => 1 => 2 => 3 => END

递归：
START => 1 => 2 => 3
                   ↓
  END <= 1 <= 2 <= 3
```