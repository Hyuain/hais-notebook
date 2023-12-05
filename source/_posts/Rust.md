---
title: Rust
date: 2023-07-18 14:38:01
categories:
  - [计算机]
---

熟练掌握三门语言计划之三：Rust。

资料来源：

- [The Book](https://doc.rust-lang.org/)
- [Learn Rust Programming - Complete Course](https://www.youtube.com/watch?v=BpPEoZW5IiY)

<!-- more -->

# Project Basics

## Cargo

Cargo is the project manager for Rust.

```bash
cargo new
cargo build
cargo run
cargo clean
cargo check
cargo doc
```

## User Input

Rust is not typically designed for user interaction, and the way it handle user input is a little bit complex.

```rust
use std::io;

fn main() {
    let mut input = String::new();
    println!("Say something");
    match io::stdin().read_line(&mut input) {
        Ok(_) => {
            println!("You said {}", input);
        },
        Err(e) => {
            println!("Something went wrong {}", e);
        }
    }
}
```

## Comments

```rust
// line coment
/*
  multiline coment
*/
/// doc comments, used to document functionality
//! doc comments, used to document crates
//! # Heading
//! ```
//!   document code
//! ```
```

Run `cargo doc` to generate documents according to comments.

## Printing Values

```rust
println!("Hello world!");

// Formatting
println!("My name is {} and I am {}", "Harvey", 26);

// Expressions
println!("a + b = {}", 3 + 6);

// Positional Arguments
println!("{0} has a {2} and {0} has a {1}", "Harvey", "cat", "dog");

// Named arguments
println!("{name} {surname}", surname="Z", name="Harvey");

// Printing traits
println!("Binary: {:b}, Hex: {:x}, Octal: {:o}", 5, 5, 5);

// Debug
println!("Array {:?}", [1, 2, 3]);
```

# Language Basics

## Statements & Expression

```rust
let x = 4;
let y = {
    let x_squared = x * x;
    let x_cube = x_squared * x;
    // 注意这里没有分号，这个值将成为这个 {} 表达式的值，否则表达式的值将会是 Unit Type ()
    x_cube + x_squared + x
}
```

我们将 `let y = {...}` 称为一个 **Statement**，他是一个赋值操作，并且不会产生任何值；将等号右边的部分称为一个 **Expression**，因为他最后没有以分号结尾，并且会产生一个值。

## Variables

### Specify Types

Rust is a strong typed language, types can be specified explicitly.

```rust
let name = "Harvey";
let age = 26;                   // ok
let amount = 13212435446;       // error, its default by i32
let amount: i64 = 13212435446;   // ok
```

### Unused Varibales

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

### Names

Names can contain letters, numbers and underscores, and **must start with a letter or underscore**.

Follow snake_case naming convention.

###  Immutibility

Variables defined are default as **Immutable**,  reassigning to varibales shall cause compile time error:

```rust
let x = 5;
x = 6;       // error
```

To make it mutable, add `mut`:

```rust
let mut x = 5;
x = 6;       // ok 
```

### Shadowing

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

### Destructuring

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

## Scalar Data Types

Rust 中的数据类型分为 **标量类型（Scalar Types）** 和 **组合类型（Compond Types）**。**标量类型** 包括：

- **整型（Integer）**：包括 **有符号整型（`i` 开头）**和 **无符号整型（`u` 开头）**，比如 `i8` `u128` 等，数字表示他占了多少位。其中 `isize` 和 `usize` 类型取决于程序运行所处的计算机的体系结构，比如 64 位或 32 位等。**默认为 `i32`。**
- **浮点型（Floating-Point）**：**包括 `f32` 和 `f64`。默认为 `f64`。**数字类型可以加分隔符来方便阅读，比如 `1_000_000`。
- **布尔型（Boolean）**：`bool`，可以用 `true` 或 `false` 显式指定。
- **字符型（Character）**：`char`，与字符串字面量不同，用单引号 `' '` 指定，所有的 `char` 都占 4 Bytes（不管是中文还是英文还是特殊字符），可以用 `size_of_val()` 函数看到。

## String

**String slices**

```rust
let cat: &str = "Fluffy";
let cat: &'static str = "Fluffy";
```

`str` is a scalar type.

**String slices are immutable**, because they actually point to a certain part of memory that does not belong to that particular variable (borrow).

**String objects**

```rust
let dog = String::new();
let dog = String::from("Cat");
let owner = format!("Hi, I'm {} the owner of {}", Harvey, dog)
```

**String methods**

```rust
dog.len();
cat.len();        // works on slices
dog.push(' ');    // works only on Strings, and should be `mut`
dog.push_str("haha");
let new_dog = dog.replace("Cat", "Dog");  // works when dog is slice. but new_dog is String
```

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

## Operators

**Arithmetic Operators:** Increment (`++`) and decrement (`--`) are not supported.

**Relational Operators**

**Logical Operators**

**Bitwise Operators**

## Functions

```rust
fn main() {
    for i in 1..6 {
        say_hi();
    }
}

fn say_hi() {
    printl!("Hello there");
}
```

```rust
fn main() {
    let mut name = "John";
    say_hi(name);
    printl!("{}", name);         // John
}

// Pass by value
fn say_hi(name: &str) {
    printl!("Hello {}", name);   // Hello John
}
```

```rust
fn main() {
    let mut name = "John";
    say_hi(&mut name);
    printl!("{}", name);         // Alex
}

// Pass by reference
fn say_hi(name: &mut &str) {
    printl!("Hello {}", name);   // Hello John
    *name = "Alex";
}
```

```rust
fn main () {
    let mut name = "John";
    let greeting = say_hi(&mut name);
    println!("{}", greeting);      // Hello John
}

fn say_hi(name: &mut &str) -> String {
    let greeting = format!("Hello {}", name);
    return greeting;               
    // "return and ;" can be omitted
    // greeting
}
```

# Modules & Crates

## Modules

```rust
// in player.rs, the file it self become a mod, named "player"

pub fn play_movie(name: &str) {
    println!("Playing movie {}", name);
}

pub fn play_audio(name: &str) {
    println!("Playing audio {}", name);
}
```

```rust
// in main.rs

mod player;

fn main() {
    player::play_movie("snatch.mp4");
  
    clean::perform_clean();
    clean::files::clean_files();
}

mod clean {
    pub fn perform_clean() {
        prinln!("Cleaning HDD");
    }
  
    // nested module
    pub mod files() {
        pub fn clean_files() {
            prinln!("Cleaning files");
        }
    }
}
```

## Crates

> Multiple modules are grouped into a crate.

Two types of crates:

- **Binary Crates:** the main component (entry point) of the crate has a `main` function.
- **Library Crates:** has no entry point.

```rust
// in archive.rs

pub mod arch {
    pub fn arch_file(name: &str) {
        println!("Archiving file {}", name);
    }
}
```

```rust
// in main.rs

// use crate::archive::arch::arch_file;
use crate::archive::arch::arch_file as arc;


mod archive;

fn main() {
    // arch_file("file.txt");
    arc("file.txt");
}
```

Cargo is used to manage crates.

External crates are imported in to the project must be added to the `Cargo.toml` file.

```tom
[dependencies]
rand = "0.8.5"
```

```rust
// use rand::Rng;
use rand::{Rng,};

fn main() {
    let mut rng = rand::thread_rng();
    let a: i32 = rng.gen();
    println!("{}", a);
}
```

## Random Generator

```rust
use rand::Rng;
let mut rng = rand::thread_rng();
// generate an integer
rng.gen();
// bounded generation
rng.gen_range(0..10);
rng.gen_range(0.0..10.0);
```

```rust
use rand::{Rng, thread_rng};
use rand::distributions::Alphanumeric;

fn main() {
    let rand_string: String = thread_rng()
        .sample_iter(&Alphanumeric)
        .take(30)
        .map(char::from)
        .collect();
    println!("Gen string: {}", rand_string);
}

```

# Data Types

## Arrays

> A collection of values of the same type

**In Rust, you cannot have different types of data inside an array.**

- **Static:** cannot be resized
- Element values can be modified but **not deleted**
- Indexed

Create array:

```rust
let primes = [2, 3, 5, 7, 11];
let doubles: [f64: 4] = [2.0, 4.0, 6.0, 8.0];
println!("{}", primes);     // error
println!("{:?}", primes);   //ok
```

Create array with default values:

```rust
let numbers = [0; 4];        // [0, 0, 0, 0];
const DEFAULT: i32 = 3;
let numbers = [DEFAULT;4];  // [3, 3, 3, 3];
```

Updating elements:

```rust
let mut numbers = [0;4];
numbers[3] = 2;              // [0, 0, 0, 2];
```



Using an iterator:

```rust
for number in numbers.iter() {
    println!("{}", number + 3);
}
```

## Vectors

> Arrays of variable size

Unlike array, vectors are structures, rather than types.

```rust
let primes: Vec<i32> = Vec::new();
let primes = vec![2, 3, 5];
println!("{:?}", primes);      // [2, 3, 5]
```

Adding and deleting elements:

```rust
let mut primes: Vec<i32> = Vec::new();
primes.push(2);                // [2]
primes.push(3);                // [2, 3]
primes.remove(0);              // [3]
```

Create vactors with default values:

```rust
let numbers = vec![2;4];       // [2, 2, 2, 2]
const DEFAULT: bool = true;
let values = vec![DEFAULT;4];  // [true, true, true, true]
```

Updating elements:

```rust
let mut numbers = vec![0;4];
numbers[3] = 2;                // [0, 0, 0, 2];
```

Using an iterator:

```rust
for number in numbers.iter() {
    println!("{}", number);
}
```

## Slices

> A slice is a pointer to a block of memory

```rust
let numbers = [1, 2, 3, 4, 5];
let slice = &numbers[1..4];      // [2, 3, 4], index at 4 is not included
```

- Size is determined at runtime

- Can be used on arrays, vectors and strings
- Indexed

Mutable slices allow us to change values:

```rust
let mut colors = ["red", "blue", "green", "black"];
update_colors(&mut colors[2..4]);
println!("{:?}", colors);      // ["red", "blue", "yellow", "orange"]

fn update_colors(colors_slice: &mut [&str]) {
    colors_slice[0] = "yellow";
    colors_slice[1] = "orange";
    // colors_slice[2] = "brown";       // error at runtime, index out of bounds
}
```

## Tuples

> A collection of values of various types

```rust
let person = ("John", 27, true);
let person: (&str, i32, bool) = ("John", 27, true);
println!("{:?}", person.0);     // "John"
```

- **Static**: cannot be resized
- Element values can be updated
- Indexed
- **Limited to 12 elements**

Updating elements:

```rust
let mut person = ("John", 27);
person.0 = "Jack";
```

Desctructuring a tuple. **Number of variables must correspond to number of elements**:

```rust
let person = ("John", 27, true);
let (name, age) = person;             // error
let (name, age, employed) = person;   //ok
```

Empty tuple is also known as **Unit Type**. A function returns a unit type when not specifiied:

```rust
fn main() {
  let v: () = ();
  assert_eq!(v, implicitly_return_unit());
}

fn implicitly_return_unit() {
  println("I will return a ()");
}
```

## Structures

> A collection of key-value pairs

Can be seen as classes, but in Rust there are not concepts such as objects and classes.

```rust
let emp = Employee {
    name: String::from("John"),
    company: String::from("Google"),
    age: 26
};
println!("{:?}", emp);     // Employee { name: "John", company: "Google", age: 26 }
println!("{}", emp.name)   // John

// this derive make the structure printable
#[derive(Debug)]
struct Employee {
    name: String,
    company: String,
    age: u32,
}
```

Adding methods to a structure:

```rust
println!("{}", emp.fn_details());     // name: John, age: 26, company: Google

impl Employee {
    fn fn_details(&self) -> String {
        format!("name: {}, age: {}, company: {}", &self.name, &self.age, &self.company)
    }
}
```

A structure can have static methods without `&self`:

```rust
println!("{}", Employee::static_fn_detail());    // Details of a person

impl Employee {
    fn static_fn_detail() -> String {
        String::from("Details of a person")
    }
}
```

## Enums

> A collection of values

```rust
let my_color = Colors::Red;
println!("{:?}", my_color);      // Red

// this derive make the enum printable
#[derive(Debug)]
enum Colors {
    Red,
    Green,
    Blue
}
```

Data types can be added to enum elements:

```rust
let person = Person::Name(String::from("John"));
println!("{:?}", person)         // Name(John)

#[derive(Debug)]
enum Person {
    Name(String),
    Surname(String),
    Age(u32)
}
```

## Generics

> Allows us to have varibale data types

```rust
let p1: Point<i32> = Point {
    x: 6,
    y: 8
};
let p2 : Point<f64> = Point {
    x: 3.2,
    y: 45.1
};
println!("{:?}", p1);      // Point { x: 6, y: 8 }
println!("{:?}", p2);      // Point { x: 3.2, y: 45.1 }

#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

let c1 = Red("#f00");
let c2 = Red(255);
println!("{:?}", c1);      // Red("#f00") 
println!("{:?}", c2);      // Red(255)

#[derive(Debug)]
enum Colors<T> {
    Red(T),
    Blue(T),
}
```

# Control Structures

## Conditions

### If Statement

```rust
if number < 5 {
    // ..
} else if number < 10 {
    // ..
} else {
    // ..
}
```

```rust
let condition = true
let number = if condition { 5 } else { 6 }
```

### Match

```rust
let choice = Suit::Heart;

match choice {
    Suit::Heart => {
      // ...
    }
    Suit::Spade => {
      // ...
    }
    Suit::Club => {
      // ...  
    }
    Suit::Diamond => {
      // ...
    }
}

enum Suit {
    Heart,
    Spade,
    Club,
    Diamond,
}
```

```rust
fn country(code: i32) {
    let country = match code {
        44 => {
            "UK"
        }
        34 => "Span",
        1..=999 => "unknown",
        _ => "invalid"
    }
    println!("Country is {}", country)
}
```

### Pattern Matching

```rust
fn get_oranges(amount: i32) -> &'static str {
    return match amount {
        0 => "no",
        // multiple values
        1 | 2 => "one or two",
        // ranges
        3..=7 => "a few",
        // conditions
        _ if amount % 2 == 0 => "an even amount of",
        // default
        _ => "lots of"
    }
}
```

Tuple matching:

```rust
let point = (1, 0);

match point {
    (0, 0) => println!("origin"),
    (x, 0) => println!("x axis ({}, 0)", x),
    (0, y) => println!("y axis (0, {})", y),
    (x, y) => println!("({}, {})", x, y)
}
```

## Loops

```rust
loop {
    // ..
}
```

```rust
let result = loop {
    counter += 1;
    if counter == 10 {
        break counter * 2;  
    }
}
```

```rust
let mut count = 0;
'counting_up: loop {
    println!("count = {count}")
    let mut remaining = 10;
  
    loop {
        printl!("reamianing = {remaining}")    // always 10 -> 9
        if remaining == 9 {
            break;  
        }
        if count == 2 {
            break 'counting_up;  
        }
        remaining -= 1;
    }
  
    count += 1;
}
println!("End count = {count}")  // 2
```

### For Loop

```rust
let a = [10, 20, 30, 40, 50];
for element in a {
    println!("the value is: {element}")
}
```

```rust
for number in (1..4).rev() {
    println!("{number}!")
}
```

```rust
let pets = ["cat", "dog", "chihuahua", "hamster", "bear"];

// pet is &str
for pet in pets {
    if pet == "chihuahua" {
        println!("found chihuahua!")
    }
}

// pet is &&str
for pet in pets.iter() {
    // add & before "chihuahua"
    if pet == &"chihuahua" {
        println!("found chihuahua!")
    }
}
```

### While Loop

```rust
while number != 0 {
    number -= 1;
}
```

# Function

## Scope

**Local scope** in Rust can be specified using `{}`:

```rust
{                      // s is not valid here, it’s not yet declared
    let s = "hello";   // s is valid from this point forward

    // do stuff with s
}                      // this scope is now over, and s is no longer valid
```

**Global varibales** can be declared but they are unsafe:

```rust
static mut a = 3;
fn main() {
    unsafe {
        println!("{}", a);  
    }
}
```

## Closures

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

## Higher Order Functions (HOFs)

> Functions that take another function as a parameter

```rust
let square = |a: i32| { a * a };
apply(square, 3);

fn apply(f: fn(i32) -> i32, a: i32) {
    println!("Result {}", f(a));
}
```

Usage of HOFs with built-in libs:

```rust
// Calculate the sum of all the squares less than 500
// Only for even numbers
let limit = 500;
let mut sum = 0;
for i in 0.. {
    let isq = i * i;
    if isq > limit { break; }
    if is_even(!isq) { continue; }
    sum += isq;
}
println!("The sum is {}", sum);

// With HOFs
let sum2 =
    (0..).map(|x| x * x)
        .take_while(|&x| x <= limit)
        .filter(|x| is_even(*x))
        .fold(0, |sum, x| sum + x);

println!("The sum2 is {}", sum2);

fn is_even(x: i32) -> bool {
    x % 2 == 0
}
```

## Macros

> Write code that writes code - meta programming

Match an expression and perform som operation

Code is expanded and compiled

```rust
macro_rules! my_macro {
    () => {
        println!("First macro")
    };
}

my_macro!();
```

```rust
// macro with one parameter
macro_rules! name {
    ($name: expr) => {
        println!("Hey {}", $name)
    };
}

name!("John");
```

```rust
// multiple parameters
macro_rules! super_name {
    ($($name: expr),*) => {
        $(println!("Hey {}", $name);)*
    };
}

super_name!("Harvey", "John", "Jack");
```

```rust
// multiple match statements
macro_rules! xy {
    (x => $e: expr) => (println!("x is {}", $e));
    (y => $e: expr) => (println!("y is {}", $e));
}

xy!(x => 5);
xy!(y => 3 * 9);
```

```rust
// ident as parameter, create a function using macro
macro_rules! build_fn {
    ($fn_name: ident) => {
        fn $fn_name() {
            println!("{:?} was called", stringify!($fn_name));
        }
    };
}

build_fn!(hey);
hey();
```

# Traits

> Similar to an interface or abstract class

Add definition to a structure

# Common Concepts



## Functions

### Diverging Functions

 **分流函数（Diverging Functions）**永远不会返回：

```rust
fn main() {
    never_return();
    // 下面的永远不会执行
}

fn never_return() -> ! {
    // 抛出一个错误
    panic!()
}
```

```rust
fn main() {
  
}

fn get_option(tp: u8) -> Option<i32> {
    match tp {
        1 => {
            // TODO  
        }
        _ => {
            // TODO  
        }
    };
  
    // 使用分流函数，而不是返回 None
    never_return_fn()
}

fn never_return_fn() -> ! {
    // panic!()
    // unimplemented!()
    todo!()
}
```

# Ownership

Rust 中的一个核心概念是 **所有权（Ownership）**。Rust 中没有 GC，但也不需要开发者手动释放内存来防止内存泄漏，而是使用了 Ownership 与一系列的编译时规则来实现内存安全。

## Heap and Stack Memory

大多数语言都有 **堆内存和栈内存**，但通常不需要太过深究，但 Rust 中数据是存在堆还是栈中会很大程度上影响其表现。

堆与栈内存都是在运行时代码可以使用的内存，关于堆栈的相关介绍情看 {% post_link Algorithm %}  相关部分。

**重要的是：**

- **在栈中存储的数据必须是确定的、固定的大小；**
- **在堆中分配（Allocate）内存时，会返回一个指针指向这个内存的地址，该定长地址可以存储在栈中；**
- 向栈中 Push 数据比在堆中 Allocate 要快得多，因为不需要先找到合适的地方来存储这个数据；
- 访问栈中的数据要比对堆中的快得多，因为堆中的数据要顺着指针来找；
- 调用函数的时候，传入函数的参数与函数本地的数据会 Push 进栈中，函数运行结束后，栈中的数据会被 Pop 出来；
- 栈内存有天然的 Push 和 Pop，使其不会有冗余的数据，但堆内存则需要及时堆不需要的部分进行清理。

## Ownership Rules

Rust 中有三条 Ownership Rules：

- **Rust 中的每个值都有一个 Owner。**
- **每个值同时只能有一个 Owner。**
- **当 Owner 离开作用域（Scope）时，值会被丢弃。**

## Complex Types

对于简单类型，比如 `integer`，表现跟其他语言类似：

```rust
let x = 5;
let y = x;
```

以上代码，会创建两个变量 `x` 和 `y`，均为 `5`，并 Push 到了 **栈** 中。

而复杂类型则不同，他会使用 **堆** 内存，下面以 `String` 举例。

### String vs. &str

```rust
// 可以通过字符串字面量创建一个 String
let mut s = String::from("hello");
s.push_str(", world!")
```

`String` 是复杂类型，他由三部分组成（这三部分数据保存在 **栈** 中）：

- 一个指针指向 **堆** 中保存字符串内容的的内存 `ptr`；
- 字符串的长度 `len`；
- 字符串的容量 `capacity`。

因此上面 `s` 会在栈中占据 `3 * 8 = 24 Bytes` 大小，

### Copy

简单类型赋值时，会**自然地将栈中的数据复制一份**，并且原来的变量也不会失效，这称为 **Copy**：

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

### Move

而复杂类型则不同，比如对于 `String`，如果我们将 `s1` 赋值给 `s2`，那么 **堆中的数据不会被复制，栈中的数据会被复制一份**：

```rust
let s1 = String::from("hello");
let s2 = s1;
```

此外，由于变量离开作用域时，Rust 会自动调用 `drop` 方法来清理内存，这就导致如果 `s1` 和 `s2` 都离开作用域时，堆中的同一块内存会被清理两次，这会导致内存安全问题。

**因此，当 `s1` 赋值给 `s1` 之后，`s1` 将不再可用。**这样的话 Rust 就不会再清理 `s1` 的内存。

**这种特性与其他语言的浅拷贝（Shallow Copy）有所不同，Rust 称其为移动（Move），因为之前的变量将不再可用。**

### Clone

如果想要对复杂类型执行 **深拷贝（Deep Copy）**，不仅复制其栈中的数据，还复制其堆中的数据，可以使用这些类型中提供的 `clone` 方法：

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

## Ownership and Functions

```rust
fn main() {
    let s = String::from("hello");  // s comes into scope

    takes_ownership(s);             // s's value moves into the function...
                                    // ... and so is no longer valid here

    let x = 5;                      // x comes into scope

    makes_copy(x);                  // x would move into the function,
                                    // but i32 is Copy, so it's okay to still
                                    // use x afterward

} // Here, x goes out of scope, then s. But because s's value was moved, nothing
  // special happens.

fn takes_ownership(some_string: String) { // some_string comes into scope
    println!("{}", some_string);
} // Here, some_string goes out of scope and `drop` is called. The backing
  // memory is freed.

fn makes_copy(some_integer: i32) { // some_integer comes into scope
    println!("{}", some_integer);
} // Here, some_integer goes out of scope. Nothing special happens.
```

```rust
fn main() {
    let s1 = gives_ownership();         // gives_ownership moves its return
                                        // value into s1

    let s2 = String::from("hello");     // s2 comes into scope

    let s3 = takes_and_gives_back(s2);  // s2 is moved into
                                        // takes_and_gives_back, which also
                                        // moves its return value into s3
} // Here, s3 goes out of scope and is dropped. s2 was moved, so nothing
  // happens. s1 goes out of scope and is dropped.

fn gives_ownership() -> String {             // gives_ownership will move its
                                             // return value into the function
                                             // that calls it

    let some_string = String::from("yours"); // some_string comes into scope

    some_string                              // some_string is returned and
                                             // moves out to the calling
                                             // function
}

// This function takes a String and returns one
fn takes_and_gives_back(a_string: String) -> String { // a_string comes into
                                                      // scope

    a_string  // a_string is returned and moves out to the calling function
}
```

## Reference and Borrowing

如果传参的时候交出了 Ownership，原来的变量就不可用了，因此需要将原来的值再手动返回回来，重获 Ownership：

```rust
fn main() {
    let s1 = String::from("hello");

    let (s2, len) = calculate_length(s1);

    println!("The length of '{}' is {}.", s2, len);
}

fn calculate_length(s: String) -> (String, usize) {
    let length = s.len(); // len() returns the length of a String

    (s, length)
}
```

**这时可以使用引用（Reference），这样函数就不会拿到 Ownership，在其作用域结束的时候也就不会被清除：**

```rust
fn main() {
    let s1 = String::from("hello");

    // &s1 创建了一个 Reference，指向 s1，但并没有 **拥有** 它，Ownership 没有发生转移
    let len = calculate_length(&s1);

    println!("The length of '{}' is {}.", s1, len);
}

// &String s 指向 String s1
fn calculate_length(s: &String) -> usize {
    s.len()
} // Here, s goes out of scope. But because it does not have ownership of what
  // it refers to, it is not dropped.
```

创建 Reference 的过程称为 **借用（Borrowing）**。

### Mutable Reference

可以通过 `&mut` 来创建可变引用：

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s);
}

fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

**Mutable References 有一些限制：**

- **每个变量同时只能有一个 Mutable Reference（在一个作用域中），但可以拥有多个 Immutable References；**
- **变量不能同时拥有 Mutable Reference 和 Immutable Reference（防止 Dangling References）；**

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
let r3 = &mut s;   // Error

println("{}, {}, and {}", r1, r2, r3);
```

```rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
println("{} and {}", r1, r2);
// variables r1 and r2 will not be used after this point

let r3 = &mut s;   // No problem
println("{}", r3);
```

