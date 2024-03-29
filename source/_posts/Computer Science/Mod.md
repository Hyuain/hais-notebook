---
title: Mod
date: 2022-02-05 15:49:12
categories:
  - - 计算机
mathjax: true
tags:
  - CX
---

# Mod

最简单的模运算：计算乘积的个位数，比如 $1234\cdot6789$ 的个位数是 $(4\cdot9)\%10=6$ ，$1234+6789$ 的个位数是 $(4+9)\%10=3$。

将上述结论抽象成数学公式就是：
$$
(a+b) \% m = ((a \% m) + (b \% m)) \% m
$$

$$
(a \cdot b) \% m = ((a \% m) \cdot (b \% m)) \% m
$$

# Mod 1e9 + 7

由于有的语言对大数并没有进行特殊处理，所以无法进行高精度的大数运算，有的题目便会要求将结果对 1e9 + 7 取模，这样就不用考虑大数运算的问题了。

但是，**不能只是单纯地对结果取模**，因为这样就没有意义了，并不能避免大数运算带来的问题。

此外，还有几个注意点：

1. 取模之后就不能用 Max 比较最大值了，因此有的题就不能用动态规划了（但是仍然可以使用 Sum）

2. 注意是否真的取模了：
   ```js
   // 用 += 的时候注意
   for(int i = 1; i <= n ; i ++) {
       res += i % (1e9 + 7); // 实际上并没有对 res 取模，只是取模了 i
   }
   
   // 由于 * 和 % 是从左到右运算的，因此需要加上括号
   res = res % mod + pre[i][j] % mod * a[i][j] % mod;
   ```

