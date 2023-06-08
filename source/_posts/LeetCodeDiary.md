---
title: LeetCodeDiray
date: 2023-02-05 15:49:12
categories:
  - [计算机]
mathjax: true
---

与 {% post_link Algorithm %} 不同，算法与数据结构一文中主要是从理论、宏观、模板的角度体系化地记录常见的算法与数据结构。而本文是做算法题中的学习记录，主要记录主观的心路历程和一些实际做题中的坑点、想法和解法。

本文将按照一定的 Tag，比如二分查找、双指针等来进行展开。

<!-- more -->

# 二分查找

## LeetCode 704. BinarySearch

> [LeetCode 704. 二分查找](https://leetcode.cn/problems/binary-search/)
>
> Last Reviewed: 2023.6.8

该题主要需要注意熟悉 **左闭右开** 和 **左闭右闭** 的两种写法，具体写法见 {% post_link Algorithm %} 对应章节。

## LeetCode 35. SearchInsertPosition

> [LeetCode 35. 搜索插入位置](https://leetcode.cn/problems/search-insert-position/)
>
> Last Reviewed: 2023.6.8

该题与二分查找别无二致，返回值既可以是右指针 `r`，也可以是左指针 `l`，比如我们如果使用左闭右闭区间 `[l, r]` 的写法：

```javascript
while (l <= r) {
  if (num[mid] === target) {
  	return mid
  } else if (nums[mid] < target) {
    l = mid + 1
  } else {
    r = mid - 1
  }
}
return l
```

可以看到三种情况下返回值均可以为 `l`：

- 当 `target < nums[0]`，我们要将 `target` 插入第 `0` 位，此时 `l` 也将永远为 `0`。
- 当 `target` 在 `nums` 范围内时，比如 `[0, 2]` 中插入 `1`，应该返回 `1`，也就是大于 `1` 的第一个数字的 `index`；由于 `l` 最后加了一，所以正好是这一位。
- 当 `target > nums[nums.length - 1]` 时，我们要将 `target` 第 `nums.length` 位，同上，也正好可以用 `l` 来表示。

## LeetCode 34. FindFirstAndLastPositionOfElementInSortedArray

> [LeetCode 34. 在排序数组中查找元素的第一个和最后一个位置](https://leetcode.cn/problems/find-first-and-last-position-of-element-in-sorted-array/)
>
> Last Reviewed: 2023.6.8

该题刚拿到的时候毫无头绪，想着可能是二分查找嵌套。看了题解之后其实也是有点懵，题解将整体拆成三步：

1. 查找区间的右边界（约定右边界也不包含 target）
2. 查找区间的左边界（约定左边界不包含 target）
3. 根据左右边界确定最终 target 的 position
   1. target 不在 nums 的范围内：表现为该区间的右边界不在 nums 的范围内（target 位于 nums 的左侧），或该区间的左边界不在 nums 的范围内（target 位于 nums 的右侧）。
   2. target 在 nums 的范围内，但 nums 中不存在 target：表现为左右区间中间的元素数量小于 1。
   3. target 在 nums 的范围内，且 nums 中存在 target：输出结果即可。

## LeetCode 69. SqrtX

> [LeetCode 69. X 的平方根](https://leetcode.cn/problems/sqrtx/)
>
> Last Reviewed: 2023.6.8

因为要找到的结果是整数，因此直接在整数范围用二分查找即可。

## LeetCode 367. ValidPerfectSquare

> [LeetCode 367. 有效的完全平方数](https://leetcode.cn/problems/valid-perfect-square/)
>
> Last Reviewed: 2023.6.8

与 69 题相同。

# 双指针

## LeetCode 27. RemoveElement

> [LeetCode 27. 移除元素](https://leetcode.cn/problems/remove-element/)
>
> Last Reviewed: 2023.6.8

一眼快慢指针。

## LeetCode 283. MoveZeros

> [LeetCode 283. 移动零](https://leetcode.cn/problems/move-zeroes/)
>
> Last Reviewd: 2023.6.8

快慢指针，需要想清楚慢指针的定义，我的解法是慢指针指向最后一个非 0 的后一位。

## LeetCode 844. BackspaceStringCompare

> [LeetCode 844. 比较含退格的字符串](https://leetcode.cn/problems/backspace-string-compare/)
>
> Last Reviewed: 2023.6.8

刚拿到这道题的时候只能想到用栈来模拟退格的过程，创建两个新的结果字符串来进行比较，想不到怎样将使用双指针来解。

题解确实巧妙，将字符串反向遍历，将 # 看做是 Skip 而不是 Back。

## LeetCode 977. SquaresOfASortedArray

> [LeetCode 977. 有序数组的平方](https://leetcode.cn/problems/squares-of-a-sorted-array/)
>
> Last Reviewed: 2023.6.8

刚拿到这道题的时候，想的是找到第一个非负数，然后用两个指针向两头移动，逐个按平方大小的顺序推入结果数组。

题解也确实巧妙，不是从中间向两边发散，而是从两头向中间走，相当于反向生成结果，与 844 有异曲同工之妙。

## LeetCode 209. MinimunSizeSubarraySum

> [LeetCode 209. 长度最小的子数组](https://leetcode.cn/problems/minimum-size-subarray-sum/)
>
> Last Reviewed: 2023.6.8

这道题也是刚拿到的时候毫无头绪，结果发现是题目看得有问题，他要求的子数组是连续的而不是随便选，这样自然就可以用双指针（滑动窗口）来做了。

## LeetCode 904. FruitIntoBaskets

> [LeetCode 904. 水果成篮](https://leetcode.cn/problems/fruit-into-baskets/)
>
> Last Reviewed: 2023.6.8

该题用滑动窗口，但有几个坑点：

1. 最开始是用 Set 存的已经采了哪几种水果，但后来发现需要用 Map 来将每种水果的数量存下来，这样才能在滑动窗口滑走之后将前面所有的水果删掉；
2. 最开始以为是按照入水果篮的顺序依次删除水果，但其实是需要在有新的品种进入之后，保留新品种的前一个水果的品种（即使该品种可能是先入库的）。

## LeetCode 59. SpiralMatrixII

> [LeetCode 59. SpiralMatrixII](螺旋矩阵 II)
>
> Last Reviewed: 2023.6.8

该题题解是可以以一圈为一个大循环，在每个大循环内确定该次大循环的四个小循环的边界，然后填充矩阵。

我自己考虑的解法是使用一个 `next` 变量维护下一次应该朝哪个方向走，比如 `right` `down` 等，然后走到边界之后再换向，模拟人思考的过程。
