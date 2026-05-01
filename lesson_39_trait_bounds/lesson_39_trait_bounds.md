# 📘 Lesson 39 — Trait Bounds (T2)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T2 · Category: 🔷 Traits  
> **Previous:** [Lesson 38 — Traits: Definition & Implementation](../lesson_38_traits/lesson_38_traits.md)  
> **Next:** [Lesson 40 — Generics in Functions](../lesson_40_generics/lesson_40_generics.md)  
> **Practice:** [Questions](./lesson_39_questions.md) · [Answers](./lesson_39_answers.md)  
> **Practice Task:** Generic largest() fn bounded by PartialOrd + Copy

---

## Table of Contents

1. [What Are Trait Bounds?](#1-what-are-trait-bounds)
2. [Basic Trait Bound Syntax](#2-basic-trait-bound-syntax)
3. [Multiple Trait Bounds](#3-multiple-trait-bounds)
4. [where Clauses](#4-where-clauses)
5. [Conditional Method Implementation](#5-conditional-method-implementation)
6. [Blanket Implementations](#6-blanket-implementations)
7. [Common Standard Library Bounds](#7-common-standard-library-bounds)
8. [Real-World Example: largest()](#8-real-world-example-largest)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Are Trait Bounds?

Trait bounds **constrain** generic types: "T can be any type, **as long as** it implements these traits."

```rust
// Without bounds — can't compare items
// fn largest<T>(list: &[T]) -> &T { ... }  // ❌ What if T can't be compared?

// With bounds — T must be comparable
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut max = &list[0];
    for item in &list[1..] {
        if item > max {
            max = item;
        }
    }
    max
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    println!("Largest: {}", largest(&numbers));  // 100

    let chars = vec!['y', 'm', 'a', 'q'];
    println!("Largest: {}", largest(&chars));  // y
}
```

---

## 2. Basic Trait Bound Syntax

Two equivalent ways to write trait bounds:

```rust
use std::fmt::Display;

// Syntax 1: impl Trait (shorthand)
fn print_item(item: &impl Display) {
    println!("{item}");
}

// Syntax 2: Generic with bound (explicit)
fn print_item_v2<T: Display>(item: &T) {
    println!("{item}");
}

// Syntax 2 is required when you need the SAME type for multiple params:
fn compare<T: PartialOrd + Display>(a: &T, b: &T) {
    if a > b {
        println!("{a} is greater than {b}");
    } else {
        println!("{b} is greater than or equal to {a}");
    }
}

fn main() {
    print_item(&42);
    print_item(&"hello");
    compare(&10, &20);
}
```

---

## 3. Multiple Trait Bounds

Use `+` to require multiple traits:

```rust
use std::fmt::{Debug, Display};

// T must implement BOTH Display AND Debug
fn inspect<T: Display + Debug>(item: &T) {
    println!("Display: {item}");
    println!("Debug:   {item:?}");
}

// Multiple generics with bounds
fn process<T: Display + Clone, U: Debug>(item: T, context: U) {
    let copy = item.clone();
    println!("Item: {item}, Copy: {copy}");
    println!("Context: {context:?}");
}

fn main() {
    inspect(&42);
    process("hello".to_string(), vec![1, 2, 3]);
}
```

---

## 4. where Clauses

When bounds get long, `where` clauses improve readability:

```rust
use std::fmt::{Debug, Display};

// Without where — hard to read
fn complex<T: Display + Clone + Debug, U: Debug + PartialOrd>(a: T, b: U) -> String {
    format!("{a} / {b:?}")
}

// With where — much cleaner
fn complex_v2<T, U>(a: T, b: U) -> String
where
    T: Display + Clone + Debug,
    U: Debug + PartialOrd,
{
    format!("{a} / {b:?}")
}

// Especially useful with return types
fn create_pair<T, U>(a: T, b: U) -> (T, U)
where
    T: Clone + Display,
    U: Clone + Debug,
{
    println!("Creating pair: {a} + {b:?}");
    (a, b)
}

fn main() {
    let pair = create_pair("hello".to_string(), 42);
    println!("{:?}", pair);
}
```

### When to use `where`:
- More than 2 bounds on a type parameter
- More than 2 type parameters
- Complex return types
- Personal preference for readability

---

## 5. Conditional Method Implementation

Add methods only when the generic type satisfies certain bounds:

```rust
use std::fmt::Display;

struct Pair<T> {
    x: T,
    y: T,
}

// Methods available for ALL Pair<T>
impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Pair { x, y }
    }
}

// Methods ONLY available when T: Display + PartialOrd
impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("Largest: {}", self.x);
        } else {
            println!("Largest: {}", self.y);
        }
    }
}

fn main() {
    let pair = Pair::new(5, 10);
    pair.cmp_display();  // ✅ i32 implements Display + PartialOrd

    let pair2 = Pair::new(vec![1], vec![2]);
    // pair2.cmp_display();  // ❌ Vec doesn't implement Display
}
```

---

## 6. Blanket Implementations

Implement a trait for ALL types that satisfy a bound:

```rust
use std::fmt::Display;

trait Printable {
    fn print_fancy(&self);
}

// Blanket impl: ANY type that implements Display gets Printable for free
impl<T: Display> Printable for T {
    fn print_fancy(&self) {
        println!("★ {self} ★");
    }
}

fn main() {
    42.print_fancy();          // ★ 42 ★
    "hello".print_fancy();     // ★ hello ★
    3.14_f64.print_fancy();    // ★ 3.14 ★
}
```

The standard library uses this extensively:

```rust
// From std: any type that implements Display automatically gets ToString
// impl<T: Display> ToString for T { ... }

fn main() {
    let s: String = 42.to_string();  // works because i32: Display
    println!("{s}");
}
```

---

## 7. Common Standard Library Bounds

| Bound | Meaning | Example Types |
|---|---|---|
| `Display` | Can be formatted with `{}` | `i32`, `String`, `&str` |
| `Debug` | Can be formatted with `{:?}` | Almost everything |
| `Clone` | Can be duplicated | `String`, `Vec`, `i32` |
| `Copy` | Can be bitwise copied | `i32`, `f64`, `bool`, `char` |
| `PartialOrd` | Can be compared with `<`, `>` | `i32`, `f64`, `String` |
| `Ord` | Total ordering | `i32`, `String` (NOT `f64`) |
| `PartialEq` | Can be compared with `==` | Most types |
| `Eq` | Reflexive equality | `i32`, `String` (NOT `f64`) |
| `Hash` | Can be hashed | `i32`, `String` (NOT `f64`) |
| `Default` | Has a default value | `i32`→0, `String`→"", `Vec`→[] |
| `Send` | Can be sent between threads | Most types (NOT `Rc`) |
| `Sync` | Can be shared between threads | Most types (NOT `RefCell`) |

---

## 8. Real-World Example: largest()

The roadmap practice task:

```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut max = list[0];
    for &item in &list[1..] {
        if item > max {
            max = item;
        }
    }
    max
}

// Alternative: using Clone instead of Copy (works with String too)
fn largest_clone<T: PartialOrd + Clone>(list: &[T]) -> T {
    let mut max = list[0].clone();
    for item in &list[1..] {
        if *item > max {
            max = item.clone();
        }
    }
    max
}

// Alternative: returning a reference (no Copy/Clone needed)
fn largest_ref<T: PartialOrd>(list: &[T]) -> &T {
    let mut max = &list[0];
    for item in &list[1..] {
        if item > max {
            max = item;
        }
    }
    max
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    println!("Largest number: {}", largest(&numbers));

    let chars = vec!['y', 'm', 'a', 'q'];
    println!("Largest char: {}", largest(&chars));

    let strings = vec!["hello".to_string(), "world".to_string(), "abc".to_string()];
    println!("Largest string: {}", largest_clone(&strings));
    println!("Largest (ref): {}", largest_ref(&strings));
}
```

---

## 9. Summary Cheat Sheet

```
TRAIT BOUND SYNTAX
────────────────────────────────────────────────────────────
fn f(x: &impl Trait)            impl Trait shorthand
fn f<T: Trait>(x: &T)           explicit bound
fn f<T: A + B>(x: &T)           multiple bounds
fn f<T>(x: &T) where T: A + B  where clause

CONDITIONAL IMPLEMENTATION
────────────────────────────────────────────────────────────
impl<T: Display> Pair<T> {     methods only when T: Display
    fn show(&self) { ... }
}

BLANKET IMPLEMENTATION
────────────────────────────────────────────────────────────
impl<T: Display> MyTrait for T  implement for ALL Display types

COMMON BOUNDS
────────────────────────────────────────────────────────────
PartialOrd + Copy    for comparisons (primitives)
PartialOrd + Clone   for comparisons (all types)
Display + Debug      for printing
Hash + Eq            for HashMap/HashSet keys
Clone + Default      for flexible construction

WHERE CLAUSE — USE WHEN
────────────────────────────────────────────────────────────
Many bounds (3+)
Many type parameters
Complex signatures
Readability matters
```

---

## What's Next?

**Lesson 40 — Generics in Functions** — Deep dive into generic functions, monomorphization, and zero-cost abstraction. Build a generic `Stack<T>` with push, pop, and peek.

## Further Reading
- [The Rust Book — Ch 10.2: Traits as Parameters](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters)
- [Rust Reference — Trait Bounds](https://doc.rust-lang.org/reference/trait-bounds.html)

---

*Trait bounds: the guardrails that make generics safe! 🦀*
