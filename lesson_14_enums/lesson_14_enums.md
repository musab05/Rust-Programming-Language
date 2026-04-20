# 📘 Lesson 14 — Enums (S3)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** S3 · Category: 🏗 Structs/Enums  
> **Previous:** [Lesson 13 — Methods & impl](./lesson_13_methods_impl.md)  
> **Next:** [Lesson 15 — Pattern Matching: match](./lesson_15_pattern_matching.md)  
> **Practice:** [Questions](./lesson_14_questions.md) · [Answers](./lesson_14_answers.md)  
> **Practice Task:** Model a `Shape` enum with `Circle(r)`, `Rect(w,h)`, `Triangle(b,h)`

---

## Table of Contents

1. [What Is an Enum?](#1-what-is-an-enum)
2. [Simple Enums — Unit Variants](#2-simple-enums--unit-variants)
3. [Enums With Data — Tuple Variants](#3-enums-with-data--tuple-variants)
4. [Enums With Data — Struct Variants](#4-enums-with-data--struct-variants)
5. [Mixed Variants in One Enum](#5-mixed-variants-in-one-enum)
6. [Methods on Enums](#6-methods-on-enums)
7. [The `Option<T>` Enum](#7-the-optiont-enum)
8. [Using `Option` — Key Methods](#8-using-option--key-methods)
9. [Enums and Memory](#9-enums-and-memory)
10. [Recursive Enums with `Box<T>`](#10-recursive-enums-with-boxt)
11. [Deriving Traits on Enums](#11-deriving-traits-on-enums)
12. [Enums vs Structs — When to Use Which](#12-enums-vs-structs--when-to-use-which)
13. [Summary Cheat Sheet](#13-summary-cheat-sheet)

---

## 1. What Is an Enum?

An **enum** (enumeration) defines a type that can be **one of several named variants**. Each variant is mutually exclusive — a value can only be one variant at a time.

In most languages, enums are just named integers. In Rust, **each variant can carry different data** — making Rust enums the most expressive type in the language.

```rust
// Simple — no data
enum Direction { North, South, East, West }

// Rich — data varies by variant
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(u8, u8, u8),
}
```

The technical term is **algebraic data type** (ADT) — specifically a **sum type**: a value is this *or* that, but only one at a time.

---

## 2. Simple Enums — Unit Variants

The simplest form: a closed set of named choices with no attached data.

```rust
#[derive(Debug, PartialEq)]
enum Season { Spring, Summer, Autumn, Winter }

fn describe(s: &Season) -> &str {
    match s {
        Season::Spring => "flowers bloom",
        Season::Summer => "hot days",
        Season::Autumn => "leaves fall",
        Season::Winter => "snow and cold",
    }
}

fn main() {
    let now = Season::Summer;
    println!("{:?}: {}", now, describe(&now));

    // Variants are values — store them, compare them
    if now == Season::Summer { println!("it's summer!"); }
}
```

### `use` to bring variants into scope

```rust
use Season::*;
let s = Winter; // no prefix needed
```

---

## 3. Enums With Data — Tuple Variants

Variants can hold positional (unnamed) data:

```rust
#[derive(Debug)]
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

fn main() {
    let home     = IpAddr::V4(127, 0, 0, 1);
    let loopback = IpAddr::V6(String::from("::1"));
    println!("{:?}", home);
    println!("{:?}", loopback);
}
```

To get the data out, you use **pattern matching** (next lesson). For now:

```rust
if let IpAddr::V4(a, b, c, d) = home {
    println!("{a}.{b}.{c}.{d}");
}
```

---

## 4. Enums With Data — Struct Variants

Variants can also hold named fields:

```rust
#[derive(Debug)]
enum Event {
    KeyPress { key: char, ctrl: bool },
    Click    { x: i32,   y: i32 },
    Resize   { width: u32, height: u32 },
}

fn handle(e: &Event) {
    match e {
        Event::KeyPress { key, ctrl } => println!("key '{key}' ctrl={ctrl}"),
        Event::Click { x, y }         => println!("click at ({x},{y})"),
        Event::Resize { width, height } => println!("resize to {width}×{height}"),
    }
}

fn main() {
    let events = vec![
        Event::KeyPress { key: 'A', ctrl: false },
        Event::Click { x: 100, y: 200 },
        Event::Resize { width: 1920, height: 1080 },
    ];
    for e in &events { handle(e); }
}
```

---

## 5. Mixed Variants in One Enum

A single enum can freely mix unit, tuple, and struct variants:

```rust
#[derive(Debug)]
enum Shape {
    Circle(f64),             // radius
    Rect(f64, f64),          // width, height
    Triangle(f64, f64),      // base, height
}
```

This is the **roadmap practice task**. Let's implement it fully:

```rust
use std::f64::consts::PI;

impl Shape {
    fn area(&self) -> f64 {
        match self {
            Shape::Circle(r)       => PI * r * r,
            Shape::Rect(w, h)      => w * h,
            Shape::Triangle(b, h)  => 0.5 * b * h,
        }
    }

    fn perimeter(&self) -> f64 {
        match self {
            Shape::Circle(r)      => 2.0 * PI * r,
            Shape::Rect(w, h)     => 2.0 * (w + h),
            Shape::Triangle(b, h) => {
                // right triangle assumption: hypotenuse from b, h
                b + h + (b*b + h*h).sqrt()
            }
        }
    }

    fn name(&self) -> &str {
        match self {
            Shape::Circle(_)   => "Circle",
            Shape::Rect(_, _)  => "Rectangle",
            Shape::Triangle(..) => "Triangle",
        }
    }
}

fn main() {
    let shapes = vec![
        Shape::Circle(5.0),
        Shape::Rect(4.0, 6.0),
        Shape::Triangle(3.0, 4.0),
    ];
    for s in &shapes {
        println!("{}: area={:.2}", s.name(), s.area());
    }
}
```

---

## 6. Methods on Enums

`impl` blocks work on enums identically to structs:

```rust
#[derive(Debug, PartialEq)]
enum TrafficLight { Red, Yellow, Green }

impl TrafficLight {
    fn next(&self) -> TrafficLight {
        match self {
            TrafficLight::Red    => TrafficLight::Green,
            TrafficLight::Green  => TrafficLight::Yellow,
            TrafficLight::Yellow => TrafficLight::Red,
        }
    }

    fn can_go(&self) -> bool {
        *self == TrafficLight::Green
    }

    fn duration_secs(&self) -> u32 {
        match self { TrafficLight::Red => 60, TrafficLight::Yellow => 5, TrafficLight::Green => 45 }
    }
}

fn main() {
    let mut light = TrafficLight::Red;
    for _ in 0..4 {
        println!("{:?} — go={} ({}s)", light, light.can_go(), light.duration_secs());
        light = light.next();
    }
}
```

---

## 7. The `Option<T>` Enum

`Option<T>` is Rust's replacement for `null`. It is defined in the standard library as:

```rust
enum Option<T> {
    Some(T),  // there IS a value
    None,     // there is NO value
}
```

It is **automatically in scope** — you never need to write `Option::Some` or `Option::None`, just `Some(x)` and `None`.

```rust
fn divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 { None } else { Some(a / b) }
}

fn main() {
    println!("{:?}", divide(10.0, 2.0)); // Some(5.0)
    println!("{:?}", divide(10.0, 0.0)); // None
}
```

### Why not just use a sentinel value?

```rust
// Bad — caller can forget to check:
fn find_index(v: &[i32], target: i32) -> i32 {
    v.iter().position(|&x| x == target).unwrap_or(-1) as i32  // -1 means "not found"
}

// Good — the type system forces you to handle absence:
fn find_index(v: &[i32], target: i32) -> Option<usize> {
    v.iter().position(|&x| x == target)
}
```

With `Option`, the compiler will not let you accidentally use a "not found" value as if it were a real index.

---

## 8. Using `Option` — Key Methods

### Getting the value

```rust
let x: Option<i32> = Some(42);
let y: Option<i32> = None;

// .unwrap() — get the value, PANIC if None
println!("{}", x.unwrap()); // 42
// y.unwrap();               // PANIC: called unwrap on None

// .unwrap_or(default) — get value or fallback
println!("{}", y.unwrap_or(0)); // 0

// .unwrap_or_else(|| ...) — lazy fallback
println!("{}", y.unwrap_or_else(|| 99)); // 99

// .expect("message") — like unwrap but better error message
println!("{}", x.expect("should have a value")); // 42
```

### Checking presence

```rust
println!("{}", x.is_some()); // true
println!("{}", y.is_none()); // true
```

### Transforming

```rust
// .map(|v| ...) — transform Some value, leave None unchanged
let doubled = x.map(|v| v * 2);
println!("{:?}", doubled); // Some(84)

// .filter(|v| ...) — keep Some only if predicate holds
let big = x.filter(|&v| v > 100);
println!("{:?}", big); // None

// .and_then(|v| ...) — chain Option-returning operations
let result = x.and_then(|v| if v > 10 { Some(v + 1) } else { None });
println!("{:?}", result); // Some(43)
```

### The `?` operator with Option

In a function returning `Option<T>`, `?` early-returns `None` if the value is `None`:

```rust
fn double_first(v: &[i32]) -> Option<i32> {
    let first = v.first()?;   // returns None if v is empty
    Some(first * 2)
}

fn main() {
    println!("{:?}", double_first(&[3, 5, 7])); // Some(6)
    println!("{:?}", double_first(&[]));          // None
}
```

---

## 9. Enums and Memory

Rust stores an enum as a **discriminant tag** (which variant) plus enough space for the largest variant:

```rust
enum SmallOrLarge {
    Small(u8),           // 1 byte of data
    Large([u8; 1000]),   // 1000 bytes of data
}
// Size = 1000 bytes (for Large) + tag — Small "wastes" 999 bytes
```

For large data that varies significantly between variants, use `Box<T>` to heap-allocate:

```rust
enum SmallOrLarge {
    Small(u8),
    Large(Box<[u8; 1000]>),  // now just 8 bytes (pointer) + tag
}
```

---

## 10. Recursive Enums with `Box<T>`

Enums can't directly contain themselves — that would be infinite size. Use `Box<T>` to break the cycle:

```rust
// ❌ INVALID — infinite size
// enum List { Node(i32, List), Nil }

// ✅ Box makes the inner List a fixed-size heap pointer
#[derive(Debug)]
enum List {
    Node(i32, Box<List>),
    Nil,
}

impl List {
    fn sum(&self) -> i32 {
        match self {
            List::Nil => 0,
            List::Node(val, rest) => val + rest.sum(),
        }
    }
}

fn main() {
    let list = List::Node(1, Box::new(
                   List::Node(2, Box::new(
                       List::Node(3, Box::new(List::Nil))))));
    println!("sum = {}", list.sum()); // 6
}
```

---

## 11. Deriving Traits on Enums

`#[derive]` works the same on enums as on structs:

```rust
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
enum Status { Active, Inactive, Suspended }

use std::collections::HashMap;

fn main() {
    let s = Status::Active;
    let s2 = s.clone();
    println!("{:?}", s);
    println!("equal: {}", s == s2);

    // Can use as HashMap key because of Hash + Eq
    let mut counts: HashMap<Status, u32> = HashMap::new();
    *counts.entry(Status::Active).or_insert(0) += 3;
    *counts.entry(Status::Inactive).or_insert(0) += 1;
    println!("{counts:?}");
}
```

`Copy` works only if **all** variant data is `Copy`:

```rust
#[derive(Clone, Copy)]
enum Direction { North, South, East, West }  // no data → Copy OK

// #[derive(Copy)]  ← would FAIL because String is not Copy
// enum Msg { Text(String), Quit }
```

---

## 12. Enums vs Structs — When to Use Which

| Situation | Use |
|-----------|-----|
| Value is always composed of A **and** B **and** C | Struct |
| Value is one of A **or** B **or** C | Enum |
| Fixed set of states a thing can be in | Enum |
| Entity with multiple attributes | Struct |
| Error type with different error cases | Enum |
| Configuration with many settings | Struct |

```rust
// Struct — has all three parts at once
struct RGB { r: u8, g: u8, b: u8 }

// Enum — is one of three colour formats at a time
enum Color {
    Rgb(u8, u8, u8),
    Hsl(f64, f64, f64),
    Hex(String),
}
```

---

## 13. Summary Cheat Sheet

```
DEFINING AN ENUM
────────────────────────────────────────────────────────────
enum Name {
    Unit,                    // no data
    Tuple(Type, Type),       // positional data
    Struct { field: Type },  // named data
}

CREATING VALUES
────────────────────────────────────────────────────────────
let a = Name::Unit;
let b = Name::Tuple(1, 2);
let c = Name::Struct { field: val };

ACCESSING DATA — requires match or if let
────────────────────────────────────────────────────────────
match val {
    Name::Unit            => { ... }
    Name::Tuple(x, y)     => { ... }
    Name::Struct { field } => { ... }
}

OPTION<T>
────────────────────────────────────────────────────────────
Some(value)            present
None                   absent
.unwrap()              get value, PANIC if None
.unwrap_or(default)    get value or fallback
.is_some()/.is_none()  check presence
.map(|v| ...)          transform Some value
.and_then(|v| ...)     chain Option-returning fn
?                      early-return None in Option fn

DERIVE TRAITS
────────────────────────────────────────────────────────────
#[derive(Debug, Clone, PartialEq, Eq, Hash)]
Copy only if ALL variant data is Copy
```

---

## What's Next?

**Lesson 15 — Pattern Matching: `match`** — The full power of Rust's `match` expression: destructuring variants, guards, OR patterns, wildcard, and `@` bindings.

## Further Reading
- [The Rust Book — Ch 6: Enums and Pattern Matching](https://doc.rust-lang.org/book/ch06-00-enums.html)
- [Rust by Example — Enums](https://doc.rust-lang.org/rust-by-example/custom_types/enum.html)
