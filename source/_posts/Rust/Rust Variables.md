---
title: Rust Variables
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - "#RSX"
---

# Variable Declaration

## Specify Types

Rust is a strong typed language, types can be specified explicitly.

```rust
let name = "Harvey";
let age = 26;                   // ok
let amount = 13212435446;       // error, its default by i32
let amount: i64 = 13212435446;   // ok
```

## Names

Names can contain letters, numbers and underscores, and **must start with a letter or underscore**.

Follow snake_case naming convention.

## Constant

此外还可以用 `const` 定义常量，习惯性上用大写：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量有这样几个特点：

1. 永远不可以改变
2. 需要定义数据类型
3. 在程序运行的所有生命周期内可在作用域内访问
4. 不可以使用 Shadowing

#  Immutibility

Variables defined are default as **Immutable**,  reassigning to variables shall cause compile time error:

```rust
let x = 5;
x = 6;       // error
```

To make it mutable, add `mut`:

```rust
let mut x = 5;
x = 6;       // ok 
```

# Shadowing

Rust has a feature called **Shadowing**, allowing variables to be declared again. (In JavaScript, variables with same name can not be redeclare in one scope)

```rust
let x = 5;
let x = 6;       // ok
{
  let x = x * 2; // ok
}
```

Shadowing allows the vairable be redeclared with a different type, because they are actually two variables.

While a variable can not be reassigned to a different type using `mut`.

```rust
let spaces = "    ";
let spaces = spaces.len();    // ok
```

```rust
let mut spaces = "    ";
spaces = spaces.len();        // error
```

# Destructuring

Rust allow destructuring of tuples. Multiple variables can be decalred simultaneously: 

```rust
let (a, b) = (1, 2);
let (mut x, y) = (1, 2);
```

Additional ways:

```rust
let (x, y);
(x,..) = (3, 4);
[.., y] = [1, 2];
```

# Unused Variables

如果定义了一个变量但没有使用，Rust 编译器会报 Warning：

```rust
fn main() {
    let x = 1;
}

// Warning: unused variable: `x`
```

有两种方式可以解决，一种是在变量前面加下划线：

```rust
fn main() {
    let _x = 1;
}
```

另一种是在函数上面加 `#[allow(unused_varibales)]`：

```rust
#[allow(unused_varibales)]
fn main() {
    let x = 1;
}
```
