# 📘 Lesson 13 — Methods & `impl` (S2)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** S2 · Category: 🏗 Structs/Enums  
> **Previous:** [Lesson 12 — Structs](./lesson_12_structs.md)  
> **Next:** [Lesson 14 — Enums](./lesson_14_enums.md)  
> **Practice:** [Questions](./lesson_13_questions.md) · [Answers](./lesson_13_answers.md)  
> **Practice Task:** Add a `new()` constructor and a `scale()` method to `Rectangle`

---

## Table of Contents

1. [Recap — What Is an `impl` Block?](#1-recap--what-is-an-impl-block)
2. [The Three Forms of `self`](#2-the-three-forms-of-self)
3. [Auto-Ref and Auto-Deref](#3-auto-ref-and-auto-deref)
4. [Associated Functions — Constructors](#4-associated-functions--constructors)
5. [`Self` vs `self`](#5-self-vs-self)
6. [Multiple `impl` Blocks](#6-multiple-impl-blocks)
7. [Methods on Enums](#7-methods-on-enums)
8. [Method Chaining](#8-method-chaining)
9. [Implementing Standard Traits](#9-implementing-standard-traits)
10. [Getters and the Naming Convention](#10-getters-and-the-naming-convention)
11. [When to Use a Method vs a Function](#11-when-to-use-a-method-vs-a-function)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. Recap — What Is an `impl` Block?

An `impl` block attaches **methods** and **associated functions** to a type:

```rust
struct Circle { radius: f64 }

impl Circle {
    // Associated function (no self) — called as Circle::new(...)
    fn new(radius: f64) -> Self {
        Circle { radius }
    }

    // Method — called as circle.area()
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
}
```

You can have as many `impl` blocks for the same type as you want — they're all merged.

---

## 2. The Three Forms of `self`

The first parameter of every method controls access to the instance:

### `&self` — immutable borrow (read only)

```rust
impl Rectangle {
    fn area(&self) -> f64 {       // borrows rect for reading
        self.width * self.height
    }
}
```
- Does **not** move or modify the struct
- Caller still owns the struct after the call
- Most methods use this form

### `&mut self` — mutable borrow (can modify)

```rust
impl Rectangle {
    fn scale(&mut self, factor: f64) {   // borrows rect for modification
        self.width  *= factor;
        self.height *= factor;
    }
}
```
- Can read **and** write fields
- Caller must have a `mut` binding
- The struct remains owned by the caller after the call

### `self` — takes ownership (consuming method)

```rust
impl Rectangle {
    fn into_square(self) -> Rectangle {   // consumes rect
        let side = (self.width + self.height) / 2.0;
        Rectangle { width: side, height: side }
    }
}

fn main() {
    let r = Rectangle { width: 10.0, height: 4.0 };
    let sq = r.into_square();
    // println!("{}", r.width); // ❌ r was moved
    println!("{}×{}", sq.width, sq.height); // 7×7
}
```
- Moves `self` into the method
- The caller no longer has access to the original value
- Used for transformations that produce a new value, and builder patterns

### Summary table

| Form | Access | After call |
|------|--------|-----------|
| `&self` | read-only | caller keeps ownership |
| `&mut self` | read + write | caller keeps ownership |
| `self` | read + write + own | caller's value is gone |

---

## 3. Auto-Ref and Auto-Deref

When you call a method, Rust automatically inserts `&`, `&mut`, or nothing based on the method signature. You never need to write `(&rect).area()`:

```rust
rect.area()          // Rust inserts &    → Rectangle::area(&rect)
rect.scale(2.0)      // Rust inserts &mut → Rectangle::scale(&mut rect)
rect.into_square()   // Rust moves        → Rectangle::into_square(rect)
```

This **auto-ref** behaviour means method calls always "just work" as long as the binding has the right mutability.

**Universal Function Call Syntax (UFCS)** — the explicit form:
```rust
Rectangle::area(&rect)        // same as rect.area()
Rectangle::scale(&mut rect, 2.0) // same as rect.scale(2.0)
```

UFCS is useful when you need to disambiguate between two traits that both define a method with the same name.

---

## 4. Associated Functions — Constructors

Associated functions don't take `self`. The most common use is constructors:

```rust
impl Rectangle {
    fn new(width: f64, height: f64) -> Self {
        Rectangle { width, height }
    }

    fn square(side: f64) -> Self {
        Self { width: side, height: side }
    }

    fn unit() -> Self {
        Self { width: 1.0, height: 1.0 }
    }
}

fn main() {
    let r = Rectangle::new(10.0, 5.0);   // :: syntax
    let s = Rectangle::square(4.0);
    let u = Rectangle::unit();
}
```

### Naming conventions for associated functions

| Pattern | Example |
|---------|---------|
| Primary constructor | `fn new(...)` |
| Named constructors | `fn from_area(a: f64)`, `fn with_ratio(r: f64, ratio: f64)` |
| Constant instances | `fn zero()`, `fn identity()`, `fn unit()` |

The standard library follows this everywhere: `String::new()`, `Vec::with_capacity(n)`, `HashMap::from([...])`.

---

## 5. `Self` vs `self`

| | Meaning |
|---|---|
| `self` (lowercase) | The instance — the value itself |
| `Self` (uppercase) | The type of the instance — alias for the implementing type |

```rust
impl Circle {
    // Self = Circle (the type)
    fn doubled(&self) -> Self {
        Self { radius: self.radius * 2.0 }
    }
}
```

Using `Self` instead of the type name means if you rename `Circle` to `Ellipse`, you only change the struct definition — not every method return type.

---

## 6. Multiple `impl` Blocks

You can split a type's methods across multiple `impl` blocks. They are completely equivalent — just a code organisation tool:

```rust
struct Matrix { data: [[f64; 2]; 2] }

// Constructors
impl Matrix {
    fn new(a: f64, b: f64, c: f64, d: f64) -> Self {
        Matrix { data: [[a, b], [c, d]] }
    }
    fn identity() -> Self {
        Matrix { data: [[1.0, 0.0], [0.0, 1.0]] }
    }
    fn zero() -> Self {
        Matrix { data: [[0.0, 0.0], [0.0, 0.0]] }
    }
}

// Operations
impl Matrix {
    fn determinant(&self) -> f64 {
        self.data[0][0] * self.data[1][1] - self.data[0][1] * self.data[1][0]
    }
    fn transpose(&self) -> Self {
        Matrix { data: [
            [self.data[0][0], self.data[1][0]],
            [self.data[0][1], self.data[1][1]],
        ]}
    }
}

fn main() {
    let m = Matrix::new(1.0, 2.0, 3.0, 4.0);
    println!("det = {}", m.determinant()); // -2
}
```

The two `impl` blocks are merged at compile time — there is no performance difference from having one large block.

---

## 7. Methods on Enums

`impl` works on enums too — exactly the same syntax:

```rust
#[derive(Debug)]
enum Direction { North, South, East, West }

impl Direction {
    fn opposite(&self) -> Direction {
        match self {
            Direction::North => Direction::South,
            Direction::South => Direction::North,
            Direction::East  => Direction::West,
            Direction::West  => Direction::East,
        }
    }

    fn is_vertical(&self) -> bool {
        matches!(self, Direction::North | Direction::South)
    }

    fn vector(&self) -> (i32, i32) {
        match self {
            Direction::North => (0,  1),
            Direction::South => (0, -1),
            Direction::East  => (1,  0),
            Direction::West  => (-1, 0),
        }
    }
}

fn main() {
    let d = Direction::North;
    println!("{:?} → {:?}", d, d.opposite());  // North → South
    println!("vertical: {}", d.is_vertical()); // true
    println!("vector: {:?}", d.vector());      // (0, 1)
}
```

---

## 8. Method Chaining

When a method returns `Self` (or `&mut Self`), you can chain multiple calls:

### Consuming chain (returns `Self`)

```rust
#[derive(Debug)]
struct Config {
    host: String,
    port: u16,
    debug: bool,
}

impl Config {
    fn new(host: &str, port: u16) -> Self {
        Config { host: host.to_string(), port, debug: false }
    }

    fn debug(mut self) -> Self {
        self.debug = true;
        self
    }
}

fn main() {
    let cfg = Config::new("localhost", 8080)
        .debug();
    println!("{cfg:?}");
}
```

### Mutable chain (returns `&mut Self`)

```rust
struct Builder { parts: Vec<String> }

impl Builder {
    fn new() -> Self { Builder { parts: vec![] } }

    fn add(&mut self, part: &str) -> &mut Self {
        self.parts.push(part.to_string());
        self
    }

    fn build(&self) -> String { self.parts.join(", ") }
}

fn main() {
    let mut b = Builder::new();
    b.add("alpha").add("beta").add("gamma");
    println!("{}", b.build()); // alpha, beta, gamma
}
```

---

## 9. Implementing Standard Traits

Traits are Rust's way of defining shared behaviour. You implement them with `impl TraitName for Type { }`:

### `Display` — human-readable `{}`

```rust
use std::fmt;

struct Point { x: f64, y: f64 }

impl fmt::Display for Point {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "({:.1}, {:.1})", self.x, self.y)
    }
}
```

### `Default` — zero/empty value

```rust
#[derive(Debug)]
struct Config { width: u32, height: u32, title: String }

impl Default for Config {
    fn default() -> Self {
        Config { width: 800, height: 600, title: String::from("Untitled") }
    }
}

fn main() {
    let c = Config::default();
    let c2 = Config { width: 1920, ..Config::default() };
}
```

Or use `#[derive(Default)]` if all fields implement `Default` (zeros and empty strings):

```rust
#[derive(Default, Debug)]
struct Config { width: u32, height: u32, title: String }
```

### `From` / `Into` — type conversions

```rust
struct Celsius(f64);
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self {
        Fahrenheit(c.0 * 9.0 / 5.0 + 32.0)
    }
}

fn main() {
    let boiling = Celsius(100.0);
    let f: Fahrenheit = boiling.into();  // Into auto-derived from From
    println!("{:.1}°F", f.0);            // 212.0°F
}
```

---

## 10. Getters and the Naming Convention

Rust's convention for "getter" methods: **name the method the same as the field**. No `get_` prefix:

```rust
pub struct Circle {
    radius: f64,   // private
}

impl Circle {
    pub fn new(radius: f64) -> Self {
        assert!(radius > 0.0, "radius must be positive");
        Circle { radius }
    }

    pub fn radius(&self) -> f64 { self.radius }  // NOT get_radius()

    pub fn set_radius(&mut self, r: f64) {
        assert!(r > 0.0);
        self.radius = r;
    }
}
```

Private fields + public getters enforce **invariants** — conditions that must always hold. Code outside the module can't set `radius` to a negative value.

---

## 11. When to Use a Method vs a Function

| Situation | Use |
|-----------|-----|
| Operation is "about" a specific type | Method |
| Needs access to `self` fields | Method |
| Constructor or factory | Associated function |
| Generic utility across many types | Free function |
| Operation doesn't logically "belong" to one type | Free function |

```rust
// Method — clearly belongs to Rectangle
impl Rectangle {
    fn area(&self) -> f64 { self.width * self.height }
}

// Free function — operates on two different types
fn distance(a: &Point, b: &Point) -> f64 {
    ((a.x-b.x).powi(2) + (a.y-b.y).powi(2)).sqrt()
}
```

---

## 12. Summary Cheat Sheet

```
SELF FORMS
────────────────────────────────────────────────────────────
&self           read-only borrow — most common
&mut self       mutable borrow — can modify fields
self            takes ownership — consuming/transforming

Self (capital)  type alias for the implementing type

CALL SYNTAX
────────────────────────────────────────────────────────────
value.method()           dot notation (auto-ref applied)
Type::assoc_fn(args)     associated function (no self)
Type::method(&value)     UFCS explicit form

AUTO-REF RULES
────────────────────────────────────────────────────────────
value.method()  →  Type::method(&value)      if &self
value.method()  →  Type::method(&mut value)  if &mut self
value.method()  →  Type::method(value)       if self

COMMON STANDARD TRAITS
────────────────────────────────────────────────────────────
Display         impl fmt::Display  → {}
Debug           #[derive(Debug)]   → {:?}
Default         impl Default       → Type::default()
From/Into       impl From<A> for B → B::from(a), a.into()
Clone           #[derive(Clone)]   → .clone()
PartialEq       #[derive(PartialEq)] → ==, !=

GETTER CONVENTION
────────────────────────────────────────────────────────────
fn radius(&self) -> f64  ← NOT fn get_radius(...)
fn set_radius(&mut self, r: f64)
```

---

## What's Next?

**Lesson 14 — Enums** — Define types that are *one of several variants*, attach data to variants, and meet `Option<T>`.

## Further Reading
- [The Rust Book — Ch 5.3: Method Syntax](https://doc.rust-lang.org/book/ch05-03-method-syntax.html)
- [The Rust Book — Ch 10.2: Traits](https://doc.rust-lang.org/book/ch10-02-traits.html)
