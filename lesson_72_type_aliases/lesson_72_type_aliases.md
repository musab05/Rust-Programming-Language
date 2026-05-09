# 📘 Lesson 72 — Type Aliases (AT1)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** AT1 · Category: 🔷 Advanced Types  
> **Previous:** [Lesson 71 — Weak\<T\>](../lesson_71_weak/lesson_71_weak.md)  
> **Next:** [Lesson 73 — Newtype Pattern](../lesson_73_newtype/lesson_73_newtype.md)  
> **Practice:** [Questions](./lesson_72_questions.md) · [Answers](./lesson_72_answers.md)  
> **Practice Task:** Create type aliases for a complex closure signature

---

## Table of Contents

1. [What Are Type Aliases?](#1-what-are-type-aliases)
2. [Basic Syntax](#2-basic-syntax)
3. [Simplifying Complex Types](#3-simplifying-complex-types)
4. [Type Aliases with Generics](#4-type-aliases-with-generics)
5. [The Result Alias Pattern](#5-the-result-alias-pattern)
6. [Type Aliases vs Newtypes](#6-type-aliases-vs-newtypes)
7. [Common Standard Library Aliases](#7-common-standard-library-aliases)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. What Are Type Aliases?

`type` creates an alternative name for an existing type. **No new type is created** — it's purely cosmetic:

```rust
type Meters = f64;
type Seconds = f64;

fn speed(distance: Meters, time: Seconds) -> f64 {
    distance / time  // ⚠️ No type safety — both are just f64
}

fn main() {
    let d: Meters = 100.0;
    let t: Seconds = 9.58;
    println!("Speed: {:.2} m/s", speed(d, t));

    // ⚠️ This compiles — aliases are interchangeable!
    let wrong: Meters = t;  // no error!
    println!("{wrong}");
}
```

---

## 2. Basic Syntax

```rust
// Simple aliases
type Kilometers = i32;
type Username = String;
type UserId = u64;

// More descriptive signatures
fn find_user(id: UserId) -> Option<Username> {
    if id == 1 { Some("Alice".to_string()) }
    else { None }
}

fn main() {
    let id: UserId = 1;
    if let Some(name) = find_user(id) {
        println!("User {id}: {name}");
    }
}
```

---

## 3. Simplifying Complex Types

The main power of aliases — taming long generic types:

```rust
use std::collections::HashMap;

// Without alias — verbose and repetitive
fn process_data(data: &HashMap<String, Vec<(i32, String, bool)>>) {
    for (key, values) in data {
        println!("{key}: {} items", values.len());
    }
}

// With alias — clean!
type DataMap = HashMap<String, Vec<(i32, String, bool)>>;

fn process_data_clean(data: &DataMap) {
    for (key, values) in data {
        println!("{key}: {} items", values.len());
    }
}

fn main() {
    let mut data: DataMap = HashMap::new();
    data.insert("users".into(), vec![(1, "Alice".into(), true)]);
    process_data_clean(&data);
}
```

### Closure type aliases:

```rust
// Complex closure signature — hard to read
fn apply(f: Box<dyn Fn(i32, i32) -> Result<i32, String>>, a: i32, b: i32) {
    match f(a, b) {
        Ok(result) => println!("Result: {result}"),
        Err(e) => println!("Error: {e}"),
    }
}

// Much cleaner with an alias
type MathOp = Box<dyn Fn(i32, i32) -> Result<i32, String>>;

fn apply_clean(f: &MathOp, a: i32, b: i32) {
    match f(a, b) {
        Ok(result) => println!("Result: {result}"),
        Err(e) => println!("Error: {e}"),
    }
}

fn main() {
    let add: MathOp = Box::new(|a, b| Ok(a + b));
    let div: MathOp = Box::new(|a, b| {
        if b == 0 { Err("division by zero".into()) } else { Ok(a / b) }
    });

    apply_clean(&add, 10, 5);  // Result: 15
    apply_clean(&div, 10, 0);  // Error: division by zero
}
```

---

## 4. Type Aliases with Generics

```rust
use std::collections::HashMap;

// Partial alias — still generic
type StringMap<V> = HashMap<String, V>;

fn main() {
    let mut users: StringMap<u32> = HashMap::new();
    users.insert("Alice".into(), 30);
    users.insert("Bob".into(), 25);

    let mut config: StringMap<String> = HashMap::new();
    config.insert("host".into(), "localhost".into());
    config.insert("port".into(), "8080".into());

    println!("Users: {:?}", users);
    println!("Config: {:?}", config);
}
```

### Function pointer aliases:

```rust
type Transformer<T> = fn(T) -> T;

fn apply_twice<T: Copy>(f: Transformer<T>, value: T) -> T {
    f(f(value))
}

fn double(x: i32) -> i32 { x * 2 }
fn negate(x: i32) -> i32 { -x }

fn main() {
    println!("{}", apply_twice(double, 3));   // 12 (3 → 6 → 12)
    println!("{}", apply_twice(negate, 5));   // 5  (5 → -5 → 5)
}
```

---

## 5. The Result Alias Pattern

The most common pattern in the standard library:

```rust
// std::io defines:
// type Result<T> = std::result::Result<T, std::io::Error>;

// So instead of writing:
fn read_file(path: &str) -> std::result::Result<String, std::io::Error> {
    std::fs::read_to_string(path)
}

// You write:
fn read_file_clean(path: &str) -> std::io::Result<String> {
    std::fs::read_to_string(path)
}
```

### Create your own Result alias:

```rust
#[derive(Debug)]
enum AppError {
    NotFound(String),
    ParseError(String),
    IoError(std::io::Error),
}

type AppResult<T> = Result<T, AppError>;

fn find_user(id: u32) -> AppResult<String> {
    if id == 0 { Err(AppError::NotFound("User 0 not found".into())) }
    else { Ok(format!("User_{id}")) }
}

fn parse_id(s: &str) -> AppResult<u32> {
    s.parse().map_err(|_| AppError::ParseError(format!("Invalid ID: {s}")))
}

fn main() {
    match find_user(1) {
        Ok(name) => println!("Found: {name}"),
        Err(e) => println!("Error: {e:?}"),
    }
}
```

---

## 6. Type Aliases vs Newtypes

| | Type Alias | Newtype |
|---|---|---|
| Syntax | `type X = Y` | `struct X(Y)` |
| New type? | ❌ Same type | ✅ Distinct type |
| Type safety | ❌ Interchangeable | ✅ Compile-time check |
| Runtime cost | Zero | Zero |
| Use case | Readability | Safety |

```rust
// Alias — no safety
type Meters = f64;
type Seconds = f64;
let m: Meters = 5.0;
let s: Seconds = m;  // ✅ compiles — both are f64

// Newtype — type safe
struct MetersNT(f64);
struct SecondsNT(f64);
let m = MetersNT(5.0);
// let s: SecondsNT = m;  // ❌ compile error — different types!
```

**Rule of thumb:** Use aliases for readability, newtypes for safety.

---

## 7. Common Standard Library Aliases

```rust
// std::io::Result<T> = Result<T, io::Error>
// std::fmt::Result    = Result<(), fmt::Error>
// std::thread::Result<T> = Result<T, Box<dyn Any + Send>>

use std::io;
use std::fmt;

fn read_something() -> io::Result<String> {
    Ok("data".into())
}

struct MyType;
impl fmt::Display for MyType {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "MyType")
    }
}
```

---

## 8. Summary Cheat Sheet

```
BASIC ALIAS
────────────────────────────────────────────────────────────
type Name = ExistingType;
type UserId = u64;

COMPLEX TYPE SIMPLIFICATION
────────────────────────────────────────────────────────────
type DataMap = HashMap<String, Vec<(i32, String)>>;
type Callback = Box<dyn Fn(i32) -> Result<i32, String>>;

GENERIC ALIAS
────────────────────────────────────────────────────────────
type StringMap<V> = HashMap<String, V>;
type Transformer<T> = fn(T) -> T;

RESULT ALIAS PATTERN
────────────────────────────────────────────────────────────
type AppResult<T> = Result<T, AppError>;

KEY RULE
────────────────────────────────────────────────────────────
Aliases are COSMETIC — same type underneath
For type safety → use newtypes (struct Wrapper(Inner))
```

---

## What's Next?

**Lesson 73 — Newtype Pattern** — Create distinct types that prevent mixing units, enforce validation, and bypass the orphan rule.

## Further Reading
- [The Rust Book — Ch 19.4: Advanced Types](https://doc.rust-lang.org/book/ch19-04-advanced-types.html)

---

*Type aliases: readability without ceremony! 🦀*
