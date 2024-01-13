---
title: Rust Closures
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

> A function within a function

An anonymous function, lambda expression:

```rust
// {} can be omitted, if no return type and only one expression
|a: i32, b: i32| println!("{}", a + b);
|a: i32, b: i32| -> i32 { a + b };
|a: i32, b: i32| { a + b };
```

A function can be assigned to a variable:

```rust
let sum = |a: i32, b: i32| -> i32 {a + b};
sum(2, 3);
```

Closure can be generic:

```rust
let gen = |x| { println!(x) };
gen(3);        // 3
gen(true);     // error, because gen has already be called with integer
```
