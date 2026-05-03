# 📘 Lesson 43 — Standard Traits (T6)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T6 · Category: 🔷 Traits  
> **Previous:** [Lesson 42 — impl Trait & dyn Trait](../lesson_42_impl_dyn_trait/lesson_42_impl_dyn_trait.md)  
> **Next:** [Lesson 44 — Operator Overloading](../lesson_44_operator_overloading/lesson_44_operator_overloading.md)  
> **Practice:** [Questions](./lesson_43_questions.md) · [Answers](./lesson_43_answers.md)  
> **Practice Task:** Implement 8 standard traits for a custom Color struct

---

## Table of Contents

1. [Display — User-Facing Output](#1-display--user-facing-output)
2. [Debug — Developer Output](#2-debug--developer-output)
3. [Clone and Copy](#3-clone-and-copy)
4. [Default — Sensible Defaults](#4-default--sensible-defaults)
5. [PartialEq and Eq](#5-partialeq-and-eq)
6. [PartialOrd and Ord](#6-partialord-and-ord)
7. [Hash](#7-hash)
8. [From and Into](#8-from-and-into)
9. [Real-World Example: Color Struct](#9-real-world-example-color-struct)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Display — User-Facing Output

`Display` controls how a type looks with `{}`:

```rust
use std::fmt;

struct Color {
    r: u8,
    g: u8,
    b: u8,
}

impl fmt::Display for Color {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "rgb({}, {}, {})", self.r, self.g, self.b)
    }
}

fn main() {
    let red = Color { r: 255, g: 0, b: 0 };
    println!("{red}");           // rgb(255, 0, 0)
    println!("Color: {red}");    // Color: rgb(255, 0, 0)
    let s = red.to_string();     // Display enables .to_string()
    println!("{s}");
}
```

`Display` **cannot** be derived — you must implement it manually because the format is a design choice.

---

## 2. Debug — Developer Output

`Debug` controls how a type looks with `{:?}` and `{:#?}`:

```rust
// Derive is the easiest way
#[derive(Debug)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let p = Point { x: 1.5, y: 2.5 };
    println!("{:?}", p);    // Point { x: 1.5, y: 2.5 }
    println!("{:#?}", p);   // pretty-printed:
    // Point {
    //     x: 1.5,
    //     y: 2.5,
    // }
}
```

### Custom Debug implementation:

```rust
use std::fmt;

struct Secret {
    username: String,
    password: String,
}

impl fmt::Debug for Secret {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("Secret")
            .field("username", &self.username)
            .field("password", &"****")  // hide password!
            .finish()
    }
}

fn main() {
    let s = Secret { username: "alice".into(), password: "hunter2".into() };
    println!("{:?}", s);  // Secret { username: "alice", password: "****" }
}
```

---

## 3. Clone and Copy

### Clone — explicit duplication:

```rust
#[derive(Debug, Clone)]
struct Config {
    name: String,
    retries: u32,
}

fn main() {
    let c1 = Config { name: "app".into(), retries: 3 };
    let c2 = c1.clone();  // explicit deep copy
    println!("{:?}", c1);  // still works — c1 wasn't moved
    println!("{:?}", c2);
}
```

### Copy — implicit bitwise copy:

```rust
#[derive(Debug, Clone, Copy)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let p1 = Point { x: 1.0, y: 2.0 };
    let p2 = p1;  // COPY, not move — p1 still valid
    println!("{:?} {:?}", p1, p2);
}
```

**Rules:**
- `Copy` requires `Clone`
- `Copy` only works if ALL fields are `Copy`
- Types with heap data (`String`, `Vec`) **cannot** be `Copy`

---

## 4. Default — Sensible Defaults

```rust
#[derive(Debug, Default)]
struct Config {
    host: String,      // defaults to ""
    port: u16,         // defaults to 0
    verbose: bool,     // defaults to false
    max_retries: u32,  // defaults to 0
}

fn main() {
    let default_config = Config::default();
    println!("{:?}", default_config);

    // Override only some fields
    let custom = Config {
        port: 8080,
        verbose: true,
        ..Config::default()
    };
    println!("{:?}", custom);
}
```

### Custom Default:

```rust
#[derive(Debug)]
struct ServerConfig {
    host: String,
    port: u16,
    max_connections: u32,
}

impl Default for ServerConfig {
    fn default() -> Self {
        ServerConfig {
            host: "localhost".into(),
            port: 8080,
            max_connections: 100,
        }
    }
}

fn main() {
    let config = ServerConfig::default();
    println!("{:?}", config);
    // ServerConfig { host: "localhost", port: 8080, max_connections: 100 }
}
```

---

## 5. PartialEq and Eq

```rust
#[derive(Debug, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let a = Point { x: 1.0, y: 2.0 };
    let b = Point { x: 1.0, y: 2.0 };
    let c = Point { x: 3.0, y: 4.0 };

    println!("a == b: {}", a == b);  // true
    println!("a == c: {}", a == c);  // false
    println!("a != c: {}", a != c);  // true
}
```

**`PartialEq`** — allows `==` and `!=`. Can have values where `a != a` (like `f64::NAN`).

**`Eq`** — marker trait guaranteeing reflexivity (`a == a` always). Required for `HashMap` keys.

```rust
#[derive(Debug, PartialEq, Eq, Hash)]  // Eq needed for HashMap key
struct UserId(u64);
```

---

## 6. PartialOrd and Ord

```rust
#[derive(Debug, PartialEq, PartialOrd)]
struct Score {
    value: f64,
}

fn main() {
    let a = Score { value: 85.0 };
    let b = Score { value: 92.0 };
    println!("a < b: {}", a < b);   // true
    println!("a > b: {}", a > b);   // false
}
```

**`Ord`** — total ordering (required for `BTreeMap` keys and `.sort()`):

```rust
#[derive(Debug, PartialEq, Eq, PartialOrd, Ord)]
struct Priority {
    level: u32,
    name: String,
}

fn main() {
    let mut tasks = vec![
        Priority { level: 2, name: "B".into() },
        Priority { level: 1, name: "A".into() },
        Priority { level: 3, name: "C".into() },
    ];
    tasks.sort();  // Ord enables sort()
    println!("{:?}", tasks);
}
```

---

## 7. Hash

Required for `HashMap`/`HashSet` keys:

```rust
use std::collections::HashSet;

#[derive(Debug, Hash, PartialEq, Eq)]
struct Student {
    id: u32,
    name: String,
}

fn main() {
    let mut students = HashSet::new();
    students.insert(Student { id: 1, name: "Alice".into() });
    students.insert(Student { id: 2, name: "Bob".into() });
    students.insert(Student { id: 1, name: "Alice".into() });  // duplicate

    println!("Unique students: {}", students.len());  // 2
}
```

**Rule:** If `a == b` then `hash(a) == hash(b)`. Always derive `Hash` together with `PartialEq + Eq`.

---

## 8. From and Into

Type conversion traits:

```rust
#[derive(Debug)]
struct Celsius(f64);

#[derive(Debug)]
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self {
        Fahrenheit(c.0 * 9.0 / 5.0 + 32.0)
    }
}

// Implementing From<Celsius> for Fahrenheit automatically gives us
// Into<Fahrenheit> for Celsius — for free!

fn main() {
    let boiling = Celsius(100.0);
    let f: Fahrenheit = Fahrenheit::from(boiling);
    println!("{:?}", f);  // Fahrenheit(212.0)

    let body = Celsius(37.0);
    let f: Fahrenheit = body.into();  // uses the auto-generated Into
    println!("{:?}", f);  // Fahrenheit(98.6)
}
```

### From<&str> for String:

```rust
fn greet(name: impl Into<String>) {
    let name: String = name.into();
    println!("Hello, {name}!");
}

fn main() {
    greet("Alice");                    // &str → String
    greet(String::from("Bob"));        // String → String
    greet("Charlie".to_string());     // already String
}
```

---

## 9. Real-World Example: Color Struct

The roadmap practice task — implementing 8 standard traits:

```rust
use std::fmt;
use std::hash::{Hash, Hasher};

#[derive(Clone, Copy, Eq)]
struct Color {
    r: u8,
    g: u8,
    b: u8,
}

// 1. Display
impl fmt::Display for Color {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "#{:02X}{:02X}{:02X}", self.r, self.g, self.b)
    }
}

// 2. Debug
impl fmt::Debug for Color {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        f.debug_struct("Color")
            .field("r", &self.r)
            .field("g", &self.g)
            .field("b", &self.b)
            .field("hex", &format!("{self}"))
            .finish()
    }
}

// 3 & 4. Clone + Copy — derived above

// 5. Default
impl Default for Color {
    fn default() -> Self { Color { r: 0, g: 0, b: 0 } }  // black
}

// 6. PartialEq
impl PartialEq for Color {
    fn eq(&self, other: &Self) -> bool {
        self.r == other.r && self.g == other.g && self.b == other.b
    }
}

// 7. Hash
impl Hash for Color {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.r.hash(state);
        self.g.hash(state);
        self.b.hash(state);
    }
}

// 8. From
impl From<(u8, u8, u8)> for Color {
    fn from((r, g, b): (u8, u8, u8)) -> Self { Color { r, g, b } }
}

impl From<u32> for Color {
    fn from(hex: u32) -> Self {
        Color {
            r: ((hex >> 16) & 0xFF) as u8,
            g: ((hex >> 8) & 0xFF) as u8,
            b: (hex & 0xFF) as u8,
        }
    }
}

fn main() {
    let red = Color { r: 255, g: 0, b: 0 };
    let blue: Color = (0, 0, 255).into();
    let green = Color::from(0x00FF00u32);
    let black = Color::default();

    println!("{red}");            // #FF0000
    println!("{:?}", blue);       // Color { r: 0, g: 0, b: 255, hex: "#0000FF" }
    println!("{green}");          // #00FF00
    println!("Default: {black}"); // #000000

    let red2 = red;               // Copy — no move
    println!("Equal: {}", red == red2);  // true

    use std::collections::HashSet;
    let mut palette = HashSet::new();
    palette.insert(red);
    palette.insert(blue);
    palette.insert(red);  // duplicate
    println!("Palette size: {}", palette.len());  // 2
}
```

---

## 10. Summary Cheat Sheet

```
FORMATTING
────────────────────────────────────────────────────────────
Display         {}      user-facing output (manual impl)
Debug           {:?}    developer output (derive or manual)

COPYING
────────────────────────────────────────────────────────────
Clone           .clone()    explicit deep copy
Copy            implicit    bitwise copy (only for simple types)

DEFAULTS
────────────────────────────────────────────────────────────
Default         ::default() sensible default values

COMPARISON
────────────────────────────────────────────────────────────
PartialEq       == !=       partial equality
Eq              (marker)    reflexive equality (a == a)
PartialOrd      < > <= >=   partial ordering
Ord             .sort()     total ordering

HASHING
────────────────────────────────────────────────────────────
Hash            HashMap/HashSet keys (always with Eq)

CONVERSION
────────────────────────────────────────────────────────────
From<T>         Type::from(val)     infallible conversion
Into<T>         val.into()          auto from From
TryFrom<T>      returns Result      fallible conversion
```

---

## What's Next?

**Lesson 44 — Operator Overloading** — Implement `+`, `-`, `*`, `[]`, and other operators for your custom types using `std::ops` traits.

## Further Reading
- [The Rust Book — Appendix: Derivable Traits](https://doc.rust-lang.org/book/appendix-03-derivable-traits.html)
- [std::fmt](https://doc.rust-lang.org/std/fmt/index.html)
- [std::convert](https://doc.rust-lang.org/std/convert/index.html)

---

*Standard traits: the language Rust types speak! 🦀*
