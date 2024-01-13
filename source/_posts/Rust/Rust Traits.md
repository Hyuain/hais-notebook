---
title: Rust Traits
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

# Traits

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

# Trait Generics

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

# Returning Traits

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

# Adding Traits to Existing Structures

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

# Operator Overloading

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

# Static Dispatch

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

# Dynamic Dispatch

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
