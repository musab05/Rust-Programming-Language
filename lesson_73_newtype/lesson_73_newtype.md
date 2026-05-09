# 📘 Lesson 73 — Newtype Pattern (AT2)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** AT2 · Category: 🔷 Advanced Types  
> **Previous:** [Lesson 72 — Type Aliases](../lesson_72_type_aliases/lesson_72_type_aliases.md)  
> **Next:** [Lesson 74 — Stack vs Heap](../lesson_74_stack_heap/lesson_74_stack_heap.md)  
> **Practice:** [Questions](./lesson_73_questions.md) · [Answers](./lesson_73_answers.md)  
> **Practice Task:** Newtype for validated Email address

---

## Table of Contents

1. [What Is the Newtype Pattern?](#1-what-is-the-newtype-pattern)
2. [Newtype for Type Safety](#2-newtype-for-type-safety)
3. [Newtype to Bypass the Orphan Rule](#3-newtype-to-bypass-the-orphan-rule)
4. [Validated Newtypes](#4-validated-newtypes)
5. [Transparent Access with Deref](#5-transparent-access-with-deref)
6. [Implementing Traits on Newtypes](#6-implementing-traits-on-newtypes)
7. [Cost of Newtypes](#7-cost-of-newtypes)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. What Is the Newtype Pattern?

Wrap a type in a single-field tuple struct to create a **distinct type**:

```rust
struct Meters(f64);
struct Seconds(f64);

fn speed(distance: Meters, time: Seconds) -> f64 {
    distance.0 / time.0
}

fn main() {
    let d = Meters(100.0);
    let t = Seconds(9.58);
    println!("Speed: {:.2} m/s", speed(d, t));

    // speed(t, d);  // ❌ compile error! Seconds ≠ Meters
}
```

Unlike `type Meters = f64` (just an alias), `struct Meters(f64)` is a **truly distinct type**.

---

## 2. Newtype for Type Safety

Prevent mixing up values that have the same underlying type:

```rust
struct UserId(u64);
struct OrderId(u64);
struct ProductId(u64);

fn find_order(user: UserId, order: OrderId) -> String {
    format!("User {} → Order {}", user.0, order.0)
}

fn main() {
    let user = UserId(42);
    let order = OrderId(1001);

    println!("{}", find_order(user, order));

    // find_order(order, user);  // ❌ compile error! Arguments swapped
    // find_order(UserId(42), ProductId(5));  // ❌ ProductId ≠ OrderId
}
```

### Real-world: preventing unit confusion:

```rust
struct Celsius(f64);
struct Fahrenheit(f64);
struct Kelvin(f64);

impl Celsius {
    fn to_fahrenheit(&self) -> Fahrenheit {
        Fahrenheit(self.0 * 9.0 / 5.0 + 32.0)
    }
    fn to_kelvin(&self) -> Kelvin {
        Kelvin(self.0 + 273.15)
    }
}

impl std::fmt::Display for Celsius {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{:.1}°C", self.0)
    }
}

impl std::fmt::Display for Fahrenheit {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{:.1}°F", self.0)
    }
}

fn boil_water(temp: Celsius) {
    println!("Heating water to {temp}");
}

fn main() {
    let c = Celsius(100.0);
    boil_water(c);
    // boil_water(Fahrenheit(212.0));  // ❌ wrong type!

    let c = Celsius(0.0);
    println!("{} = {}", c, c.to_fahrenheit());
}
```

---

## 3. Newtype to Bypass the Orphan Rule

Rust's orphan rule: you can't implement a foreign trait on a foreign type. Newtypes solve this:

```rust
use std::fmt;

// ❌ Can't do this — Vec and Display are both foreign
// impl fmt::Display for Vec<String> { ... }

// ✅ Wrap in a newtype — now it's YOUR type
struct Wrapper(Vec<String>);

impl fmt::Display for Wrapper {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec!["hello".into(), "world".into()]);
    println!("{w}");  // [hello, world]
}
```

---

## 4. Validated Newtypes

Enforce invariants at construction time:

```rust
#[derive(Debug, Clone)]
struct Email(String);

impl Email {
    fn new(s: &str) -> Result<Self, String> {
        if !s.contains('@') {
            return Err(format!("Invalid email: missing '@' in '{s}'"));
        }
        if s.starts_with('@') || s.ends_with('@') {
            return Err(format!("Invalid email: '@' at boundary in '{s}'"));
        }
        if !s.contains('.') {
            return Err(format!("Invalid email: missing domain in '{s}'"));
        }
        Ok(Email(s.to_lowercase()))
    }

    fn as_str(&self) -> &str { &self.0 }
    fn domain(&self) -> &str { self.0.split('@').nth(1).unwrap_or("") }
}

impl std::fmt::Display for Email {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{}", self.0)
    }
}

fn send_email(to: &Email, subject: &str) {
    // We KNOW `to` is valid — enforced by Email::new
    println!("📧 To: {to} | Subject: {subject}");
}

fn main() {
    match Email::new("alice@example.com") {
        Ok(email) => {
            println!("Domain: {}", email.domain());
            send_email(&email, "Hello!");
        }
        Err(e) => println!("Error: {e}"),
    }

    // Invalid emails caught at construction
    println!("{:?}", Email::new("no-at-sign"));
    println!("{:?}", Email::new("@bad.com"));
}
```

### More validated newtypes:

```rust
#[derive(Debug)]
struct Port(u16);

impl Port {
    fn new(p: u16) -> Result<Self, String> {
        if p == 0 { Err("Port cannot be 0".into()) }
        else { Ok(Port(p)) }
    }

    fn is_privileged(&self) -> bool { self.0 < 1024 }
    fn value(&self) -> u16 { self.0 }
}

#[derive(Debug)]
struct Percentage(f64);

impl Percentage {
    fn new(val: f64) -> Result<Self, String> {
        if !(0.0..=100.0).contains(&val) {
            Err(format!("{val} is not in 0..100"))
        } else { Ok(Percentage(val)) }
    }
}

fn main() {
    let p = Port::new(8080).unwrap();
    println!("Port {} privileged: {}", p.value(), p.is_privileged());

    let pct = Percentage::new(85.5).unwrap();
    println!("{:?}", pct);
}
```

---

## 5. Transparent Access with Deref

Make the newtype behave like its inner type when convenient:

```rust
use std::ops::Deref;

struct Name(String);

impl Name {
    fn new(s: &str) -> Result<Self, String> {
        if s.trim().is_empty() { Err("Name cannot be empty".into()) }
        else { Ok(Name(s.trim().to_string())) }
    }
}

impl Deref for Name {
    type Target = str;
    fn deref(&self) -> &str { &self.0 }
}

impl std::fmt::Display for Name {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{}", self.0)
    }
}

fn main() {
    let name = Name::new("  Alice  ").unwrap();
    println!("Name: {name}");
    println!("Length: {}", name.len());       // via Deref → &str
    println!("Upper: {}", name.to_uppercase()); // via Deref → &str
    println!("Starts with A: {}", name.starts_with('A'));
}
```

---

## 6. Implementing Traits on Newtypes

```rust
use std::ops::Add;

#[derive(Debug, Clone, Copy)]
struct Meters(f64);

#[derive(Debug, Clone, Copy)]
struct Kilometers(f64);

impl Add for Meters {
    type Output = Meters;
    fn add(self, rhs: Meters) -> Meters { Meters(self.0 + rhs.0) }
}

impl From<Kilometers> for Meters {
    fn from(km: Kilometers) -> Self { Meters(km.0 * 1000.0) }
}

impl From<Meters> for Kilometers {
    fn from(m: Meters) -> Self { Kilometers(m.0 / 1000.0) }
}

impl std::fmt::Display for Meters {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{:.1}m", self.0)
    }
}

fn main() {
    let a = Meters(500.0);
    let b = Meters(300.0);
    let total = a + b;
    println!("{a} + {b} = {total}");

    let km: Kilometers = total.into();
    println!("{total} = {:.2}km", km.0);

    let from_km: Meters = Kilometers(5.0).into();
    println!("5km = {from_km}");
}
```

---

## 7. Cost of Newtypes

Newtypes are **zero-cost** — the wrapper is optimized away:

```rust
#[repr(transparent)]  // guarantee same layout as inner type
struct Wrapper(u64);

// In memory: Wrapper and u64 are identical
// No indirection, no overhead
```

---

## 8. Summary Cheat Sheet

```
NEWTYPE PATTERN
────────────────────────────────────────────────────────────
struct MyType(InnerType);        distinct type!

USE CASES
────────────────────────────────────────────────────────────
Type safety         Meters vs Seconds vs Kilometers
Orphan rule         impl Display for Wrapper(Vec<String>)
Validation          Email::new() enforces format
Semantic meaning    UserId vs OrderId vs ProductId

TRANSPARENT ACCESS
────────────────────────────────────────────────────────────
impl Deref for MyType { type Target = Inner; ... }

COMMON TRAITS TO IMPLEMENT
────────────────────────────────────────────────────────────
Display, Debug, Clone, From, Into, Add, PartialEq

COST
────────────────────────────────────────────────────────────
Zero-cost — #[repr(transparent)] guarantees same layout

ALIAS vs NEWTYPE
────────────────────────────────────────────────────────────
type X = Y    → same type, just readable  (no safety)
struct X(Y)   → distinct type             (type safe)
```

---

## What's Next?

**Lesson 74 — Stack vs Heap** — Where Rust values live in memory, allocation costs, and when to use Box.

## Further Reading
- [Rust Design Patterns — Newtype](https://rust-unofficial.github.io/patterns/patterns/behavioural/newtype.html)
- [The Rust Book — Ch 19.4](https://doc.rust-lang.org/book/ch19-04-advanced-types.html)

---

*Newtypes: zero-cost type safety — the compiler as your guardian! 🦀*
