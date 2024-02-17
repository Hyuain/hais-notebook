---
title: Rust Ownership & Moves
date: 2024-01-25 20:15:01
categories:
  - - 计算机
tags:
  - RSX
---

所有权（Ownership）是 Rust 内存管理中的核心概念，Rust 通过在编译器进行复杂检查来避免运行时的内存错误，并且避免了使用垃圾回收器，从而得到了更好的性能表现。

- 每个值都有 **唯一拥有者（A Single Owner）**，拥有者被丢弃时，其值也会被一并丢弃；
- 可以将值从一个拥有者转移给另一个拥有者，这被称为 **移动（Move）**，这允许开发者构建、重排和拆除树形所有权结构；
- **`Copy` 类型** 是一类非常简单的类型，比如整数、浮点数等，他们不受所有权规则的约束；
- 标准库也提供了 `Rc` 和 `Arc` 这两个 **引用计数指针类型（Reference-Counted Pointer Types）**，允许有多个拥有者；
- 通过对值的 **借用（Borrow）**，可以暂时获得值的引用，这种引用是 **非拥有型指针（Non-Owning Pointer）**，他们的丢弃不会使得值一并被丢弃。

<!-- more -->

# Ownership

C++ 中，开发者使用口头约定来规定所有权，而 Rust 则将所有权的概念内置于语言本身，并且会在编译器进行强制检查。

**每个值都有决定其生命周期的唯一拥有者（Owner）。** 当拥有者被释放时，他拥有的值也会被释放，这个释放行为在 Rust 中被称为 **丢弃（Drop）**。

当控制流离开声明变量的块时，这个变量会被丢弃，因此他的值也会被丢弃：

```rust
fn print_padovan() {
    let mut padovan = vec![1,1,1];   // allocated here
    for i in 3..10 {
        let next = padovan[i-3] + padovan[i-2];
        padovan.push(next);
    }
    println("P(1..10) = {:?}", padovan)
}                                    // dropped here 
```

![Vec<i32>](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/image-20240125203121920.png)

变量 `padovan` 的类型为 `Vec<i32>`，`print_padovan` 的栈帧上存储着 **指向缓冲区的指针、缓冲区的容量、已使用的长度**。而向量的缓冲区则被分配在堆上。

当 `padovan` 超出作用域时就会被丢弃。**因为向量拥有他的缓冲区，缓冲区也会一起被丢弃。**

`Box` 也是类似的，当程序使用 `Box::new(v)` 在堆上分配一些内存后，会得到一个指向该堆空间 `Box`，当这个 `Box` 被丢弃时，指向的堆空间也会被丢弃。

此外，数组、向量、元组拥有他们自己的元素，结构体（Struct）拥有他们自己的字段，他们最后可能会形成一个树形结构，比如对于下面代码：

```rust
struct Person { name: String, birth: i32 }

let mut composers = Vec::new();
composers.push(Person { name: "Palestrina".to_string, birth: 1525 });
composers.push(Person { name: "Dowland".to_string, birth: 1563 });
composers.push(Person { name: "Lully".to_string, birth: 1632 });

for composer in composers {
    println!("{}, born {}", composer.name, composer.birth);
}
```

![A tree of ownership](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/image-20240125204723063.png)

`composers` 拥有他里面的三个元素，而每个元素又拥有他的字段们。当控制流离开 `composers` 的作用域时，整棵所有权树都会被一起丢弃。

# Moves

