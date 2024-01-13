---
title: Rust Error Handling
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---

# Files

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

# Error Handling

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

# Helper Methods

`unwrap()` will return data if it's available or `panic!` of it's not

```rust
// will return a file or panic!
File::open("src/example.txt").unwrap();
```

`expect()` similar to unwrap but allows us to set a custom error message

```rust
File::open("src/example.txt").expect("Unable to open file");
```

# ? Operator

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
