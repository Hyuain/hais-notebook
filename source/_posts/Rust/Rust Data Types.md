---
title: Rust Data Types
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

## Scalar Data Types

Rust 中的数据类型分为 **标量类型（Scalar Types）** 和 **组合类型（Compound Types）**。**标量类型** 包括：

- **整型（Integer）**：包括 **有符号整型（`i` 开头）** 和 **无符号整型（`u` 开头）**，比如 `i8` `u128` 等，数字表示他占了多少位。其中 `isize` 和 `usize` 类型取决于程序运行所处的计算机的体系结构，比如 64 位或 32 位等。**默认为 `i32`。**
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
