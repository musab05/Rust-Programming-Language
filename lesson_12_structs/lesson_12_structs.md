# 📘 Lesson 12 — Structs (S1)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** S1 · Category: 🏗 Structs/Enums  
> **Previous:** [Lesson 11 — Slices](./lesson_11_slices.md)  
> **Next:** [Lesson 13 — Methods & impl](./lesson_13_methods_impl.md)  
> **Practice:** [Questions](./lesson_12_questions.md) · [Answers](./lesson_12_answers.md)  
> **Practice Task:** Model a `Rectangle` with `area()` and `perimeter()` methods

---

## Table of Contents

1. [What Is a Struct?](#1-what-is-a-struct)
2. [Defining a Struct](#2-defining-a-struct)
3. [Creating Struct Instances](#3-creating-struct-instances)
4. [Accessing and Mutating Fields](#4-accessing-and-mutating-fields)
5. [Field Init Shorthand](#5-field-init-shorthand)
6. [Struct Update Syntax](#6-struct-update-syntax)
7. [Structs and Ownership](#7-structs-and-ownership)
8. [Printing Structs — `Debug` and `Display`](#8-printing-structs--debug-and-display)
9. [Methods on Structs — First Look](#9-methods-on-structs--first-look)
10. [Associated Functions (Constructors)](#10-associated-functions-constructors)
11. [Structs vs Tuples](#11-structs-vs-tuples)
12. [Nested Structs](#12-nested-structs)
13. [Summary Cheat Sheet](#13-summary-cheat-sheet)

---

## 1. What Is a Struct?

A **struct** (short for *structure*) groups related data under a single named type. It is Rust's primary tool for creating custom data types.

```rust
// Without struct — scattered, no relationship expressed
let user_name = String::from("alice");
let user_email = String::from("alice@example.com");
let user_active = true;

// With struct — grouped, named, reusable
struct User {
    name: String,
    email: String,
    active: bool,
}
```

Structs are the foundation of data modeling in Rust. Combined with `impl` blocks (next lesson), they provide everything you need to build well-organized programs.

---

## 2. Defining a Struct

```rust
struct Rectangle {
    width: f64,
    height: f64,
}
```

Rules:
- Name uses **PascalCase** by convention
- Each field has an explicit `name: Type`
- No default values at definition time (use `Default` trait or constructors)
- Fields are **private** by default within other modules (`pub` to expose)

---

## 3. Creating Struct Instances

Every field must be given a value:

```rust
fn main() {
    let rect = Rectangle {
        width: 10.0,
        height: 5.0,
    };
}
```

Order of fields doesn't matter at instantiation:

```rust
let rect2 = Rectangle {
    height: 3.0,
    width: 8.0,   // different order — fine
};
```

### Returning a struct from a function

```rust
fn make_rect(w: f64, h: f64) -> Rectangle {
    Rectangle { width: w, height: h }
}
```

---

## 4. Accessing and Mutating Fields

Use dot notation:

```rust
fn main() {
    let rect = Rectangle { width: 10.0, height: 5.0 };
    println!("width = {}", rect.width);
    println!("height = {}", rect.height);
}
```

To mutate, the **entire binding** must be `mut` — you can't mark individual fields mutable:

```rust
fn main() {
    let mut rect = Rectangle { width: 10.0, height: 5.0 };
    rect.width = 20.0;  // ✅
    println!("{}", rect.width); // 20.0
}
```

```rust
// ❌ This is NOT valid Rust syntax:
// let rect = Rectangle { mut width: 10.0, height: 5.0 };
```

---

## 5. Field Init Shorthand

When a variable name exactly matches a field name, you can skip the repetition:

```rust
fn build_user(name: String, email: String) -> User {
    User {
        name,          // same as name: name
        email,         // same as email: email
        active: true,
        sign_in_count: 1,
    }
}
```

---

## 6. Struct Update Syntax

Copy most fields from an existing instance, override a few:

```rust
struct User {
    name: String,
    email: String,
    active: bool,
    sign_in_count: u32,
}

fn main() {
    let user1 = User {
        name: String::from("Alice"),
        email: String::from("alice@example.com"),
        active: true,
        sign_in_count: 1,
    };

    let user2 = User {
        email: String::from("bob@example.com"),
        name: String::from("Bob"),
        ..user1   // copy remaining fields from user1
    };

    // user1.active (bool, Copy) — not moved
    println!("{}", user1.active); // ✅
    // user1.name (String, not Copy) — moved into user2 if not overridden
    // But we overrode name above, so user1.name was not moved here
    println!("{}", user1.name);   // ✅ because we provided a new name for user2
}
```

⚠️ If a **non-Copy** field (like `String`) is **not overridden**, it is **moved** from the source:

```rust
fn main() {
    let u1 = User { name: String::from("Alice"), email: String::from("a@b.com"),
                    active: true, sign_in_count: 1 };

    let u2 = User {
        email: String::from("x@y.com"),
        ..u1   // u1.name is moved into u2.name here!
    };

    // println!("{}", u1.name); // ❌ u1.name was moved
    println!("{}", u1.active);   // ✅ bool is Copy
    println!("{}", u2.name);     // ✅
}
```

---

## 7. Structs and Ownership

### Owned fields — simplest case

Structs that own their data (using `String`, `Vec<T>`, etc.) have straightforward lifetime — they live as long as the struct:

```rust
struct Article {
    title: String,       // owned
    content: String,     // owned
    word_count: usize,   // Copy
}
```

When `article` is dropped, its `title` and `content` are dropped too.

### Moving a struct

A struct is moved just like any other non-Copy value:

```rust
fn print_rect(r: Rectangle) {    // takes ownership
    println!("{}×{}", r.width, r.height);
}

fn main() {
    let rect = Rectangle { width: 5.0, height: 3.0 };
    print_rect(rect);
    // println!("{}", rect.width); // ❌ rect moved
}
```

### Borrowing a struct

```rust
fn area(r: &Rectangle) -> f64 {   // borrows — doesn't take ownership
    r.width * r.height
}

fn main() {
    let rect = Rectangle { width: 5.0, height: 3.0 };
    println!("{}", area(&rect));
    println!("{}", rect.width);    // ✅ rect still valid
}
```

---

## 8. Printing Structs — `Debug` and `Display`

By default you can't `println!("{}", rect)` — you need to implement or derive a formatting trait.

### `#[derive(Debug)]` — quick programmer output

```rust
#[derive(Debug)]
struct Rectangle {
    width: f64,
    height: f64,
}

fn main() {
    let rect = Rectangle { width: 10.0, height: 5.0 };
    println!("{:?}", rect);   // Rectangle { width: 10.0, height: 5.0 }
    println!("{:#?}", rect);  // pretty-printed (multi-line)
    dbg!(&rect);              // prints to stderr with file/line info
}
```

### Implementing `Display` — human-readable output

```rust
use std::fmt;

struct Rectangle { width: f64, height: f64 }

impl fmt::Display for Rectangle {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "Rectangle({}×{})", self.width, self.height)
    }
}

fn main() {
    let rect = Rectangle { width: 10.0, height: 5.0 };
    println!("{rect}"); // Rectangle(10×5)
}
```

---

## 9. Methods on Structs — First Look

Methods are functions tied to a struct, defined inside an `impl` block. The first parameter is always some form of `self`.

```rust
struct Rectangle { width: f64, height: f64 }

impl Rectangle {
    // &self — immutable borrow (read only)
    fn area(&self) -> f64 {
        self.width * self.height
    }

    fn perimeter(&self) -> f64 {
        2.0 * (self.width + self.height)
    }

    fn is_square(&self) -> bool {
        (self.width - self.height).abs() < f64::EPSILON
    }

    // &mut self — mutable borrow (can modify fields)
    fn scale(&mut self, factor: f64) {
        self.width  *= factor;
        self.height *= factor;
    }
}

fn main() {
    let mut rect = Rectangle { width: 10.0, height: 5.0 };
    println!("area = {}", rect.area());
    println!("perimeter = {}", rect.perimeter());
    println!("is square = {}", rect.is_square());
    rect.scale(2.0);
    println!("after scale: {}×{}", rect.width, rect.height);
}
```

This is the **roadmap practice task**: a Rectangle with `area()` and `perimeter()`.

---

## 10. Associated Functions (Constructors)

Functions in `impl` that do **not** take `self` are called **associated functions**. They're called with `Type::function()` — most commonly used as constructors:

```rust
impl Rectangle {
    // Constructor — associated function
    fn new(width: f64, height: f64) -> Self {
        Rectangle { width, height }
    }

    fn square(side: f64) -> Self {
        Rectangle { width: side, height: side }
    }
}

fn main() {
    let r = Rectangle::new(10.0, 5.0);
    let s = Rectangle::square(4.0);
    println!("{}×{}", r.width, r.height);
    println!("{}×{}", s.width, s.height);
}
```

`Self` (capital S) is an alias for the type being implemented — preferred over spelling out `Rectangle` explicitly.

---

## 11. Structs vs Tuples

| | Tuple | Struct |
|---|---|---|
| Field access | `.0`, `.1`, `.2` | `.name` |
| Self-documenting | No | Yes |
| Pattern matching | `let (x, y) = t;` | `let Point { x, y } = p;` |
| Good for | Small, obvious groupings | Named, meaningful types |

```rust
// Tuple — ok for simple cases
let point = (3.0_f64, 4.0_f64);
let dist = (point.0.powi(2) + point.1.powi(2)).sqrt();

// Struct — clearer, more maintainable
struct Point { x: f64, y: f64 }
let p = Point { x: 3.0, y: 4.0 };
let dist = (p.x.powi(2) + p.y.powi(2)).sqrt();
```

---

## 12. Nested Structs

Structs can contain other structs:

```rust
#[derive(Debug)]
struct Point { x: f64, y: f64 }

#[derive(Debug)]
struct Circle {
    center: Point,
    radius: f64,
}

impl Circle {
    fn new(cx: f64, cy: f64, r: f64) -> Self {
        Circle { center: Point { x: cx, y: cy }, radius: r }
    }

    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }

    fn contains(&self, p: &Point) -> bool {
        let dx = self.center.x - p.x;
        let dy = self.center.y - p.y;
        (dx * dx + dy * dy).sqrt() <= self.radius
    }
}

fn main() {
    let c = Circle::new(0.0, 0.0, 5.0);
    println!("{:#?}", c);
    println!("area = {:.2}", c.area());
    println!("contains (3,4) = {}", c.contains(&Point { x: 3.0, y: 4.0 }));
    println!("contains (4,4) = {}", c.contains(&Point { x: 4.0, y: 4.0 }));
}
```

Accessing nested fields:

```rust
println!("{}", c.center.x); // dot-chain through nesting
```

---

## 13. Summary Cheat Sheet

```
DEFINING A STRUCT
──────────────────────────────────────────────────────────────
struct Name {
    field1: Type1,
    field2: Type2,
}

CREATING AN INSTANCE
──────────────────────────────────────────────────────────────
let x = Name { field1: val1, field2: val2 };
let y = Name { field1, field2 };   // shorthand when var = field name
let z = Name { field1: new_val, ..x }; // update syntax

FIELD ACCESS
──────────────────────────────────────────────────────────────
x.field1           read
x.field1 = val     write (x must be mut)

METHODS (inside impl block)
──────────────────────────────────────────────────────────────
fn read(&self)           borrows — cannot modify
fn modify(&mut self)     mutably borrows — can modify
fn consume(self)         takes ownership
fn new(...) -> Self      associated function (constructor)

PRINTING
──────────────────────────────────────────────────────────────
#[derive(Debug)]  →  {:?} and {:#?}
impl Display      →  {}

OWNERSHIP RULES
──────────────────────────────────────────────────────────────
Struct with owned fields (String, Vec) — moves like any type
Struct update syntax ..other moves non-Copy fields from other
Borrowing a struct field borrows from the struct
```

---

## What's Next?

**Lesson 13 — Methods & `impl`** — Deep dive into `self`, `&self`, `&mut self`, associated functions, multiple `impl` blocks, and implementing standard traits.

## Further Reading
- [The Rust Book — Ch 5: Using Structs](https://doc.rust-lang.org/book/ch05-00-structs.html)
- [Rust by Example — Structs](https://doc.rust-lang.org/rust-by-example/custom_types/structs.html)
