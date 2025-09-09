# Rust Cheat Sheet

Comprehensive reference for Rust programming language covering ownership, borrowing, traits, error handling, and essential features for safe systems programming.

---

## Table of Contents
- [Basic Syntax](#basic-syntax)
- [Variables and Mutability](#variables-and-mutability)
- [Data Types](#data-types)
- [Ownership and Borrowing](#ownership-and-borrowing)
- [Structs and Enums](#structs-and-enums)
- [Pattern Matching](#pattern-matching)
- [Error Handling](#error-handling)
- [Traits](#traits)
- [Collections](#collections)
- [Concurrency](#concurrency)
- [Modules and Crates](#modules-and-crates)
- [Memory Management](#memory-management)

---

## Basic Syntax

| Feature | Syntax | Example |
|---------|--------|---------|
| Function | `fn name(params) -> ReturnType` | `fn add(a: i32, b: i32) -> i32` |
| Variable | `let name = value;` | `let x = 5;` |
| Mutable Variable | `let mut name = value;` | `let mut y = 10;` |
| Constant | `const NAME: Type = value;` | `const PI: f64 = 3.14159;` |
| Comments | `// single line` or `/* multi-line */` | `// This is a comment` |
| Macro | `macro_name!()` | `println!("Hello");` |

### Basic Program Structure
```rust
fn main() {
    println!("Hello, world!");
}
```

## Variables and Mutability

### Variable Declaration
```rust
// Immutable by default
let x = 5;
// x = 6; // Error! Cannot mutate immutable variable

// Mutable variable
let mut y = 5;
y = 6; // OK

// Type annotation
let z: i32 = 10;

// Multiple assignment
let (a, b) = (1, 2);
```

### Shadowing
```rust
let x = 5;
let x = x + 1;        // Shadow previous x
let x = x * 2;        // Shadow again, x is now 12

// Can change type with shadowing
let spaces = "   ";
let spaces = spaces.len(); // Now spaces is usize
```

### Constants
```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;

// Constants can be declared in any scope
const MAX_POINTS: u32 = 100_000;
```

## Data Types

### Scalar Types
| Type | Size | Range | Example |
|------|------|-------|---------|
| `i8` | 8 bits | -128 to 127 | `let a: i8 = -42;` |
| `i16` | 16 bits | -32,768 to 32,767 | `let b: i16 = 1000;` |
| `i32` | 32 bits | -2^31 to 2^31-1 | `let c = 42;` |
| `i64` | 64 bits | -2^63 to 2^63-1 | `let d: i64 = 1000;` |
| `i128` | 128 bits | Very large range | `let e: i128 = 1000;` |
| `isize` | arch | Pointer size | `let f: isize = 1000;` |
| `u8` | 8 bits | 0 to 255 | `let g: u8 = 255;` |
| `u16`, `u32`, `u64`, `u128`, `usize` | Various | Unsigned ranges | Similar patterns |
| `f32` | 32 bits | IEEE 754 | `let h: f32 = 3.14;` |
| `f64` | 64 bits | IEEE 754 | `let i = 3.14;` |
| `bool` | 1 byte | true/false | `let j = true;` |
| `char` | 4 bytes | Unicode scalar | `let k = '🦀';` |

### Compound Types
```rust
// Tuples
let tup: (i32, f64, u8) = (500, 6.4, 1);
let (x, y, z) = tup; // Destructuring
let first = tup.0;   // Access by index

// Arrays (fixed size)
let arr: [i32; 5] = [1, 2, 3, 4, 5];
let arr2 = [3; 5];   // [3, 3, 3, 3, 3]
let first = arr[0];  // Access by index
```

### String Types
```rust
// String slice (&str) - immutable reference
let s: &str = "hello";

// String - owned, mutable, heap-allocated
let mut s = String::new();
s.push_str("hello");
s.push(' ');
s.push('w');

// Convert between types
let s1 = "hello".to_string();
let s2 = String::from("hello");
let s3: &str = &s1;
```

## Ownership and Borrowing

### Ownership Rules
1. Each value in Rust has a variable that's called its owner
2. There can only be one owner at a time
3. When the owner goes out of scope, the value will be dropped

### Move Semantics
```rust
let s1 = String::from("hello");
let s2 = s1; // s1 is moved to s2, s1 is no longer valid
// println!("{}", s1); // Error! s1 is no longer valid

// Clone if you need a copy
let s1 = String::from("hello");
let s2 = s1.clone();
println!("{} {}", s1, s2); // OK, both are valid
```

### Borrowing
```rust
// Immutable borrowing
fn calculate_length(s: &String) -> usize {
    s.len()
} // s goes out of scope but nothing happens (it's a reference)

let s1 = String::from("hello");
let len = calculate_length(&s1); // Borrow s1
println!("The length of '{}' is {}.", s1, len); // s1 still valid

// Mutable borrowing
fn change(s: &mut String) {
    s.push_str(", world");
}

let mut s = String::from("hello");
change(&mut s);
println!("{}", s); // "hello, world"
```

### Borrowing Rules
```rust
let mut s = String::from("hello");

// Rule 1: Can have multiple immutable references OR one mutable reference
let r1 = &s;
let r2 = &s;
println!("{} and {}", r1, r2); // OK - multiple immutable refs

// let r3 = &mut s; // Error! Cannot have mutable ref while immutable refs exist

// Rule 2: References must always be valid (no dangling pointers)
// This function would not compile:
/*
fn dangle() -> &String {
    let s = String::from("hello");
    &s // Error! s goes out of scope, leaving dangling reference
}
*/
```

### Lifetimes
```rust
// Explicit lifetime parameters
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// Lifetime in structs
struct ImportantExcerpt<'a> {
    part: &'a str,
}

impl<'a> ImportantExcerpt<'a> {
    fn level(&self) -> i32 {
        3
    }
}
```

## Structs and Enums

### Structs
```rust
// Classic struct
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}

// Create instance
let mut user1 = User {
    email: String::from("someone@example.com"),
    username: String::from("someusername123"),
    active: true,
    sign_in_count: 1,
};

// Update syntax
let user2 = User {
    email: String::from("another@example.com"),
    ..user1 // Use remaining fields from user1
};

// Tuple struct
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point(0, 0, 0);

// Unit struct
struct AlwaysEqual;
let subject = AlwaysEqual;
```

### Implementing Methods
```rust
impl User {
    // Associated function (like static method)
    fn new(email: String, username: String) -> User {
        User {
            email,
            username,
            active: true,
            sign_in_count: 1,
        }
    }
    
    // Method
    fn is_active(&self) -> bool {
        self.active
    }
    
    // Mutable method
    fn deactivate(&mut self) {
        self.active = false;
    }
    
    // Method that takes ownership
    fn into_email(self) -> String {
        self.email
    }
}

// Usage
let mut user = User::new(
    String::from("test@example.com"),
    String::from("testuser")
);
println!("{}", user.is_active());
user.deactivate();
```

### Enums
```rust
// Basic enum
enum IpAddrKind {
    V4,
    V6,
}

// Enum with data
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

let home = IpAddr::V4(127, 0, 0, 1);
let loopback = IpAddr::V6(String::from("::1"));

// Complex enum
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

impl Message {
    fn call(&self) {
        // Method body
    }
}

let m = Message::Write(String::from("hello"));
m.call();
```

### Option Enum
```rust
// Option is defined as:
// enum Option<T> {
//     None,
//     Some(T),
// }

let some_number = Some(5);
let some_string = Some("a string");
let absent_number: Option<i32> = None;

// Working with Option
match some_number {
    Some(i) => println!("Got a number: {}", i),
    None => println!("Got nothing"),
}

// Convenience methods
let x: Option<i32> = Some(2);
assert_eq!(x.is_some(), true);
assert_eq!(x.unwrap(), 2);
assert_eq!(x.unwrap_or(0), 2);
```

## Pattern Matching

### Match Expression
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => {
            println!("Lucky penny!");
            1
        }
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}

// Match with Option
fn plus_one(x: Option<i32>) -> Option<i32> {
    match x {
        None => None,
        Some(i) => Some(i + 1),
    }
}

// Catch-all pattern
let dice_roll = 9;
match dice_roll {
    3 => add_fancy_hat(),
    7 => remove_fancy_hat(),
    other => move_player(other), // Bind to variable
    // _ => (), // Ignore value
}
```

### If Let
```rust
let config_max = Some(3u8);

// Instead of:
match config_max {
    Some(max) => println!("The maximum is configured to be {}", max),
    _ => (),
}

// Use if let:
if let Some(max) = config_max {
    println!("The maximum is configured to be {}", max);
}

// With else
let mut count = 0;
if let Coin::Quarter = coin {
    println!("State quarter!");
} else {
    count += 1;
}
```

### Pattern Syntax
```rust
// Literal patterns
match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}

// Range patterns
match x {
    1..=5 => println!("one through five"),
    _ => println!("something else"),
}

// Destructuring
struct Point {
    x: i32,
    y: i32,
}

let p = Point { x: 0, y: 7 };
match p {
    Point { x, y: 0 } => println!("On the x axis at {}", x),
    Point { x: 0, y } => println!("On the y axis at {}", y),
    Point { x, y } => println!("On neither axis: ({}, {})", x, y),
}

// Guards
let num = Some(4);
match num {
    Some(x) if x < 5 => println!("less than five: {}", x),
    Some(x) => println!("{}", x),
    None => (),
}
```

## Error Handling

### Result Enum
```rust
// Result is defined as:
// enum Result<T, E> {
//     Ok(T),
//     Err(E),
// }

use std::fs::File;
use std::io::ErrorKind;

// Basic error handling
let greeting_file_result = File::open("hello.txt");
let greeting_file = match greeting_file_result {
    Ok(file) => file,
    Err(error) => match error.kind() {
        ErrorKind::NotFound => match File::create("hello.txt") {
            Ok(fc) => fc,
            Err(e) => panic!("Problem creating the file: {:?}", e),
        },
        other_error => {
            panic!("Problem opening the file: {:?}", other_error);
        }
    },
};
```

### Shortcuts: unwrap and expect
```rust
// unwrap: panic on error
let greeting_file = File::open("hello.txt").unwrap();

// expect: panic with custom message
let greeting_file = File::open("hello.txt")
    .expect("hello.txt should be included in this project");
```

### Propagating Errors
```rust
use std::fs::File;
use std::io::{self, Read};

// Manual propagation
fn read_username_from_file() -> Result<String, io::Error> {
    let username_file_result = File::open("hello.txt");

    let mut username_file = match username_file_result {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut username = String::new();
    match username_file.read_to_string(&mut username) {
        Ok(_) => Ok(username),
        Err(e) => Err(e),
    }
}

// Using ? operator
fn read_username_from_file_short() -> Result<String, io::Error> {
    let mut username_file = File::open("hello.txt")?;
    let mut username = String::new();
    username_file.read_to_string(&mut username)?;
    Ok(username)
}

// Even shorter
fn read_username_from_file_shorter() -> Result<String, io::Error> {
    let mut username = String::new();
    File::open("hello.txt")?.read_to_string(&mut username)?;
    Ok(username)
}
```

### Custom Error Types
```rust
use std::fmt;

#[derive(Debug)]
struct CustomError {
    details: String,
}

impl CustomError {
    fn new(msg: &str) -> CustomError {
        CustomError {
            details: msg.to_string(),
        }
    }
}

impl fmt::Display for CustomError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{}", self.details)
    }
}

impl std::error::Error for CustomError {
    fn description(&self) -> &str {
        &self.details
    }
}
```

## Traits

### Defining and Implementing Traits
```rust
// Define a trait
trait Summary {
    fn summarize(&self) -> String;
    
    // Default implementation
    fn summarize_default(&self) -> String {
        String::from("(Read more...)")
    }
}

struct NewsArticle {
    pub headline: String,
    pub location: String,
    pub author: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}

struct Tweet {
    pub username: String,
    pub content: String,
    pub reply: bool,
    pub retweet: bool,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("{}: {}", self.username, self.content)
    }
}
```

### Traits as Parameters
```rust
// Trait bound syntax
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}

// impl Trait syntax (sugar for above)
pub fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// Multiple trait bounds
pub fn notify(item: &(impl Summary + Display)) {
    // ...
}

// Where clauses for readability
fn some_function<T, U>(t: &T, u: &U) -> i32
where
    T: Display + Clone,
    U: Clone + Debug,
{
    // ...
}
```

### Returning Traits
```rust
// Return impl Trait
fn returns_summarizable() -> impl Summary {
    Tweet {
        username: String::from("horse_ebooks"),
        content: String::from("of course, as you probably already know, people"),
        reply: false,
        retweet: false,
    }
}
```

### Trait Objects
```rust
// Dynamic dispatch
trait Draw {
    fn draw(&self);
}

struct Screen {
    pub components: Vec<Box<dyn Draw>>,
}

impl Screen {
    pub fn run(&self) {
        for component in self.components.iter() {
            component.draw();
        }
    }
}

struct Button {
    pub width: u32,
    pub height: u32,
    pub label: String,
}

impl Draw for Button {
    fn draw(&self) {
        // Draw button
    }
}
```

### Associated Types
```rust
trait Iterator {
    type Item; // Associated type
    
    fn next(&mut self) -> Option<Self::Item>;
}

struct Counter {
    current: usize,
    max: usize,
}

impl Iterator for Counter {
    type Item = usize;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            let current = self.current;
            self.current += 1;
            Some(current)
        } else {
            None
        }
    }
}
```

## Collections

### Vectors
```rust
// Create vector
let mut v: Vec<i32> = Vec::new();
let v2 = vec![1, 2, 3]; // vec! macro

// Add elements
v.push(5);
v.push(6);
v.push(7);

// Access elements
let third = &v[2];                    // Panics if out of bounds
let third = v.get(2);                 // Returns Option<&T>

match v.get(2) {
    Some(third) => println!("The third element is {}", third),
    None => println!("There is no third element."),
}

// Iterate
for i in &v {
    println!("{}", i);
}

// Mutable iteration
for i in &mut v {
    *i += 50;
}

// Multiple types with enum
enum SpreadsheetCell {
    Int(i32),
    Float(f64),
    Text(String),
}

let row = vec![
    SpreadsheetCell::Int(3),
    SpreadsheetCell::Text(String::from("blue")),
    SpreadsheetCell::Float(10.12),
];
```

### Strings
```rust
// Creating strings
let mut s = String::new();
let s2 = "initial contents".to_string();
let s3 = String::from("initial contents");

// Updating strings
s.push_str("bar");
s.push('l');

// Concatenation
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1 is moved and no longer valid

// format! macro
let s4 = format!("{}-{}", s2, "suffix");

// Indexing (not supported directly)
// let h = s2[0]; // Error!

// Slicing (be careful with UTF-8)
let hello = "Здравствуйте";
let s = &hello[0..4]; // "Зд" (4 bytes)

// Iterating over characters
for c in "नमस्ते".chars() {
    println!("{}", c);
}

// Iterating over bytes
for b in "नमस्ते".bytes() {
    println!("{}", b);
}
```

### Hash Maps
```rust
use std::collections::HashMap;

// Create hash map
let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Yellow"), 50);

// From vectors
let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];
let scores: HashMap<_, _> = teams.into_iter().zip(initial_scores.into_iter()).collect();

// Access values
let team_name = String::from("Blue");
let score = scores.get(&team_name); // Returns Option<&V>

// Iterate
for (key, value) in &scores {
    println!("{}: {}", key, value);
}

// Update values
scores.insert(String::from("Blue"), 25); // Overwrite

// Insert if key has no value
scores.entry(String::from("Red")).or_insert(50);

// Update based on old value
let text = "hello world wonderful world";
let mut map = HashMap::new();
for word in text.split_whitespace() {
    let count = map.entry(word).or_insert(0);
    *count += 1;
}
```

## Concurrency

### Threads
```rust
use std::thread;
use std::time::Duration;

// Spawn thread
let handle = thread::spawn(|| {
    for i in 1..10 {
        println!("hi number {} from the spawned thread!", i);
        thread::sleep(Duration::from_millis(1));
    }
});

// Wait for thread to finish
handle.join().unwrap();

// Move closure
let v = vec![1, 2, 3];
let handle = thread::spawn(move || {
    println!("Here's a vector: {:?}", v);
});
handle.join().unwrap();
```

### Message Passing
```rust
use std::sync::mpsc;

// Create channel
let (tx, rx) = mpsc::channel();

thread::spawn(move || {
    let val = String::from("hi");
    tx.send(val).unwrap();
    // val is no longer valid here
});

// Receive message
let received = rx.recv().unwrap();
println!("Got: {}", received);

// Multiple producers
let (tx, rx) = mpsc::channel();
let tx1 = tx.clone();

thread::spawn(move || {
    tx.send(String::from("hi from thread")).unwrap();
});

thread::spawn(move || {
    tx1.send(String::from("hello from another thread")).unwrap();
});

// Receive multiple messages
for received in rx {
    println!("Got: {}", received);
}
```

### Shared State
```rust
use std::sync::{Arc, Mutex};

// Mutex (mutual exclusion)
let m = Mutex::new(5);

{
    let mut num = m.lock().unwrap();
    *num = 6;
} // Lock is released when num goes out of scope

// Arc (atomically reference counted) for multiple ownership
let counter = Arc::new(Mutex::new(0));
let mut handles = vec![];

for _ in 0..10 {
    let counter = Arc::clone(&counter);
    let handle = thread::spawn(move || {
        let mut num = counter.lock().unwrap();
        *num += 1;
    });
    handles.push(handle);
}

for handle in handles {
    handle.join().unwrap();
}

println!("Result: {}", *counter.lock().unwrap());
```

### Sync and Send Traits
```rust
// Send trait: ownership can be transferred between threads
// Sync trait: safe to reference from multiple threads

// Almost all primitive types are Send and Sync
// Raw pointers are neither Send nor Sync
// Rc<T> is not Send or Sync
// RefCell<T> is Send but not Sync
// Mutex<T> is both Send and Sync
```

## Modules and Crates

### Module System
```rust
// src/main.rs or src/lib.rs
mod garden; // Look for src/garden.rs or src/garden/mod.rs

// Nested modules
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        
        fn seat_at_table() {}
    }
    
    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}

// Use keyword
use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    
    // Absolute path
    crate::front_of_house::hosting::add_to_waitlist();
    
    // Relative path
    front_of_house::hosting::add_to_waitlist();
}
```

### Re-exporting
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

// Re-export
pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}
```

### Use Patterns
```rust
// Multiple items
use std::collections::{HashMap, BTreeMap, HashSet};

// Glob operator
use std::collections::*;

// Rename with as
use std::fmt::Result;
use std::io::Result as IoResult;

// External crates (add to Cargo.toml)
use rand::Rng;

fn main() {
    let secret_number = rand::thread_rng().gen_range(1..=100);
}
```

### Cargo.toml
```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8"
serde = { version = "1.0", features = ["derive"] }

[dev-dependencies]
pretty_assertions = "1"
```

## Memory Management

### Stack vs Heap
```rust
// Stack: fixed size, LIFO, fast
let x = 5;           // i32 on stack
let y = [1, 2, 3];   // Array on stack

// Heap: dynamic size, managed by allocator, slower
let s = String::from("hello"); // String data on heap
let v = vec![1, 2, 3];         // Vector data on heap
```

### Smart Pointers
```rust
use std::rc::Rc;
use std::cell::RefCell;

// Box<T> - heap allocation
let b = Box::new(5);
println!("b = {}", b);

// Rc<T> - reference counting (single-threaded)
let a = Rc::new(5);
let b = Rc::clone(&a);
println!("Reference count: {}", Rc::strong_count(&a)); // 2

// RefCell<T> - interior mutability
let data = RefCell::new(5);
let mut borrowed = data.borrow_mut();
*borrowed = 10;

// Combining Rc and RefCell
let shared_data = Rc::new(RefCell::new(vec![1, 2, 3]));
let a = Rc::clone(&shared_data);
let b = Rc::clone(&shared_data);

a.borrow_mut().push(4);
println!("{:?}", b.borrow()); // [1, 2, 3, 4]
```

### Memory Safety
```rust
// No null pointer dereferences - use Option<T>
// No buffer overflows - bounds checking
// No use after free - ownership system
// No double free - ownership system
// No memory leaks - automatic cleanup (mostly)

// Example: preventing use after free
let mut s = String::from("hello");
let r1 = &s;
let r2 = &s;
// let r3 = &mut s; // Error! Cannot borrow as mutable while immutable refs exist
println!("{} {}", r1, r2);

let r3 = &mut s; // OK, r1 and r2 are no longer used
r3.push_str(" world");
```

---

## Resources
- [The Rust Programming Language Book](https://doc.rust-lang.org/book/)
- [Rust by Example](https://doc.rust-lang.org/rust-by-example/)
- [The Rustonomicon](https://doc.rust-lang.org/nomicon/)
- [Rust Standard Library](https://doc.rust-lang.org/std/)
- [Crates.io](https://crates.io/)

---
*Originally compiled from various sources. Contributions welcome!*