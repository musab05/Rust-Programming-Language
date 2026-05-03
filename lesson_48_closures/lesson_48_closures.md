# 📘 Lesson 48 — Closures: Syntax & Captures (CL1)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** CL1 · Category: 🔒 Closures  
> **Previous:** [Lesson 47 — Publishing to crates.io](../lesson_47_publishing/lesson_47_publishing.md)  
> **Next:** [Lesson 49 — Fn, FnMut, FnOnce](../lesson_49_fn_traits/lesson_49_fn_traits.md)  
> **Practice:** [Questions](./lesson_48_questions.md) · [Answers](./lesson_48_answers.md)  
> **Practice Task:** Build a configurable filter using closures

---

## Table of Contents

1. [What Are Closures?](#1-what-are-closures)
2. [Closure Syntax](#2-closure-syntax)
3. [Type Inference](#3-type-inference)
4. [Capturing Variables](#4-capturing-variables)
5. [Three Capture Modes](#5-three-capture-modes)
6. [move Closures](#6-move-closures)
7. [Closures vs Functions](#7-closures-vs-functions)
8. [Closures as Arguments](#8-closures-as-arguments)
9. [Real-World Example: Configurable Filter](#9-real-world-example-configurable-filter)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Are Closures?

Closures are **anonymous functions** that can **capture** variables from their enclosing scope:

```rust
fn main() {
    let name = String::from("Rust");

    // A closure that captures `name`
    let greet = || println!("Hello, {name}!");

    greet();  // Hello, Rust!
    greet();  // can call multiple times

    // Regular functions CAN'T capture:
    // fn greet_fn() { println!("{name}"); }  // ❌ can't find `name`
}
```

---

## 2. Closure Syntax

```rust
fn main() {
    // Full syntax
    let add = |a: i32, b: i32| -> i32 { a + b };

    // Type inference (most common)
    let add = |a, b| a + b;

    // No parameters
    let say_hi = || println!("Hi!");

    // Multi-line body
    let complex = |x: i32| {
        let doubled = x * 2;
        let tripled = x * 3;
        doubled + tripled
    };

    println!("{}", add(3, 4));      // 7
    say_hi();                        // Hi!
    println!("{}", complex(10));     // 50
}
```

### Comparison with functions:

```rust
// Function
fn add_fn(a: i32, b: i32) -> i32 { a + b }

// Closure (equivalent)
let add_cl = |a: i32, b: i32| -> i32 { a + b };

// Closure with inference
let add_cl = |a, b| a + b;
```

---

## 3. Type Inference

Closures infer parameter and return types from their first use:

```rust
fn main() {
    let double = |x| x * 2;

    // First call fixes the type
    println!("{}", double(5));     // i32 inferred

    // ❌ Can't use different type after inference:
    // println!("{}", double(5.0));  // error: expected i32, found f64
}
```

---

## 4. Capturing Variables

Closures automatically capture variables from the surrounding scope:

```rust
fn main() {
    let x = 10;
    let y = 20;

    // Captures x and y by reference (borrows them)
    let sum = || x + y;
    println!("{}", sum());  // 30

    // x and y still available — only borrowed
    println!("x={x}, y={y}");

    // Captures a mutable reference
    let mut count = 0;
    let mut increment = || {
        count += 1;
        println!("Count: {count}");
    };
    increment();  // Count: 1
    increment();  // Count: 2
    // count is mutably borrowed while `increment` exists

    // After the closure is dropped, count is available again
    drop(increment);
    println!("Final count: {count}");  // 2
}
```

---

## 5. Three Capture Modes

Rust closures capture variables in the **least restrictive** way possible:

### 1. Borrow (`&T`) — default when only reading:

```rust
fn main() {
    let message = String::from("Hello");
    let print_msg = || println!("{message}");  // borrows &message

    print_msg();
    print_msg();
    println!("Still have: {message}");  // ✅ message not moved
}
```

### 2. Mutable borrow (`&mut T`) — when modifying:

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    let mut push_item = || list.push(4);  // borrows &mut list

    push_item();
    // println!("{:?}", list);  // ❌ can't use while mutably borrowed
    drop(push_item);
    println!("{:?}", list);  // ✅ [1, 2, 3, 4]
}
```

### 3. Move (`T`) — when consuming:

```rust
fn main() {
    let name = String::from("Alice");
    let consume = || {
        let _owned = name;  // moves name into the closure
        println!("Consumed!");
    };

    consume();
    // println!("{name}");  // ❌ name was moved
    // consume();           // ❌ can only call once (FnOnce)
}
```

### Summary:

| Capture Mode | When | Closure can be called |
|---|---|---|
| `&T` (borrow) | Only reads the variable | Multiple times |
| `&mut T` (mut borrow) | Modifies the variable | Multiple times |
| `T` (move/consume) | Takes ownership | Once (FnOnce) |

---

## 6. move Closures

Force a closure to take ownership of all captured variables:

```rust
fn main() {
    let name = String::from("Alice");

    // Without move — borrows name
    let greet = || println!("Hello, {name}");
    greet();
    println!("Still have: {name}");  // ✅

    // With move — takes ownership
    let name2 = String::from("Bob");
    let greet_owned = move || println!("Hello, {name2}");
    greet_owned();
    // println!("{name2}");  // ❌ name2 was moved into the closure
}
```

### Essential for threads:

```rust
use std::thread;

fn main() {
    let data = vec![1, 2, 3];

    // Must use move — thread might outlive current scope
    let handle = thread::spawn(move || {
        println!("Thread got: {:?}", data);
    });

    // println!("{:?}", data);  // ❌ data was moved to the thread
    handle.join().unwrap();
}
```

### move with Copy types:

```rust
fn main() {
    let x = 42;  // i32 is Copy

    let closure = move || println!("{x}");
    closure();

    // x is still available — move copies it for Copy types
    println!("Original: {x}");  // ✅
}
```

---

## 7. Closures vs Functions

| Feature | Functions | Closures |
|---|---|---|
| Capture environment | ❌ No | ✅ Yes |
| Name | Required | Anonymous (can be bound to variable) |
| Type inference | ❌ Must annotate | ✅ Inferred |
| Can be generic | ✅ Yes | ❌ Each closure has a unique type |
| Pass as argument | By function pointer `fn(T) -> U` | By trait `Fn`, `FnMut`, `FnOnce` |

```rust
fn main() {
    // Function pointer — can't capture anything
    fn double(x: i32) -> i32 { x * 2 }

    // Closure — can capture
    let multiplier = 3;
    let triple = |x| x * multiplier;

    println!("{}", double(5));   // 10
    println!("{}", triple(5));   // 15
}
```

---

## 8. Closures as Arguments

```rust
fn apply(value: i32, f: impl Fn(i32) -> i32) -> i32 {
    f(value)
}

fn apply_twice(value: i32, f: impl Fn(i32) -> i32) -> i32 {
    f(f(value))
}

fn main() {
    let result = apply(5, |x| x * 2);
    println!("{result}");  // 10

    let result = apply_twice(3, |x| x + 10);
    println!("{result}");  // 23

    // Can also pass regular functions
    fn square(x: i32) -> i32 { x * x }
    let result = apply(4, square);
    println!("{result}");  // 16

    // Closure that captures
    let offset = 100;
    let result = apply(5, |x| x + offset);
    println!("{result}");  // 105
}
```

---

## 9. Real-World Example: Configurable Filter

The roadmap practice task:

```rust
fn filter_items<T, F>(items: &[T], predicate: F) -> Vec<&T>
where
    F: Fn(&T) -> bool,
{
    items.iter().filter(|item| predicate(item)).collect()
}

fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // Different filters using closures
    let evens = filter_items(&numbers, |x| x % 2 == 0);
    println!("Evens: {:?}", evens);

    let threshold = 5;
    let above = filter_items(&numbers, |x| **x > threshold);
    println!("Above {threshold}: {:?}", above);

    // Strings
    let words = vec!["hello", "world", "hi", "rust", "hey"];
    let short = filter_items(&words, |w| w.len() <= 3);
    println!("Short: {:?}", short);

    let starts_with_h = filter_items(&words, |w| w.starts_with('h'));
    println!("Starts with 'h': {:?}", starts_with_h);

    // Composing filters
    let data = vec![
        ("Alice", 85), ("Bob", 92), ("Charlie", 78),
        ("Diana", 95), ("Eve", 65),
    ];

    let passing_grade = 80;
    let honor_roll = filter_items(&data, |(_, score)| *score >= passing_grade);
    println!("Honor roll: {:?}", honor_roll);
}
```

---

## 10. Summary Cheat Sheet

```
CLOSURE SYNTAX
────────────────────────────────────────────────────────────
|x| x + 1                  one param, expression body
|a, b| a + b               two params
|| println!("hi")          no params
|x: i32| -> i32 { x * 2 }  explicit types

CAPTURE MODES (automatic)
────────────────────────────────────────────────────────────
&T        borrow       reads value         reusable
&mut T    mut borrow   modifies value      reusable
T         move         takes ownership     one-time (FnOnce)

MOVE KEYWORD
────────────────────────────────────────────────────────────
let c = move || { ... };   force ownership transfer
Essential for threads and returning closures
Copy types are copied, not moved

AS ARGUMENTS
────────────────────────────────────────────────────────────
fn f(cb: impl Fn(i32) -> i32)       takes a closure
fn f<F: Fn(i32)>(cb: F)             generic form
fn f(cb: &dyn Fn(i32))              dynamic dispatch

KEY DIFFERENCES FROM FUNCTIONS
────────────────────────────────────────────────────────────
✅ Capture variables from enclosing scope
✅ Type inference for parameters
❌ Each closure has a unique anonymous type
```

---

## What's Next?

**Lesson 49 — Fn, FnMut, FnOnce** — The three closure traits. Understand when each is needed, how they relate to capture modes, and how to accept closures as parameters.

## Further Reading
- [The Rust Book — Ch 13.1: Closures](https://doc.rust-lang.org/book/ch13-01-closures.html)
- [Rust by Example — Closures](https://doc.rust-lang.org/rust-by-example/fn/closures.html)

---

*Closures: functions with memory! 🦀*
