---
title: 在线课程：计算机组成原理
date: 2021-07-11 11:49:12
tags:
  - 在线课程
  - 计算机组成原理
  - 计算机
categories:
  - [计算机, 计算机组成原理]
mathjax: true
---

[Computation Structures (6.004) From MIT OCW](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-004-computation-structures-spring-2017/index.htm)

<!-- more -->

# 信息的基础 Basic Of Information

## 何为信息 Information

> 信息可以 **消除不确定性**
>
> Data communicated or received that **resolves uncertainty** about a particular fact or circumstances.

比如，我们从 52 张扑克牌中选择一张，如果没有“信息”，那么就有 52 种可能性；但如果告诉你这是一张红桃，那么就只剩下 13 种可能性了——这就是信息。

## 如何量化信息 Quantifying Information

对于离散的随机变量 $X$

- 有 $N$ 个可能的值：$x_1$, $x_2$, ..., $x_n$
- 分别对应可能性：$p_1$, $p_2$, ..., $p_n$

那么 $x_i$ 携带的信息量（如果将该信息用 0/1 来进行编码，需要的位数）就是：
$$
I(x_i)=log_2(\frac 1{p_i})
$$
其中 $\frac 1{p_i}$ 衡量的是 $x_i$ 的不确定度。
$$
I(红桃)=log_2(\frac 1{\frac{13}{52}})=2 bits
$$
对于 N 选 M 类型的问题，可以写作：
$$
I(x_i)=log_2(\frac 1{M\cdot(\frac 1N)})=log_2(\frac NM)bits
$$
有时候会得到 $I$ 包含小数点，比如 5.17，这表明为了编码一条信息，我们需要 6 bits，但如果编码 100 条，我们可能不需要 600 bits，而是 520 bits。

> 由上述公式易得，如果某条信息对应的可能性越小，那么他的信息量就越多，就需要越多的 bits 来进行编码。

## 信息熵 Entropy

> 信息熵 $H(X)$ 是每条信息的信息量的平均数

$$
H(x)=E(I(x))=\sum_{i=1}^N p_i \cdot log_2(\frac 1{p_i})
$$

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-1.png)

$H(X)$ 是我们对一段数据进行编码时，所需要的能完整表示其所携带的信息的最少 bits

## 编码 Encoding

> 编码是数据与二进制数之间的 **清晰（unambiguous）** 映射

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-2.png)

一个二进制树：

1. 所有的被编码的符号都位于该二进制树叶节点上
2. 从根节点到该符号位于的叶节点的路径，即为编码

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-3.png)

### 定长编码 Fixed-length Encoding

1. 所有选择的概率是相同的
2. 在二进制树中，每个符号到根节点的长度是相同的
3. 可以很方便地在一段信息的编码中找到指定的位置
4. 通常会比较低效

$$
H(x)=\sum_{i=1}^N \frac 1N \cdot log_2(\frac 1{\frac 1N})=log_2(N)
$$

例如：

1. 4-bit 的十进制数的二进制编码：$log_2(10)=3.322$
2. 7-bit 的 ASCII：$log_2(94)=6.555$

#### 编码有符号的数字 Encoding Signed Integers

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-4.png)

这个编码方式有两个问题：

1. 出现了 +0 和 -0，这是无意义的
2. 除了加法运算之外，需要额外实现一套减法系统

#### 二进制补码 Two's Complement Encoding

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-5.png)

二进制补码就解决了上述两个问题。在二进制补码中，最高位 **不是符号位**，而是拥有一个 **负的权重**。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-6.png)

- 通过二进制补码，我们可以额观察到， -1 和 1 相加为 0
- 同时我们可以将 B-A 变为 B+(-A)，并且通过上述计算找出了 -A 的表达形式，即 (-1-A) + 1；-1 用二进制表示全部为 1，因此 (-1-A) 其实就是 **按位取反**。

### 变长编码 Variable-length Encoding

目标：将出现概率大的字符用尽可能短的编码来表示，将出现概率小的字符用长的编码来表示。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-7.png)

#### 霍夫曼算法 Huffman's Algorithm 

1. 自底向上地考虑，先构建一颗二进制子树，他们的叶节点是概率最小的两个符号
2. 这棵子树的根节点的概率是这两个叶节点之和，然后在此基础上，使用概率第三小的符号添加新的子树
3. 以此类推，直到所有的符号都用完

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-8.png)

为了找到更有效的方法，我们可以进一步地考察组合的概率：

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-9.png)

## 错误校验 Error Detection and Correction

### 汉明距离 Hamming Distance

> 两个字符串对应位置不同的字符的个数。即将一个字符串变成另一个字符串所需要替换的字符个数

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-10.png)

### 单点错误 Single-bit Error

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-11.png)

对于只有一位的编码，汉明距离是 1。发生单点错误后的，我们无法知道到底有没有发生错误，因此需要添加一个奇偶校验位（parity bit），提高他的最小汉明距离，这样就能检测出是否有单点错误了。

### 多点错误 Multi-bit Error

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-12.png)

同样的，如果我们将汉明距离提升到 3，我们可以检测出两位的错误。为了检测出 $E$ 位的错误，我们需要将汉明距离提升至 $E + 1$

### 错误校正 Error Correction

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-13.png)

以单点错误为例，如果汉明距离为 2，我们无法校正这个错误，我们只知道他错了，但是如果我们将汉明距离提升至 3，我们就能纠正这个错误的位。

同理，为了校正 $E$ 位的错误，我们需要将汉明距离提升至 $2E + 1$

# 数字的抽象 The Digital Abstraction

>  可以用电磁现象来传输数据，比如电压、相位、电流、频率

我们可以用每个点的电压来保存一张黑白相片的信息，比如 0V 表示黑色、1V 表示白色、0.37V 表示 37% 灰阶。但这样的方式会使得数据容易在原器件的处理过程中损耗，因此在处理时需要进行数字抽象，大于一定电压为 1、小于一定电压为 0。

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-14.png)

并且，元件的输出需要比输入更加严格，以留出足够多的噪声余量。

## 电压转换特性 Voltage Transfer Characteristic

![](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/Course-ComputationStructure-15.png)

如图所示是一个缓冲器的 VTC，对于任意输入的比 $V_{IL}$ 小的值，需要得到一个比 $V_{OL}$ 小的值（因此大于 $V_{OL}$ 的值都是不允许的）；对于输入任意比 $V_{IH}$ 大的值，需要得到比 $V_{OH}$ 大的值（因此小于 $V_{OH}$ 的值都是不允许的），这样就得到了阴影部分的禁止区域。

注意：

1. 在 $V_{IL} < V_{IN} < V_{IH}$ 的区间内，VTC 可以是任意形状的
2. 中间的长方形一定是瘦长的，因为 $V_{OH}-V_{OL}>V_{IH}-V_{IL}$。因此 VTC 的斜率一定有某处是大于 1 的，并且整条 VTC 是非线性的

