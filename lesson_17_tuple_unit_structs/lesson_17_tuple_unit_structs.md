# 📘 Lesson 17 — Tuple Structs & Unit Structs (S6)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** S6 · Category: 🏗 Structs/Enums  
> **Previous:** [Lesson 16 — if let / while let / let else](./lesson_16_if_let.md)  
> **Next:** [Lesson 18 — String vs &str](./lesson_18_string_vs_str.md)  
> **Practice:** [Questions](./lesson_17_questions.md) · [Answers](./lesson_17_answers.md)  
> **Practice Task:** Wrap `f64` in a `Meters` newtype to prevent unit confusion

---

## Table of Contents

1. [Recap — Three Kinds of Structs](#1-recap--three-kinds-of-structs)
2. [Tuple Structs — Definition and Use](#2-tuple-structs--definition-and-use)
3. [Accessing Tuple Struct Fields](#3-accessing-tuple-struct-fields)
4. [Destructuring Tuple Structs](#4-destructuring-tuple-structs)
5. [The Newtype Pattern](#5-the-newtype-pattern)
6. [Why Newtype? Preventing Type Confusion](#6-why-newtype-preventing-type-confusion)
7. [Adding Methods to Tuple Structs](#7-adding-methods-to-tuple-structs)
8. [Implementing Traits on Newtypes](#8-implementing-traits-on-newtypes)
9. [Unit Structs — Zero-Sized Types](#9-unit-structs--zero-sized-types)
10. [Unit Structs as Marker Types](#10-unit-structs-as-marker-types)
11. [Unit Structs Implementing Traits](#11-unit-structs-implementing-traits)
12. [Memory and Size](#12-memory-and-size)
13. [When to Use Each Struct Kind](#13-when-to-use-each-struct-kind)
14. [Summary Cheat Sheet](#14-summary-cheat-sheet)

---

## 1. Recap — Three Kinds of Structs

Rust has three struct forms:

```rust
// 1. Named-field struct (most common)
struct Point { x: f64, y: f64 }

// 2. Tuple struct (this lesson)
struct Color(u8, u8, u8);

// 3. Unit struct (this lesson)
struct AlwaysEqual;
```

Each has a distinct use case.

---

## 2. Tuple Structs — Definition and Use

A **tuple struct** looks like a struct name combined with a tuple. It has no field *names* — only positions:

```rust
struct Rgb(u8, u8, u8);
struct Point2D(f64, f64);
struct Wrapper(i32);
```

Creating an instance looks like calling a function:

```rust
fn main() {
    let red    = Rgb(255, 0, 0);
    let origin = Point2D(0.0, 0.0);
    let w      = Wrapper(42);
}
```

In fact, the name `Rgb` acts as a **constructor function** — its type is `fn(u8, u8, u8) -> Rgb`. This means you can pass it to higher-order functions:

```rust
fn main() {
    let values = vec![(255u8, 0u8, 0u8), (0, 255, 0), (0, 0, 255)];
    let colours: Vec<Rgb> = values.into_iter().map(|(r, g, b)| Rgb(r, g, b)).collect();
}
```

---

## 3. Accessing Tuple Struct Fields

Fields are accessed by **index** using dot notation:

```rust
struct Rgb(u8, u8, u8);

fn main() {
    let red = Rgb(255, 128, 0);
    println!("r={} g={} b={}", red.0, red.1, red.2);
}
```

| Named struct | Tuple struct |
|---|---|
| `point.x` | `point.0` |
| `point.y` | `point.1` |

---

## 4. Destructuring Tuple Structs

Pattern matching works on tuple structs:

```rust
struct Point2D(f64, f64);

fn main() {
    let p = Point2D(3.0, 4.0);

    // Destructure in let
    let Point2D(x, y) = p;
    println!("x={x}, y={y}");

    // Destructure in match
    let p2 = Point2D(0.0, 5.0);
    match p2 {
        Point2D(0.0, y) => println!("on y-axis at {y}"),
        Point2D(x, 0.0) => println!("on x-axis at {x}"),
        Point2D(x, y)   => println!("at ({x}, {y})"),
    }

    // Destructure in function parameter
    fn length(&Point2D(x, y): &Point2D) -> f64 {
        (x * x + y * y).sqrt()
    }
    let p3 = Point2D(3.0, 4.0);
    println!("length = {}", length(&p3)); // 5
}
```

---

## 5. The Newtype Pattern

The most important use of tuple structs is the **newtype pattern**: wrapping a primitive type to create a **distinct, named type** with the same underlying representation.

```rust
struct Meters(f64);
struct Kilograms(f64);
struct Seconds(f64);
```

Even though all three wrap `f64`, they are **completely different types** — the compiler won't let you accidentally mix them:

```rust
fn speed(distance: Meters, time: Seconds) -> f64 {
    distance.0 / time.0
}

fn main() {
    let d = Meters(100.0);
    let t = Seconds(9.58);

    println!("{:.2} m/s", speed(d, t));

    // speed(t, d);  // ❌ COMPILE ERROR: wrong argument order
    //               // Seconds where Meters expected
}
```

This is the **roadmap practice task**: wrapping `f64` in `Meters` to prevent unit confusion.

---

## 6. Why Newtype? Preventing Type Confusion

### The Mars Climate Orbiter problem

In 1999, NASA lost a $327 million spacecraft because one team used metric units and another used imperial — the types were both `f64` and there was nothing to catch the mismatch.

With newtypes, this class of bug is a compile error:

```rust
struct Newtons(f64);
struct PoundForce(f64);

fn apply_thrust(force: Newtons) {
    println!("Applying {:.1} N", force.0);
}

fn main() {
    let metric_force  = Newtons(4448.0);
    let imperial_force = PoundForce(1000.0);

    apply_thrust(metric_force);
    // apply_thrust(imperial_force); // ❌ type error — catches the bug!
}
```

### Index types — preventing ID confusion

```rust
struct UserId(u64);
struct ProductId(u64);
struct OrderId(u64);

fn get_user(id: UserId) -> String {
    format!("User #{}", id.0)
}

fn main() {
    let uid = UserId(42);
    let pid = ProductId(42);

    get_user(uid);
    // get_user(pid); // ❌ ProductId ≠ UserId — compile error
}
```

Both `uid` and `pid` hold the value `42`, but they're incompatible types. This catches real bugs in large codebases.

---

## 7. Adding Methods to Tuple Structs

Use `impl` exactly like named structs:

```rust
struct Meters(f64);
struct Kilometers(f64);

impl Meters {
    fn new(val: f64) -> Self {
        assert!(val >= 0.0, "distance cannot be negative");
        Meters(val)
    }

    fn to_km(&self) -> Kilometers { Kilometers(self.0 / 1000.0) }
    fn value(&self) -> f64 { self.0 }
}

impl Kilometers {
    fn to_meters(&self) -> Meters { Meters(self.0 * 1000.0) }
    fn value(&self) -> f64 { self.0 }
}

fn main() {
    let d = Meters::new(5280.0);
    println!("{:.3} km", d.to_km().value());  // 5.280 km
    println!("{:.1} m", d.to_km().to_meters().value()); // 5280.0 m
}
```

---

## 8. Implementing Traits on Newtypes

Newtypes let you implement external traits for external types — something the orphan rule normally prevents:

```rust
use std::fmt;

// We can't implement Display for Vec<String> directly (neither type is ours)
// But we can wrap it:
struct StringList(Vec<String>);

impl fmt::Display for StringList {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let list = StringList(vec!["alpha".into(), "beta".into(), "gamma".into()]);
    println!("{list}"); // [alpha, beta, gamma]
}
```

### Deriving traits on tuple structs

```rust
#[derive(Debug, Clone, PartialEq, PartialOrd)]
struct Celsius(f64);

#[derive(Debug, Clone, PartialEq, PartialOrd)]
struct Fahrenheit(f64);

fn main() {
    let boiling = Celsius(100.0);
    let body    = Celsius(37.0);

    println!("{:?}", boiling);
    println!("boiling > body: {}", boiling > body);  // PartialOrd

    let cold = boiling.clone();
    println!("{}", cold == boiling);  // PartialEq: true
}
```

### From / Into for ergonomic conversion

```rust
struct Meters(f64);
struct Feet(f64);

impl From<Meters> for Feet {
    fn from(m: Meters) -> Feet {
        Feet(m.0 * 3.28084)
    }
}

fn main() {
    let m = Meters(10.0);
    let f: Feet = m.into();  // Into auto-derived from From
    println!("{:.2} ft", f.0); // 32.81 ft
}
```

---

## 9. Unit Structs — Zero-Sized Types

A **unit struct** has no fields at all:

```rust
struct AlwaysEqual;
struct Pending;
struct Marker;
```

They take **zero bytes** in memory. You instantiate them with just the name:

```rust
fn main() {
    let _a = AlwaysEqual;
    let _b = Pending;
    println!("size = {}", std::mem::size_of::<AlwaysEqual>()); // 0
}
```

Unit structs are types — they carry compile-time meaning even with no runtime cost.

---

## 10. Unit Structs as Marker Types

The primary use of unit structs is as **type-level markers** that encode state or capability in the type system:

### Typestate pattern — encode state in types

```rust
struct Locked;
struct Unlocked;

struct Safe<State> {
    contents: String,
    _state: std::marker::PhantomData<State>,
}

impl Safe<Locked> {
    fn new(contents: &str) -> Self {
        Safe { contents: contents.to_string(), _state: std::marker::PhantomData }
    }

    fn unlock(self, code: &str) -> Result<Safe<Unlocked>, Safe<Locked>> {
        if code == "1234" {
            Ok(Safe { contents: self.contents, _state: std::marker::PhantomData })
        } else {
            Err(self)
        }
    }
}

impl Safe<Unlocked> {
    fn open(&self) -> &str {
        &self.contents
    }

    fn lock(self) -> Safe<Locked> {
        Safe { contents: self.contents, _state: std::marker::PhantomData }
    }
}

fn main() {
    let safe = Safe::<Locked>::new("gold bars");
    // safe.open(); // ❌ compile error — can't open a Locked safe!

    let open_safe = safe.unlock("1234").unwrap();
    println!("{}", open_safe.open()); // gold bars

    // open_safe.unlock("5678"); // ❌ compile error — can't unlock Unlocked!
}
```

### Simple marker structs

```rust
struct Validated;
struct Unvalidated;

struct Email<State> {
    address: String,
    _state: std::marker::PhantomData<State>,
}

impl Email<Unvalidated> {
    fn new(address: &str) -> Self {
        Email { address: address.to_string(), _state: std::marker::PhantomData }
    }

    fn validate(self) -> Option<Email<Validated>> {
        if self.address.contains('@') {
            Some(Email { address: self.address, _state: std::marker::PhantomData })
        } else {
            None
        }
    }
}

impl Email<Validated> {
    fn send(&self, message: &str) {
        println!("Sending '{}' to {}", message, self.address);
    }
}

fn main() {
    let raw = Email::<Unvalidated>::new("alice@example.com");
    if let Some(valid) = raw.validate() {
        valid.send("Hello!");  // only valid emails can send
    }
}
```

---

## 11. Unit Structs Implementing Traits

Unit structs often implement traits to provide behaviour without needing data:

```rust
use std::fmt;

struct Separator;

impl fmt::Display for Separator {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        write!(f, "──────────────────────────")
    }
}

struct Logger;

impl Logger {
    fn log(&self, msg: &str) {
        println!("[LOG] {msg}");
    }
    fn warn(&self, msg: &str) {
        println!("[WARN] {msg}");
    }
}

fn main() {
    let sep = Separator;
    let log = Logger;

    log.log("Starting application");
    println!("{sep}");
    log.warn("Low memory");
    println!("{sep}");
}
```

---

## 12. Memory and Size

```rust
struct NamedPoint { x: f64, y: f64 }  // 16 bytes
struct TuplePoint(f64, f64);           // 16 bytes — identical layout
struct Unit;                            // 0 bytes

fn main() {
    use std::mem::size_of;
    println!("{}", size_of::<NamedPoint>()); // 16
    println!("{}", size_of::<TuplePoint>()); // 16
    println!("{}", size_of::<Unit>());        // 0
}
```

Named and tuple structs with the same fields have **identical memory layout**. The difference is purely in the source code — names vs indices.

Unit structs are **zero-sized types (ZSTs)** — they exist in the type system but have no runtime representation. Collections of ZSTs are optimised away by the compiler.

---

## 13. When to Use Each Struct Kind

| Kind | Use when |
|------|----------|
| Named struct | Fields have meaning that benefits from names; used widely |
| Tuple struct | Wrapping a single type (newtype pattern); brief, positional data |
| Unit struct | Marker type; implementing a trait with no state; type-state machines |

```rust
// ✅ Named struct — 3+ meaningful fields
struct Rectangle { width: f64, height: f64, colour: Rgb }

// ✅ Tuple struct — wrapping a value to create a distinct type
struct Metres(f64);
struct UserId(u64);

// ✅ Tuple struct — brief coordinate with obvious positions
struct Point(f64, f64);   // but struct Point { x, y } is also fine

// ✅ Unit struct — marker with no data
struct Untrusted;
struct Verified;
```

---

## 14. Summary Cheat Sheet

```
TUPLE STRUCT
────────────────────────────────────────────────────────────
struct Name(Type1, Type2);

let x = Name(val1, val2);   // construct
x.0                          // first field
x.1                          // second field
let Name(a, b) = x;          // destructure

UNIT STRUCT
────────────────────────────────────────────────────────────
struct Marker;

let m = Marker;              // no () needed
std::mem::size_of::<Marker>() == 0   // zero bytes

NEWTYPE PATTERN (key use of tuple struct)
────────────────────────────────────────────────────────────
struct Meters(f64);          // distinct type, same layout as f64
struct Seconds(f64);         // incompatible with Meters despite same inner type

fn speed(d: Meters, t: Seconds) -> f64 { d.0 / t.0 }
// speed(seconds, meters) → COMPILE ERROR — type safety enforced

ADDING METHODS
────────────────────────────────────────────────────────────
impl Meters {
    fn new(v: f64) -> Self { Meters(v) }
    fn value(&self) -> f64 { self.0 }
    fn to_km(&self) -> f64 { self.0 / 1000.0 }
}

IMPLEMENTING TRAITS
────────────────────────────────────────────────────────────
#[derive(Debug, Clone, PartialEq, PartialOrd)]
struct Celsius(f64);

impl fmt::Display for Celsius { ... }
impl From<Fahrenheit> for Celsius { ... }

UNIT STRUCT AS MARKER
────────────────────────────────────────────────────────────
struct Locked;    struct Unlocked;
struct Safe<State> { ..., _state: PhantomData<State> }
impl Safe<Locked>   { fn unlock(self) -> Safe<Unlocked> }
impl Safe<Unlocked> { fn open(&self) -> &str }
// Compiler enforces valid transitions at compile time
```

---

## What's Next?

**Lesson 18 — String vs `&str`** — The deep difference between owned heap Strings and borrowed string slices, and when to use each.

## Further Reading
- [The Rust Book — Ch 5.1: Defining Structs](https://doc.rust-lang.org/book/ch05-01-defining-structs.html)
- [Rust by Example — Tuple Structs](https://doc.rust-lang.org/rust-by-example/custom_types/structs.html)
- [Rust API Guidelines — Newtype](https://rust-lang.github.io/api-guidelines/type-safety.html#newtypes-provide-static-distinctions-c-newtype)
