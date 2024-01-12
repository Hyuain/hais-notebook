---
title: Rust
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RS1
---

熟练掌握三门语言计划之三：Rust。

资料来源：

- [The Book](https://doc.rust-lang.org/)
- [Learn Rust Programming - Complete Course](https://www.youtube.com/watch?v=BpPEoZW5IiY)

<!-- more -->

# Project Basics

## Rustup

Rust is installed and managed by the [`rustup`](https://rust-lang.github.io/rustup/) tool. It installs Rust from the official release channels, enabling you to easily switch between stable, beta, and nightly compilers and keep them updated.

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

We can use `cargo add` to add dependencies to a cargo project:

````bash
cargo add tokio
````

And use `--features` to specify features:

```bash
cargo add --features full
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

## Unit Testing

 ```rust
 pub fn add_five(num: u32) -> u32 {
     num + 5
 }
 
 // the code only runs in `cargo test`
 #[cfg(test)]
 mod test {
     // inherit libraries imported
     use super::*;
   
     #[test]
     fn add_five_test() {
         let x = 100;
         let y = add_five(x);
         println!("x and y are from test {}", x, y);
         assert_eq!(y, 105);
     }
 }
 ```

Then use `cargo test` to run tests.

To make content in `println!()` printed in the console, use the following command:

```dahs
cargo test -- --nocapture
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

- **Binary Crates:** the main component (entry point) of the crate (`main.rs`) has a `main` function.

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

- **Library Crates:** can be without a entry point, but sometimes has a `lib.rs` as a entry point exporting functions.

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

# Macros

> Write code that writes code - meta programming

Match an expression and perform some operation

## Declarative Macros

Declarative macros start with a exclamation mark `!`. Here are some example of declarative macros:

```rust
println!("Hello 1");
dbg!("Hello 2");
let x: Vec<i32> = vec![1, 2, 3];
let formatted: String = format!("Hello 3 with vec {:?}", x);
dbg!(formatted);
```

**Macro captures**

| Keyword |          Explaination          |                     Examples                     |
| :-----: | :----------------------------: | :----------------------------------------------: |
| `expr`  | Mathes a valid rust expression | `"hello".to_string"` `vec![1, 2, 3]` `1 + 2` `1` |
| `stmt`  |  Matches to a rust statement   |     `let x= 5` `x.push(1)` `return Some(x)`      |
| `ident` |  Matches to a rust identifier  |    variable name, function name, module name     |
|  `ty`   |     Matches to a rust type     |         `i32` `Vec<String>` `Option<T>`          |
| `path`  |     Matches to a rust path     |           `std::collections::HashMap`            |

 **Repitition Specifier**

`*` - Match zero or more repititions

`+` - Match one or more repititions

`?` - Match zero or one repitition

**Examples**

Capture expressions

```rust
macro_rules! mad_skills {
    ($x: expr) => {
        format!("You sent an expression: {}", $x)
    };
}

let some_var = mad_skills!(1 + 2);
dbg!(some_var);    // 3
```

Capture types

```rust
macro_rules! mad_skills {
    ($x: ty) => {
        match stringify!($x) {
            "i32" => "You sent an i32 type".to_string(),
            _ => format!("You sent {} type", stringify!($x)).to_string()
        }
    }
}

let some_var = mad_skills!(i32);
dbg!(some_var);
let some_var = mad_skills!(Vec<i32>);
dbg!(some_var);
```

Capture identifiers (eg. function names)

```rust
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

Create `my_vec!` with repetitions

```rust
 macro_rules! my_vec {
    // expect one or more parameters
    ($($x: expr), +) => {
        // ensure variables inside the macro do not leak out to the surrounding code
        {
            let mut temp_vec = Vec::new();
            // repeat this statement one or more times
            $(
                temp_vec.push($x);
            )+
            temp_vec
        }
    };
}

// the same as my_vec!(1, 2);
let mut y = my_vec![1, 2];
dbg!("{:?}", y);   // [1, 2]
```

Multiple mathces

```rust
// multiple match statements
macro_rules! xy {
    (x => $e: expr) => (println!("x is {}", $e));
    (y => $e: expr) => (println!("y is {}", $e));
}

xy!(x => 5);
xy!(y => 3 * 9);
```

Export the macro to let it can be used in other files

```rust
#[macro_export]
macro_rules! my_vec { ... }
```

## Procedural Macros

### Derive Macros

```rust
#[derive(Debug, Serialize, Clone, Deserialize, PartialEq)]
struct User {
  //...
}
```

To create a derive macro, a new library should be created first. In the `cargo.toml` of the library, set `proc-macro` to `true`

```toml
[lib]
proc-macro = true

[dependencies]
syn = "2.0.18"      # Interact a syntax tree
quote = "1.0.28"    # Interpolate different types, identifiers, names, expressions, etc. into string
```

In the project file

```rust
extern crate proc_macro;  // A special crate included with the Rust Standard Library

use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

// Specify this function is a derive macro, and its name is HelloWorld
#[proc_macro_derive(HelloWorld)]
pub fn helloworld_object_derive(input: TokenStream) -> TokenStream {
    // Parse the input tokens into a syntax tree
    let input = parse_macro_input!(input as DeriveInput);
    // Used in the quasi-quotation below as the name of the type to implement
    let name = input.ident;
    // Generate the code to parse into the user's program
    let expanded = quote! {
        impl HelloWorld for #name {
            fn hello_world() {
                println!("Hello, world!");
            }  
        }
    };
    // Hand the output tokens back to the compiler
    TokenStream::from(expanded)
}
```

Add the macro as a denpendency in another project

```toml
[dependencies]
proc_macro = { path = "../proc_macro" }
```

### Function Like Macro

To create a function like procedural macro, add tag `#[proc_macro]`

```rust
#[proc_macro]
pub fn create_struct(input: TokenStream) -> TokenStream {
    // Parse the input tokens into a syntax tree
    let input = parse_macro_input!(input as Ident);
  
    // Construct a string representation of the struct definition
    let name: &{unknown} = &input;
    let expanded = quote! {
        struct #name {
            x: i32,
            y: i32,
        }
    };
  
    // Return the generated struct as a TokenStream, like TokenStream::from(expanded)
    expended.into()
}
```

Then the macro can be imported and used like declarative macros

```rust
use create_struct_macro::create_struct;

create_struct!(Point);

fn main() {
    let Point = Point { x: 5, y: 10 };
    println!("Point coordinates: x = {}, y = {}", point.x, point.y);
}
```

### Attribute Like Macro

Use `#[proc_macro_attribute]` to create a attribute like macro

```rust
#[proc_macro_attribute]
pub fn function_to_string(_attr: TokenStream, item: TokenStream) -> TokenStream {
    // Parse the input function
    let input_fn: ItemFn = parse_macro_input!(item as ItemFn);
  
    // Create a string representation of the function
    let function_str: String = format!("{}", input_fn.to_token_stream());
  
    // Define a new function with the same signature as the input function
    let fn_ident: proc_macro2::Ident = input_fn.sig.ident;
    let fn_inputs: sys::punctuated::Punctuated<syn::FnArg, syn::token::Comma> = input_fn.sig.inputs;
    let fn_generics: syn::Generics = input_fn.sig.generics;
  
    // Generate the output code
    let output: proc_macro2::TokenStream = quote! {
        pub fn #fn_ident #fn_generics(#fn_inputs) -> &'static str {
            #function_str
        }
    }
  
    output.into()
}
```

Use case of the `function_to_string` macro

```rust
use proc_macro::function_to_string;

#[function_to_string]
pub fn convert_user_input_to_goal(_user_request: &str) {
    println!(OUTPUT)
}
```

## Example

Build a 

# Traits

## Traits

> Similar to an interface or abstract class

Add definition to a structure

```rust
trait Name {
  fn must_implement(&self) -> i32;
  fn do_action(&self) { ... }
  fn do_non_instance_action() { ... }
  // Can provide a constructor
  fn new(name: &str) -> Self;
}
```

Implement a trait:

```rust
impl Name for Person {
  fn must_implement(&self) -> { 42 }
  fn new(name: &str) -> Person {
    Person{name: name}
  }
}
```

Use a implemented trait:

```rust
let john = Person::new("John")
```

Example:

```rust
fn main() {
    // basic default instantiation
    // let r = RustDev { awesome: true };
    // structure instantiate itself
    let r = RustDev::new(true);
    let j = JavaDev::new(false);
    println!("{}", r.language());
    r.say_hello();
    println!("{}", j.language());
    j.say_hello();
}

struct RustDev {
    awesome: bool
}

struct JavaDev {
    awesome: bool
}

trait Developer {
    fn new(awesome: bool) -> Self;
    fn language(&self) -> &str;
    fn say_hello(&self) { println!("Hello world!") }
}

// RustDev implements Developer
impl Developer for RustDev {
    fn new(awesome: bool) -> Self {
        RustDev { awesome }
    }

    fn language(&self) -> &str {
        "Rust"
    }

    fn say_hello(&self) {
        println!("println(\"Hello world!\");")
    }
}

// JavaDev implements Developer
impl Developer for JavaDev {
    fn new(awesome: bool) -> Self {
        JavaDev { awesome }
    }

    fn language(&self) -> &str {
        "Java 1.8"
    }

    fn say_hello(&self) {
        println!("System.out.println(\"Hello world!\");")
    }
}
```

## Trait Generics

> Generics can be limited by traits

```rust
let dog = Dog { species: "retriever" };
let cat = Cat { color: "black" };
bark_it(dog);
bark_it(cat); // error: the trait bound `Cat: Bark` is not satisfied

trait Bark {
    fn bark(&self) -> String;
}

struct Dog {
    species: &'static str
}

struct Cat {
    color: &'static str
}

// Dog implements Bark
impl Bark for Dog {
    fn bark(&self) -> String {
        format!("{} barking", self.species)
    }
}

fn bark_it<T: Bark>(b: T) {
    println!("{}", b.bark());
}
```

## Returning Traits

The compiler needs to know the space required for a funciton return type, so **a generic can not be used as a return type**.

```rust
trait Animal {
    fn make_noise(&self) -> &'static str;
}

struct Dog {}
struct Cat {}

fn get_animal(rand_number: f64) -> Animal {  // error: trait objects must include the `dyn` keyword
    if rand_number < 1.0 {
        Dog {}
    } else {
        Cat {}
    }
}
```

A workaround is to return a box with a dyn trait:

```rust
struct Dog {}
struct Cat {}

impl Animal for Dog {
    fn make_noise(&self) -> &'static str {
        "woof"
    }
}

impl Animal for Cat {
    fn make_noise(&self) -> &'static str {
        "meow"
    }
}

// should add Box<dyn ...>
fn get_animal(rand_number: f64) -> Box<dyn Animal> {
    if rand_number < 1.0 {
        Box::new(Dog {})
    } else {
        Box::new(Cat {})
    }
}
```

dyn is a new addition to the language, old code might not have it.

## Adding Traits to Existing Structures

```rust
fn main() {
    let a: Vec<i32> = vec![1, 2, 3, 4, 5];
    println!("sum = {}", a.sum());
    let b: Vec<f64> = vec![1.0, 2.0, 3.0, 4.0];
    println!("sum float = {}", b.sum());    // error
}

trait Summable<T> {
    fn sum(&self) -> T;
}

impl Summable<i32> for Vec<i32> {
    fn sum(&self) -> i32 {
        let mut sum: i32 = 0;
        for i in self {
            sum += *i;
        }
        sum
    }
}

```

## Operator Overloading

We can implement standard operators for our custom structs:

```rust
use std::ops::Add;

fn main() {
    let p1 = Point { x: 1.2, y: 2.4 };
    let p2 = Point { x: 1.0, y: 3.0 };
    let p3 = p1 + p2;
    println!("{:?}", p3)
}

#[derive(Debug)]
struct Point {
    x: f64,
    y: f64,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Self) -> Self::Output {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

```

## Static Dispatch

> A generic trait will be converted to the required type at compile time

Monomorphization: converting to one form. The compiler will convert the function into each required code.

```rust
fn main() {
    let a = 42;
    let b = "Hi John".to_string();
    duplicate(a);
    duplicate(b);
}

trait Duplicatable {
    fn dupl(&self) -> String;
}

impl Duplicatable for String {
    fn dupl(&self) -> String {
        format!("{0} {0}", *self)
    }
}

impl Duplicatable for i32 {
    fn dupl(&self) -> String {
        format!("{}", *self * 2)
    }
}

// duplicate function will be create once for each implementation for generic
// so there is one duplicate function for String, and one for i32
fn duplicate<T: Duplicatable>(x: T) {
    println!("{}", x.dupl());
}
```

## Dynamic Dispatch

> A generic trait will be converted to the required type at run time

```rust
fn main() {
    let a = 42;
    let b = "Hi John".to_string();
    duplicate(&a);
    duplicate(&b);
}

// use &dyn (the function also be created twice)
fn duplicate(x: &dyn Duplicatable) {
    println!("{}", x.dupl());
}
```

# Memory Management

## Heap and Stack Memory

大多数语言都有 **堆内存和栈内存**，但通常不需要太过深究，但 Rust 中数据是存在堆还是栈中会很大程度上影响其表现。

堆与栈内存都是在运行时代码可以使用的内存，关于堆栈的相关介绍情看 vvv Algorithm %}  相关部分。

|            | Stack                               | Heap                                                |
| ---------- | ----------------------------------- | --------------------------------------------------- |
| Size       | Fixed (known at compile time)       | Dynamic                                             |
| Speed      | Faster than Heap                    | Slower than Stack                                   |
| Usage      | Local variables, function call data | Dynamic data, largeobjects that outlive their scope |
| Management | Automatic in Rust                   | Automatic in Rust (ownership system)                |
| Scope      | Within scope where declared         | Anywhere complying with ownership rules             |

An example of stack:

```rust
fn man() {
    let a = 10;          // stored in stack frame main()
    let b = 20;          // stored in stack frame main()
    some_func {          // create new stack frame {}
        let c = 30;      // stored in stack frame {}
        let d = &c;      // the value of d is the address of c, d is stored in stack frame {}
    }                    // stack frame {} poped, c and d droped
    let e = add_five(a)  // create new stack frame add_five(), num and i stored there
}

fn add_five(num: i32) -> i32 { // num has the same value as a, but in different address
    let i = 5;
    num + 1;
}
```

## Primitive vs. Complex Types

对于简单类型，比如 `integer`，表现跟其他语言类似：

```rust
let x = 5;
```

以上代码，会创建变量 `x`，并 Push 到了 **栈** 中。

而复杂类型则不同，他会使用 **堆** 内存，下面以 `String` 举例。

```rust
// 可以通过字符串字面量创建一个 String
let mut s = String::from("hello");
s.push_str(", world!")
```

`String` 是复杂类型，他由三部分组成（这三部分数据保存在 **栈** 中）：

- 一个指针指向 **堆** 中保存字符串内容的的内存 `ptr`；
- 字符串的长度 `len`；
- 字符串的容量 `capacity`。

因此上面 `s` 会在栈中占据 `3 * 8 = 24 Bytes` 大小。

`&str` 有点微妙，被称为字符串切片（String Slice），他是基础类型 `str` 的引用。`str` 与 `i32` 等基础类型不同，他没有固定的长度，并且可能保存在堆中、栈中、或者是程序文本中。通常我们不能直接使用他，而是通过 `&str` 来使用他，`&str` 作为一个引用，他本身的长度是固定的，因此是保存在栈中的。

Rust 中，`&str` 类型的数据可以直接使用 `str` 的方法，比如 `len()`，因为 Rust 会自动帮我们解引用（Dereference）。

Rust 中，可以通过 Borrowing 从 `String` 中拿到 `&str`：

```rust
let owned_string: String = "Hello, world!".to_string();
// NOTE: Deref coercion happened! &String -> &str
let string_slice: &str = &owned_string; // Borrowing a slice of the whole String
```

## Ownership

> Only one variable can own a piece of memory

Copy and Move:

- For primitive types (like i32), copying data is cheap
- For complex types (like struct), ownership is transferred

Assigning values of **primitives**, **data stored in stack is copied**, previous data is still available.

```rust
let x = 5;
let y = x;

println!("x = {}, y = {}", x, y);
```

For **complex** values, if value of `s1` is assigned to `s2`, **data in stock is copied, while data in heap is not copied.**

This is called **move** in Rust.

```rust
let s1 = String::from("hello");
let s2 = s1;

println!("{}", s1);     // error, because value is nolonger owned by s1
```

Because in Rust  `drop` method is called to clean up memory.

**To prevent the same memory being clean up twice**, which can cause memory safety issues, `s1` is nolonger accessible after ownership transferred.

This way, memory of `s1` is not considered in cleaning up.

Here is another example:

```rust
fn main() {
    let v = vec![1, 2, 3, 4];

    let foo = |v: Vec<i32>| -> Vec<i32> {
        println!("Vector used in foo");
        v
    };
    
    let x = foo(v);
    println!("{:?}", v);   // error, because v is passed into v and nolonger accessible
}
```

**Move** is different from shallow copy in other languages, because the previous variables are nolonger accessible. 

## Clone

如果想要对复杂类型执行 **深拷贝（Deep Copy）**，不仅复制其栈中的数据，还复制其堆中的数据，可以使用这些类型中提供的 `clone` 方法：

```rust
let s1 = String::from("hello");
let s2 = s1.clone();

println!("s1 = {}, s2 = {}", s1, s2);
```

## Borrowing

> Variables can borrow ownership to other pieces of memory

```rust
let a = 6;
let b = &a;            // b borrows ownership from a
println!("{}", *b);    // once b is destroyed, the ownership is transfered back to a
```

The borrow has to match the mutability

- **Immutable variable cannot have mutable references**
- One variable can only have one **mutable reference** (in each scope), but can have **mutiple immutable references**
- Variables can not have **mutable reference** and **immutable reference** at the same time (prevent dangling references)

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

```rust
let mut a = 6;
let b = &mut a;
println!("{}", a);  // Error, mutable borrow occurs after
println!("{}", *b);
```

## Lifetimes

> An indication of how long an object can live

Rust prevents parts of objects outliving the object

```rust
struct Object<'lifetime> {
    field: &'lifetime str
}
```

```rust
fn main() {
    let p1 = Person { name: String::from("John") };
    let d1 = Dog { name: String::from("Max"), owner: &p1 };

    println!("{:?}", d1);
}

#[derive(Debug)]
struct Person {
    name: String
}

#[derive(Debug)]
struct Dog<'l> {
    name: String,
    // a reference to Person
    // Rust need to know how long the Person should live
    // to match the lifetime of Dog and Person
    owner: &'l Person,
}
```

To live as long as the program is running:

```rust
&'static str
```

Lifetime elision: Compiler builds lifetimes for us when evident

```rust
fn main() {
    let p1 = Person { name: String::from("John") };
    let mut a: &String;
    let mut b: &String;
    {
        let p2 = Person { name: String::from("Mary") };
        a = p1.get_name();
        b = p2.get_name();
    }
    println!("{:?}", b);    // error, borrowed value does not live long enough
  	println!("{:?}", a);    // ok
}

#[derive(Debug)]
struct Person {
    name: String
}

impl Person {
    // equals: fn get_name<'l>(&'l self) -> &'l String {
    fn get_name(&self) -> &String {
        &self.name
    }
}
```

## Rc (Referenced Counted Variables)

> A structure that can hold multiple references to a variable, and can be shared in different places

```rust
use std::rc::Rc;

fn main() {
    let brand = Rc::new(String::from("BMW"));
    println!("pointers: {}", Rc::strong_count(&brand));
    {
        let car = Car::new(brand.clone());
        car.drive();
        println!("pointers: {}", Rc::strong_count(&brand));
    }
    println!("My car is a {}", brand);
    println!("pointers: {}", Rc::strong_count(&brand));
}


struct Car {
    brand: Rc<String>
}

impl Car {
    fn new(brand: Rc<String>) -> Car {
        Car { brand }
    }
    fn drive(&self) {
        println!("{} is driving", &self.brand);
    }
}

```

Rc is useful to use multiple references without considering ownership and borrowing.

## Examples

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

# Error Handling

## Files

Create a file

```rust
use std::fs::File;
let mut file = File::create("src/example.txt").expect("create failed");
```

Write to a file

```rust
use std::io::Write;
file.write_all("Hello World\n".as_bytes()).expect("write failed");
```

Append content to a file

```rust
use std::fs::OpenOptions;
let mut file = OpenOptions::new().append(true).open("src/example.txt").expect("open failed");
```

Read from a file

```rust
use std::io::Read;
let mut file = File::open("src/example.txt").unwrap();
let mut contents = String::new();
file.read_to_string(&mut contents).unwrap();
println!("{}", contents);
```

Delete a file

```rust
use std::fs::remove_file;
remove_file("src/example.txt").expect("delete failed");
```

## Error Handling

2 types of errors:

- **Recoverable:** `Result` enum
- **Unrecoverable:** `panic!` macro

Index out of bounds is a kind of unrecoverable error

```rust
let v = vec![1, 2, 3];
v[10];      // panic
```

A panic can also raised mannually

```rust
panic!("Something went wrong. cannot proceed");
```

Panic will terminate the program or thread, and we can do nothing.

Recoverable error however can be handled and proceed using `Result` enum:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E)
}
```

Rust will actually return a `Result` if we try to open a file:

```rust
let f = File::open("main.jpg"); // f is Result<File>
match f {
    Ok(f) => {
        // f is File
        println!("file found {:?}", f);
    },
    Err(e) => {
        // e is Error
        println!("file not found \n{:?}", e);
    }
}
println!("Continuing on with the execution");
```

We can also use `Option ` enum:

```rust
enum Option<T, E> {
    Some(T),
    None
}
```

```rust
divide(Some(1));    // result is 42
divide(Some(10));   // result is 4
divide(None);       // None received, the answer is 42
divide(Some(0));    // panic

const ANSWER_TO_LIFE: i32 = 42;

fn divide(x: Option<i32>) {
    match x  {
        Some(0) => panic!("Cannot divide by 0"),
        Some(x) => println!("result is {}", ANSWER_TO_LIFE / x),
        None => println!("None received, the answer is {}", ANSWER_TO_LIFE)
    }
}
```

## Helper Methods

`unwrap()` will return data if it's available or `panic!` of it's not

```rust
// will return a file or panic!
File::open("src/example.txt").unwrap();
```

`expect()` similar to unwrap but allows us to set a custom error message

```rust
File::open("src/example.txt").expect("Unable to open file");
```

## ? Operator

A shorthand for an entire match statement, showing we want the result only if the statement succeeded.

```rust
let a = read_from_file();
println!("{:?}", a);  // Ok or Err

fn read_from_file() -> Result<String, io::Error> {
    let f = File::open("src/example.txt");
    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };
    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

Above  code can be simplified using `?`

```rust
fn read_from_file() -> Result<String, io::Error> {
    let mut f = File::open("src/username.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

# Concurrency

## Threads

Run code in parallel

Ownership/ borrowing mechanism gives us

- memory safety (even in multi-threads programs)
- no data races (because the data only have exactly one owner)

Create a thread

```rust
use std::thread;
thread::spawn(|| {
    println!("new thread");
});
```

`move` keyword helps to use variables inside the clousre

```rust
for i in 0..10 {
    // to use i inside the clouser, we must add `move`, moving data into the closure
    thread::spawn(move || {
        println!("new thread {}", i);
    });
}

println!("Main thread");

/*
new thread 1
new thread 2
new thread 3
new thread 0
new thread 4
new thread 5
new thread 6
new thread 7
new thread 8
Main thread
new thread 9
*/
```

Sleep a thread using `sleep()` method

```rust
for i in 0..10 {
    thread::spawn(move || {
        // sleep the program
        sleep(Duration::from_millis(i * 1000));
        println!("new thread {}", i);
    });
}

println!("Main thread");

/*
new thread 0
Main thread (main program finished before other threads have finished)
*/
```

Wait for a thread use `join()` method

```rust
let mut threads = vec![];

for i in 0..10 {
    let th = thread::spawn(move || {
        sleep(Duration::from_millis(i * 1000));
        println!("new thread {}", i);
    });
    threads.push(th);
}

for th in threads {
    th.join();
}

println!("Main thread");

/*
new thread 0
new thread 1
new thread 2
new thread 3
new thread 4
new thread 5
new thread 6
new thread 7
new thread 8
new thread 9
Main thread
*/
```

## Channels

A way to send data between threads

> **MPSC: Multiple producer single receiver**

Create a channel

```rust
use std::sync::mpsc;
// transimiter and receiver
let (tx, rx) = mpsc::chanel();
```

Send a message

```rust
tx.send()
```

Receive a message

```rust
rx.recv()      // blocking
rx.try_recv()  // non blocking
```

A example of blocked receving. The main thread will wait until it receives message.

```rust
let (tx, rx) = mpsc::channel();
thread::spawn(move || {
    tx.send(42).unwrap();
});
println!("received {}", rx.recv().unwrap());
```

Use `rx` to let the main thread wait for all other threads like `join`:

```rust
const NUM_THREADS: usize = 20;

fn main() {
    let (tx, rx) = mpsc::channel();
    for i in 0..NUM_THREADS {
        start_thread(i, tx.clone());
    }

    for j in rx.iter().take(NUM_THREADS) {
        println!("received {}", j);
    }
}

fn start_thread(d: usize, tx: mpsc::Sender<usize>) {
    thread::spawn(move || {
        println!("setting timer {}", d);
        thread::sleep(Duration::from_secs(d as u64));
        println!("sending {}", d);
        tx.send(d).unwrap();
    });
}

```

## Mutex

> Mutual exclusion lock. Only one thread can access the data at any one time.

**Arc - Atomically referenced counted type.** Convert data into primitive types, safe to share across threads.

Create a lock:

```rust
use std::sync::{Mutex, Arc};
let lock = Arc::new{Mutex::new(0)};
```

Acquire a lock:

```rust
lock.lock();
lock.try_lock();
```

A example that a mutal value is stored using `Mutex::new()` and shared with `Arc`:

```rust
let c = Arc::new(Mutex::new(0));
let mut threads = vec![];

for _i in 0..10 {
    let c = Arc::clone(&c);
    let t = thread::spawn(move || {
        let mut num = c.lock().unwrap();
        *num += 1;
    });
    threads.push(t);
}

for th in threads {
    th.join().unwrap();
}

println!("Result {}", *c.lock().unwrap());    // 10
```

Poisoned lock - when a thread that holds the lock panics:

```rust
lock.is_poisoned();
```

