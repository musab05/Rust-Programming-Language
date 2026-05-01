# 📘 Lesson 40 — Generics in Functions (T3)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T3 · Category: 🔷 Traits  
> **Previous:** [Lesson 39 — Trait Bounds](../lesson_39_trait_bounds/lesson_39_trait_bounds.md)  
> **Next:** Lesson 41 — Generics in Structs & Enums *(coming soon)*  
> **Practice:** [Questions](./lesson_40_questions.md) · [Answers](./lesson_40_answers.md)  
> **Practice Task:** Generic Stack\<T\> with push/pop/peek

---

## Table of Contents

1. [What Are Generics?](#1-what-are-generics)
2. [Generic Functions](#2-generic-functions)
3. [Multiple Type Parameters](#3-multiple-type-parameters)
4. [Monomorphization — Zero-Cost Abstraction](#4-monomorphization--zero-cost-abstraction)
5. [Generics with Trait Bounds](#5-generics-with-trait-bounds)
6. [Generic Return Types](#6-generic-return-types)
7. [Turbofish for Generics](#7-turbofish-for-generics)
8. [Common Generic Patterns](#8-common-generic-patterns)
9. [Real-World Example: Generic Stack](#9-real-world-example-generic-stack)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Are Generics?

Generics let you write code that works with **any type**, avoiding duplication:

```rust
// Without generics — duplicated logic
fn largest_i32(list: &[i32]) -> &i32 {
    let mut max = &list[0];
    for item in &list[1..] {
        if item > max { max = item; }
    }
    max
}

fn largest_char(list: &[char]) -> &char {
    let mut max = &list[0];
    for item in &list[1..] {
        if item > max { max = item; }
    }
    max
}

// With generics — one function for all types
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut max = &list[0];
    for item in &list[1..] {
        if item > max { max = item; }
    }
    max
}

fn main() {
    let nums = vec![34, 50, 25, 100, 65];
    let chars = vec!['y', 'm', 'a', 'q'];

    println!("Largest number: {}", largest(&nums));  // 100
    println!("Largest char: {}", largest(&chars));    // y
}
```

---

## 2. Generic Functions

The `<T>` after the function name declares a type parameter:

```rust
// T is a type parameter — placeholder for any type
fn identity<T>(value: T) -> T {
    value
}

fn first<T>(items: &[T]) -> &T {
    &items[0]
}

fn swap<T>(a: T, b: T) -> (T, T) {
    (b, a)
}

fn main() {
    let x = identity(42);          // T = i32
    let y = identity("hello");     // T = &str
    let z = identity(3.14);        // T = f64

    println!("{x}, {y}, {z}");

    let arr = [10, 20, 30];
    println!("First: {}", first(&arr));  // 10

    let (a, b) = swap("hello", "world");
    println!("{a} {b}");  // world hello
}
```

---

## 3. Multiple Type Parameters

Functions can have multiple generic types:

```rust
fn pair<A, B>(first: A, second: B) -> (A, B) {
    (first, second)
}

fn zip_with<T, U, R, F>(a: T, b: U, f: F) -> R
where
    F: FnOnce(T, U) -> R,
{
    f(a, b)
}

fn main() {
    let p = pair(42, "hello");
    println!("{:?}", p);  // (42, "hello")

    let result = zip_with(5, 3, |a, b| a + b);
    println!("{result}");  // 8

    let result = zip_with("hello", 3, |s, n| s.repeat(n));
    println!("{result}");  // hellohellohello
}
```

### Naming conventions:

| Parameter | Common Usage |
|---|---|
| `T` | Primary type ("Type") |
| `U`, `V` | Additional types |
| `K`, `V` | Key, Value (in maps) |
| `E` | Error type |
| `R` | Return type |
| `F` | Function/closure type |

---

## 4. Monomorphization — Zero-Cost Abstraction

Rust compiles generic functions into **specialized versions** for each type used. This is called **monomorphization**:

```rust
fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
    a + b
}

fn main() {
    let x = add(5_i32, 10);    // compiler generates add_i32
    let y = add(1.5_f64, 2.5); // compiler generates add_f64
}
```

At compile time, Rust creates:

```rust
// Compiler generates these specialized functions:
fn add_i32(a: i32, b: i32) -> i32 { a + b }
fn add_f64(a: f64, b: f64) -> f64 { a + b }
```

**Zero-cost** means:
- No runtime overhead — generics are as fast as hand-written specialized code
- No vtable lookup — the type is known at compile time
- Binary size may increase (one copy per type), but performance is optimal

---

## 5. Generics with Trait Bounds

Combining generics with trait bounds (review from Lesson 39):

```rust
use std::fmt::Display;

fn print_largest<T: PartialOrd + Display>(list: &[T]) {
    let max = list.iter().max_by(|a, b| a.partial_cmp(b).unwrap());
    match max {
        Some(val) => println!("Largest: {val}"),
        None => println!("Empty list!"),
    }
}

fn summarize_all<T: Display>(items: &[T]) -> String {
    items.iter()
        .map(|item| item.to_string())
        .collect::<Vec<_>>()
        .join(", ")
}

fn main() {
    print_largest(&[3, 1, 4, 1, 5, 9, 2, 6]);  // Largest: 9
    print_largest(&[3.14, 2.71, 1.41]);           // Largest: 3.14

    let summary = summarize_all(&[1, 2, 3, 4, 5]);
    println!("[{summary}]");  // [1, 2, 3, 4, 5]
}
```

---

## 6. Generic Return Types

```rust
// Returning the same generic type
fn double<T: std::ops::Add<Output = T> + Copy>(value: T) -> T {
    value + value
}

// Returning a different type based on input
fn to_vec<T: Clone>(slice: &[T]) -> Vec<T> {
    slice.to_vec()
}

// Using turbofish to specify the return type
fn parse_or_default<T: std::str::FromStr + Default>(s: &str) -> T {
    s.parse().unwrap_or_default()
}

fn main() {
    println!("{}", double(21));      // 42
    println!("{}", double(1.5));     // 3.0

    let v = to_vec(&[1, 2, 3]);
    println!("{:?}", v);             // [1, 2, 3]

    let n: i32 = parse_or_default("42");
    let f: f64 = parse_or_default("bad");
    println!("{n}, {f}");            // 42, 0.0
}
```

---

## 7. Turbofish for Generics

When the compiler can't infer the type, use turbofish `::<T>`:

```rust
fn main() {
    // Compiler can infer from usage:
    let x: i32 = "42".parse().unwrap();  // type annotation

    // Or use turbofish:
    let x = "42".parse::<i32>().unwrap();  // turbofish

    // Needed when type isn't otherwise constrained:
    let v = (0..5).collect::<Vec<i32>>();  // turbofish required
    println!("{:?}", v);

    // Multiple type parameters:
    fn make_pair<A, B>(a: A, b: B) -> (A, B) { (a, b) }
    let p = make_pair::<i32, &str>(42, "hello");
    println!("{:?}", p);
}
```

---

## 8. Common Generic Patterns

### Pattern 1: Convert with Into

```rust
fn greet(name: impl Into<String>) {
    let name = name.into();
    println!("Hello, {name}!");
}

fn main() {
    greet("Alice");           // &str → String
    greet(String::from("Bob")); // String → String (no-op)
}
```

### Pattern 2: Accept any iterable

```rust
fn sum_all<I>(items: I) -> i32
where
    I: IntoIterator<Item = i32>,
{
    items.into_iter().sum()
}

fn main() {
    println!("{}", sum_all(vec![1, 2, 3]));  // 6
    println!("{}", sum_all([4, 5, 6]));       // 15
    println!("{}", sum_all(1..=10));           // 55
}
```

### Pattern 3: Builder with generics

```rust
fn with_default<T: Default>() -> T {
    T::default()
}

fn main() {
    let n: i32 = with_default();       // 0
    let s: String = with_default();    // ""
    let v: Vec<i32> = with_default();  // []
    println!("{n}, '{s}', {:?}", v);
}
```

---

## 9. Real-World Example: Generic Stack

The roadmap practice task:

```rust
#[derive(Debug)]
struct Stack<T> {
    elements: Vec<T>,
}

impl<T> Stack<T> {
    fn new() -> Self {
        Stack { elements: Vec::new() }
    }

    fn push(&mut self, item: T) {
        self.elements.push(item);
    }

    fn pop(&mut self) -> Option<T> {
        self.elements.pop()
    }

    fn peek(&self) -> Option<&T> {
        self.elements.last()
    }

    fn is_empty(&self) -> bool {
        self.elements.is_empty()
    }

    fn len(&self) -> usize {
        self.elements.len()
    }
}

// Additional methods only for Display types
impl<T: std::fmt::Display> Stack<T> {
    fn print_top(&self) {
        match self.peek() {
            Some(top) => println!("Top: {top}"),
            None => println!("Stack is empty"),
        }
    }

    fn print_all(&self) {
        println!("Stack (top → bottom):");
        for item in self.elements.iter().rev() {
            println!("  | {item}");
        }
        println!("  +---");
    }
}

fn main() {
    // Integer stack
    let mut nums = Stack::new();
    nums.push(10);
    nums.push(20);
    nums.push(30);

    nums.print_all();
    // Stack (top → bottom):
    //   | 30
    //   | 20
    //   | 10
    //   +---

    println!("Pop: {:?}", nums.pop());  // Some(30)
    nums.print_top();                     // Top: 20

    // String stack
    let mut words = Stack::new();
    words.push("hello".to_string());
    words.push("world".to_string());
    words.push("rust".to_string());

    words.print_all();
    println!("Length: {}", words.len());

    // Works with any type — even without Display
    let mut points = Stack::new();
    points.push((1, 2));
    points.push((3, 4));
    println!("Top point: {:?}", points.peek());
    // points.print_top();  // ❌ (i32, i32) doesn't implement Display
}
```

---

## 10. Summary Cheat Sheet

```
GENERIC FUNCTIONS
────────────────────────────────────────────────────────────
fn f<T>(x: T) -> T               one type parameter
fn f<T, U>(x: T, y: U) -> (T,U)  multiple type parameters
fn f<T: Bound>(x: T)             with trait bound
fn f<T>(x: T) where T: Bound     where clause

MONOMORPHIZATION
────────────────────────────────────────────────────────────
Generic code → specialized versions at compile time
Zero runtime overhead — same speed as hand-written code
Trade-off: may increase binary size

TURBOFISH
────────────────────────────────────────────────────────────
f::<Type>(args)                  specify type explicitly
"42".parse::<i32>()              common usage
iter.collect::<Vec<_>>()         common usage

COMMON PATTERNS
────────────────────────────────────────────────────────────
impl Into<String>                accept &str or String
IntoIterator<Item = T>           accept any iterable
T: Default                       construct defaults
T: Clone + PartialOrd            flexible comparison

NAMING CONVENTIONS
────────────────────────────────────────────────────────────
T, U, V     general types
K, V        key-value
E           error
R           return type
F           function/closure
```

---

## What's Next?

**Lesson 41 — Generics in Structs & Enums** — Apply generics to data structures. Learn `struct Point<T>`, `enum Option<T>`, `impl<T>`, and how the standard library uses them everywhere.

## Further Reading
- [The Rust Book — Ch 10.1: Generic Data Types](https://doc.rust-lang.org/book/ch10-01-syntax.html)
- [Rust Performance Book — Monomorphization](https://nnethercote.github.io/perf-book/type-sizes.html)

---

*Generics: write once, work with every type — at zero cost! 🦀*
