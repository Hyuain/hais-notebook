---
title: Rust
date: 2023-07-18 14:38:01
categories:
  - [计算机]
---

熟练掌握三门语言计划之三：Rust。

资料来源：

- [The Book](https://doc.rust-lang.org/)

<!-- more -->

# Concepts

## 变量与可变性

Rust 中可以使用 `let` 来定义变量，但定义的 **变量默认是不可变（Immutable）** 的，重复给变量赋值会在编译时报错：

```rust
let x = 5;
x = 6;       // error
```

要想让变量变得可变，需要加上 `mut`：

```rust
let mut x = 5;
x = 6;       // ok 
```

此外还可以用 `const` 定义常量：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量有这样几个特点：

1. 永远不可以改变
2. 需要定义数据类型
3. 在程序运行的所有生命周期内可在作用域内访问

Rust 还有一个特性叫 **Shadowing**，这个特性允许变量被重复定义（在 JavaScript 中，只有在新的作用域里面才能重新定义同名变量）：

```rust
let x = 5;
let x = 6;       // ok
{
  let x = x * 2; // ok
}
```

Shadowing 允许两次定义的变量类型不同，因为他们实际上是两个变量，而 `mut` 则不允许两次赋值为不一样的类型：

```rust
let spaces = "    ";
let spaces = spaces.len();    // ok
```

```rust
let mut spaces = "    ";
spaces = spaces.len();        // error
```

## 数据类型

Rust 中的数据类型分为 **标量类型（Scalar Types）** 和 **合成类型（Compond Types）**。

**标量类型** 包括：

- **整型（Integer）**：包括 **有符号整型（`i` 开头）**和 **无符号整型（`u` 开头）**，比如 `i8` `u128` 等，数字表示他占了多少位。其中 `isize` 和 `usize` 类型取决于程序运行所处的计算机的体系结构，比如 64 位或 32 位等。
- **浮点型（Floating-Point）**：包括 `f32` 和 `f64`。
- **布尔型（Boolean）**：`bool`，可以用 `true` 或 `false` 显式指定。
- **字符型（Character）**：`char`，与字符串字面量不同，用单引号 `' '` 指定。

**合成类型** 包括：

- **元组（Tuple）**：用括号框起来的、逗号隔开的一组值，比如 `let tup: (i32, f64, u8) = (500, 6.4, 1)`，可以用 `tup.0` `tup.1` 等来取到里面的值。**元组中的值可以是不同类型的。**
- **数组（Array）**：**数组中的值类型必须相同，且 Rust 中的数组是定长的，且不能越界访问。** 可以这样定义数组类型 `let a: [i32; 5] = [1, 2, 3, 4, 5];` 也可以在最开始为所有元素赋相同的值 `let a = [3; 5];`。
- 

