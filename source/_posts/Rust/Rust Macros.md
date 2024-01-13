---
title: Rust Macros
date: 2023-07-18 14:38:01
categories:
  - - 计算机
tags:
  - RSX
---


> Write code that writes code - meta programming

Match an expression and perform some operation

# Declarative Macros

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

# Procedural Macros

## Derive Macros

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

## Function Like Macro

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

## Attribute Like Macro

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

# Example

Build a 
