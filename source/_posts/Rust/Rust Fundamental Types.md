---
title: Rust Fundamental Types
date: 2023-07-18 14:38:01
categories:
  - 计算机
tags:
  - RSX
---

本章包括了 Rust 的基本类型（Fundamental Types），这些源代码级别的类型都有对应的机器级表示，具有可预测的成本和性能。

<!-- more -->

# Numeric Types

Rust 中的数值类型是定宽的（fixed-width），他是 Rust 类型系统的根基。

数字类型可以加分隔符来方便阅读，比如 `1_000_000`。

Rust 中几乎不会发生隐式类型转换，也就是说若参数需要 `u16` 类型，传给他 `u8` 也是错误的，需要通过 `as u16` 进行显式类型转换。

## Integer Types

> 包括 **有符号整型（`i` 开头）** 和 **无符号整型（`u` 开头）**，比如 `i8` `u128` 等，数字表示他占了多少位。其中 `isize` 和 `usize` 类型取决于程序运行所处的计算机的体系结构，比如 64 位或 32 位等。

**整型与字节值（Byte Values）：** Rust 使用 `u8` 作为字节值，比如从二进制文件或套接字中读取数据时会产生一个 `u8` 值构成的流。

**整型与字符（Characters）：** Rust 中数值与字符是两种不同的类型，`char` 就是 `char`，他不是 `u8` 也不是 `u32`。

**整型字面量（Integer Literals）：** Rust 的整型字面量可以带上一个后缀来表示他的类型，比如 `42u8` `1729isize` 等。可以使用 `0x` `0o` `0b` 前缀来表示十六进制、八进制、二进制字面量。

> 如果某个字面量没有写后缀，那么 Rust 会延迟确定其类型，如果最后有多个候选类型，且 `i32` 是也在候选中，那么 Rust 就会 **默认使用 `i32`**。如果无法确定类型，那么 Rust 会因为歧义而报错。

**字节字面量（Byte Literals）：** Rust 为 `u8` 值提供了一种类字符的字面量，比如 `b'X'` 用来表示字符 `X` 的 ASCII 码对应的 `u8` 值，他与 `65u8` 完全等效。对于难于书写或阅读的字符，可以将编码改成十六进制，类似于 `b\xHH` 的写法，比如用 `b\x1b` 表示字符 `escape` 的字节字面量（因为 `escape` 的 ASCII 码为 27，即十六进制的 1B）。通常我们只有当需要明确表示这个表示这个 `u8` 表示的是某个字符的时候，才会使用字节字面量，而不是简单的 `27u8`。

**整型的方法：** Rust 提供了诸如 `abs()` `pow()` 等方法，但是使用这些方法的时候需要明确表示其类型，比如 `(-4_i32).abs()` 或 `i32::abs(-4)`，尽管所有的有符号整型都有 `abs()` 方法。

> 需要注意方法调用的优先级高于一元前缀运算符，因此为了计算 -4 的绝对值，需要写作 `(-4_i32).abs()`，而不能省略括号。

**溢出：** 当整型运算溢出时，Rust 在调试构建时会 panic，而在发布构建中，会默认使用 **回绕（Wrapping）运算。**

- **检查运算（Checked Arithmetic）：** 比如 `checked_add()` 方法等，会返回结果的 `Option` 值，如果数学上有效就是 `Some(v)`，否则为 `None`；
- **回绕运算（Wrapping Arithmetic）：** 比如 `wrapping_mul()` 方法等，会返回“数学意义上正确的值”对“值类型范围”取模的结果；
- **饱和运算（Saturating Arithmetic）：** 比如 `satruating_sub()` 方法等，会返回最接近数学意义上正确值的结果；
- **溢出运算（Overflowing Arithmetic）：** 比如 `overflowing_add()` 方法等，会返回一个 `(result, overflowed)` 元祖，包含回绕版本的内容，和是否发生了溢出的布尔值。

## Floating-Point Types

> 包括 `f32` 和 `f64` 共两种。

同样，如果浮点字面量缺少后缀，且 Rust 发现他两种浮点类型都符合，那么会默认使用 `f64`。

# Booleans

> Rust 的布尔类型 `bool` 包括 `true` 和 `false` 两种。

需要注意的是，在 Rust 中，`if` `while` 这类的控制结构的条件必须是 `bool` 表达式，不能是 `0`、空字符串这种值，短路逻辑运算符 `&&` 和 `||` 也是如此。

Rust 中的 `bool` 可以用 `as` 运算符转换为整型的 `1` 或 `0`，但不能反向转换。

# Characters

> Rust 的字符类型 `char` 以 32 位（4 字节）值表示单个 Unicode 字符，无论是中文还是英文字符，可以用 `size_of_val()` 函数看到。

Rust 不会在 `char` 和其他类型之间进行隐式的类型转换，可以使用 `as` 将 `char` 显示转换为整型，在转换为小于 32 位的数字类型时，高位会被截断。

# Tuples

> 最多 12 个元素组成的一个定长序列。

```rust
let person = ("John", 27, true);
let person: (&str, i32, bool) = ("John", 27, true);
println!("{:?}", person.0);     // "John"
```

可以用模式匹配语法使用元组中的值：

```rust
let text = "hello world";
let (head, tail) = text.split_at(3);
```

其中比较特殊的是零元组（Zero-Tuple），`()`，又被称为 **单元类型（Unit Type）**。不返回值的函数返回类型就是 `()`。

# Pointer Types

Rust 中有很多表示内存地址的类型，当我们需要让一些值指向其他值时，必须显式地使用指针，Rust 对他们会有一些安全性约束，来消除未定义行为。

## References

表达式 `&x` 可以生成一个对 `x` 的引用（Reference），这个引用实际上是 **一个机器字**，其中保存了 `x` 的地址。这个地址可能指向堆中，也可能指向栈中。

在 Rust中，我们将使用 `&` 创建引用的过程叫做 **借用了一个对 `x` 的引用**。这实际上牵扯到了 Rust 核心的所有权和生命周期的概念。简单来说，由于是 **借用**，当这个引用超出作用域时，他指向的资源并不会自动释放，因为他并没有资源的所有权。

Rust 引用有两种形式：

- `&T` 表示 **不可变的共享引用**，对单个值可以同时存在多个共享引用。
- `&mut T` 表示 **可变的独占引用**，可以通过该引用读取和修改其指向的值，**只要这个引用还存在，就不能对该值有任何其他类型的引用。**

## Boxes

在堆中分配值的最简单方法就是使用 `Box`：

```rust
let t = (12, "eggs");
let b = Box::new(t);   // b 的类型是 Box<(i32, &str)>
```

调用 `Box::new()` 会在堆上分配足以容纳此元祖的内存。**当 `b` 超出作用域时，内存会被立即释放。**

## Raw Pointers

Rust 中也有裸指针类型 `*mut T` 和 `*const T`，这跟 C++ 中的指针非常像，使用裸指针是不安全的，因为 Rust 并不会跟踪他指向的内容，导致她可能为空、或者指向奇怪的值。

只能在 `unsafe` 块中对裸指针解引用（Dereference）。

# Arrays, Vectors, and Slices

Rust 中有三种类型表示内存中值的序列：

- 类型 `[T; N]` 表示有 `N` 个值，类型为 `T` 的 **数组**，他长度和类型在运行时不能改变；
- 类型 `Vec<T>` 称为 `T` 的 **向量**，他是动态分配的、长度可变的类型为 `T` 的值的序列；
- 类型 `&[T]` 和 `&mut [T]` 分别称为 `T` 的 **共享切片** 和 **可变切片**，他们是对一系列元素的引用。

这三种类型都可以用 `v.len()` 得到 `v` 中的元素数，`v[i]` 得到第 `i` 个元素，越界访问会导致 panic，同时 `i` 的类型必须是 `usize`。

## Arrays

> 数组 `[T; N]` 的类型和长度在运行时不能改变，或者说数组的长度是其类型的一部分，并且会在编译期固定下来。

创建数组

```rust
let primes = [2, 3, 5, 7, 11];
let doubles: [f64: 4] = [2.0, 4.0, 6.0, 8.0];
println!("{}", primes);     // error
println!("{:?}", primes);   //ok
```

使用默认值创建数组

```rust
let numbers = [0; 4];        // [0, 0, 0, 0];
const DEFAULT: i32 = 3;
let numbers = [DEFAULT; 4];  // [3, 3, 3, 3];

let buffer = [0u8; 1024];    // 创建 1KB 的缓冲区
```

更新数组的元素

```rust
let mut numbers = [0; 4];
numbers[3] = 2;              // [0, 0, 0, 2];
```

使用迭代器遍历

```rust
for number in numbers.iter() {
    println!("{}", number + 3);
}
```

事实上，数组上的包括遍历、搜索、排序、填充、过滤等方法都是作为切片的方法，而非数组的方法提供的。但是 Rust 在搜索各种方法的时候会隐式地将对数组的引用转换为切片，因此可以直接在数组上调用各种切片方法。

## Vectors

> 向量 `Vec<T>` 是一个可调整大小的 `T` 类型元素的数组，是在堆上分配的。
> 
> 向量并没有内置在语言中，他由 `std` 标准库提供。或者说所有在堆上分配内存的类型都没有包含在语言中。

创建向量

```rust
let primes: Vec<i32> = Vec::new();
let primes = vec![2, 3, 5];
println!("{:?}", primes);      // [2, 3, 5]
// 可以使用 collect 从迭代器创建向量，但是需要指定类型，因为他还可以构建出除了向量以外的集合
let v: Vec<i32> = (0..5).collect();
```

使用默认值创建向量

```rust
let numbers = vec![2; 4];       // [2, 2, 2, 2]
const DEFAULT: bool = true;
let values = vec![DEFAULT; 4];  // [true, true, true, true]
```

添加、删除或插入元素（将会导致所有受影响位置后面的元素移动）

```rust
let mut v: Vec<i32> = Vec::new();
v.push(2);                // [2]
v.push(3);                // [2, 3]
v.insert(1, 5);           // [2, 5, 3]
v.remove(0);              // [5, 3]
v.pop();                  // [5], 返回 Some(3)
```

更新向量中的元素

```rust
let mut numbers = vec![0;4];
numbers[3] = 2;                // [0, 0, 0, 2];
```

使用迭代器遍历

```rust
for number in numbers.iter() {
    println!("{}", number);
}
```

与数组不同，向量主要是在堆上分配内存的，`Vec<T>` 由三个部分组成：

1. 指向元素在堆中分配的缓冲区（该缓冲区由 `Vec<T>` 创建并拥有）的指针；
2. 缓冲区能够存储的元素数量，可以用 `capacity` 方法得到；
3. 现在实际拥有的数量，可以用 `len` 方法得到。

当缓冲区到达最大容量后继续添加元素时，他会：

1. 分配一个更大的缓冲区，将当前的内容复制到其中；
2. 更新向量的指针指向新的缓冲区；
3. 释放旧的缓冲区。

可以用 `Vec::with_capacity()` 创建一个指定缓冲区大小的向量，这样就减少了在添加值的过程中不必要的内存重新分配，当然如果缓冲区满了，他还是会正常进行扩容。

## Slices

 > 切片 `[T]` 是数组或向量中的一个区域。切片可以是任意长度，因此他不能存储在变量中或者作为函数的参数进行传递，切片总是通过引用进行传递。
 > 
 > 也就是说我们一般不直接使用切片，而是使用 **对切片的引用**。因此很多时候我们说的“切片”，实际上指的是对“切片的引用”，这个术语经常混用，需要注意。

 对切片的引用是一个 **胖指针（Fat Pointer）**，他是两个字长的值，包括：

1. 指向切片第一个元素的指针；
2. 切片中元素的数量。

比如数组切片和向量切片的例子：

```rust
let v: Vec<f64> = vec![0.0,  0.707,  1.0,  0.707];
let a: [f64; 4] =     [0.0, -0.707, -1.0, -0.707];

// 下面两行代码自动把 &Vec<f64> 和 &[f64; 4] 转换为了对切片的引用 
let sv: &[f64] = &v;
let sa: &[f64] = &a;
```

![切片 sv 和 sa](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/image-20240123191930996.png)

普通的引用（Reference），比如 `&i32` 等，是指向 **单个值** 的 **非拥有型指针（Non-Owning Pointer）**；而对切片的引用（Reference to a Slice）是指向内存中的 **一系列连续值** 的 **非拥有型指针**，可以看到他额外用了一个值表示切片的长度。

事实上很多属于向量或数组的方法实际上是在切片上定义的，比如 `sort` `reverse` 等，都是定义在切片类型 `[T]` 上的方法。

可以使用 Range 作为索引来获取到对切片的引用，该引用既可以指向一个数组或向量，也可以指向一个既有的切片：

```rust
print(&v[0..2]);
print(&a[2..]);
print(&sv[1..3]);
```

# Strings

**字符串字面量（String Literals）**

Rust 中字符串字面量需要用双引号 `""` 括起来，可以使用 `\` 对特殊字符进行转义。注意不能使用单引号，单引号表示的是字符类型 `char` 字面量。

Rust 中字符串里面的换行符会被记录，但是如果某一行是以 `\` 结尾，那么就会丢弃其后的换行符。

如果不想每个都加一个反斜杠，可以使用 **原始字符串（Raw String）**，原始字符串不识别任何转译序列：

```rust
let path = r"C:\Program Files\Gorillas";
let pattern = Regex::new(r"\d+(\.\d+)*");
```

如果想在原始字符串中使用双引号，那么可以在原始字符串开头和结尾添加 `#`，`#` 的数量可以随意，但是前后的 `#` 数量要一致：

```rust
println!(r###"
 This raw string started with 'r###"'.
 Therefore it does not end until we reach a quote mark ('"')
 followed immediately by three pound signs ('###'):
"###);
```

**字节串（Byte Strings）**

带有 `b` 前缀的字符串都是字节串，他们可以看做 `u8` 的切片：

```rust
let method = b"GET";  // 类型是 &[u8; 3]
assert_eq!(method, &[b'G', b'E', b'T']);
```

**`String` 与 `&str`**

**Rust 字符串不以是 `char` 数组的形式存储在内存中的，而是以 [[Charset and Encoding|UTF-8 编码]] 的形式存储的。** 因此字符串中的每个字符并不是像 `char` 类型一样统一占用 4 字节，而是占用一个（ASCII 字符）或多个（其他字符）字节。

下面的例子展示了 `String`、`String` 切片、`str` 和 `&str` 在内存中是怎么存放的：

```rust
let noodles = "noodles".to_string();
let oodles = &noodles[1..];
let poodles = "ಠ_ಠ";
```

![String, &str 与 str](https://hais-note-pics-1301462215.cos.ap-chengdu.myqcloud.com/image-20240124211147052.png)

`String` 有一个可以调整大小的缓冲区，**可以将 `String` 视为 `Vec<u8>`**，但他同时还保证了其是格式良好的 UTF-8。

**`&str` 同其他切片类型一样，也是一个胖指针，可以认为 `&str` 是 `&[u8]`**，但他能保证其是格式良好的 UTF-8.

**字符串字面量就是 `&str`**，它通常与程序的机器码一起存储在只读内存区。

`String` 或 `&str` 的 `len()` 方法会返回他的长度，**这个长度是以字节而不是以字符为单位的**：

```rust
assert_eq("ಠ_ಠ".len(), 7);
assert_eq("ಠ_ಠ".chars().len(), 3);
```

`&str` 是不能修改的，`&mut str` 也没什么用，因为对 UTF-8 的几乎所有操作都会更改总长度，但切片不能重新分配其引用目标的缓冲区，一般最多用 `make_ascii_uppercase` 和 `make_ascii_lowercase` 两个方法。

**`&str` 可以引用任意字符串的任意切片，无论是存储在可执行文件中的字符串字面量，还是在运行时分配的 `String`。因此，通常将 `&str` 作为函数参数类型。**

>  字符串支持 `==` 和 `!=` 运算符，他会按照顺序比较字符串，不管他们是否存储在内存的相同位置。

