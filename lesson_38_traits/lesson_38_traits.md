# 📘 Lesson 38 — Traits: Definition & Implementation (T1)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T1 · Category: 🔷 Traits  
> **Previous:** [Lesson 37 — anyhow & error-stack](../lesson_37_anyhow/lesson_37_anyhow.md)  
> **Next:** [Lesson 39 — Trait Bounds](../lesson_39_trait_bounds/lesson_39_trait_bounds.md)  
> **Practice:** [Questions](./lesson_38_questions.md) · [Answers](./lesson_38_answers.md)  
> **Practice Task:** Define a Summary trait; implement for Tweet and Article

---

## Table of Contents

1. [What Are Traits?](#1-what-are-traits)
2. [Defining a Trait](#2-defining-a-trait)
3. [Implementing a Trait](#3-implementing-a-trait)
4. [Default Method Implementations](#4-default-method-implementations)
5. [Traits as Parameters](#5-traits-as-parameters)
6. [Returning Types That Implement Traits](#6-returning-types-that-implement-traits)
7. [The Orphan Rule](#7-the-orphan-rule)
8. [Multiple Traits on One Type](#8-multiple-traits-on-one-type)
9. [Trait Inheritance (Supertraits)](#9-trait-inheritance-supertraits)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Are Traits?

A **trait** defines shared behavior — a contract that types can implement. Think of traits as Rust's version of interfaces.

```rust
// "Any type that implements Summary can be summarized"
trait Summary {
    fn summarize(&self) -> String;
}
```

Traits are Rust's primary tool for:
- **Polymorphism** — treating different types uniformly
- **Code reuse** — shared behavior across types
- **Abstraction** — programming against interfaces, not concrete types

---

## 2. Defining a Trait

```rust
trait Shape {
    // Required methods — implementors MUST provide these
    fn area(&self) -> f64;
    fn perimeter(&self) -> f64;
    fn name(&self) -> &str;
}
```

Traits can have:
- **Required methods** — signature only, no body
- **Default methods** — with a body (can be overridden)
- **Associated types** and **constants** (covered in later lessons)

---

## 3. Implementing a Trait

```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct Article {
    title: String,
    author: String,
    content: String,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}, by {} — {}...", self.title, self.author, &self.content[..20.min(self.content.len())])
    }
}

struct Tweet {
    username: String,
    content: String,
    retweets: u32,
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("@{}: {} ({} retweets)", self.username, self.content, self.retweets)
    }
}

fn main() {
    let article = Article {
        title: "Rust 2024".into(),
        author: "Ferris".into(),
        content: "Rust continues to grow in popularity across industries...".into(),
    };

    let tweet = Tweet {
        username: "rustlang".into(),
        content: "Rust 1.80 is out!".into(),
        retweets: 5000,
    };

    println!("{}", article.summarize());
    println!("{}", tweet.summarize());
}
```

---

## 4. Default Method Implementations

Provide a default body that implementors can use or override:

```rust
trait Summary {
    fn summarize_author(&self) -> String;

    // Default implementation — uses summarize_author()
    fn summarize(&self) -> String {
        format!("(Read more from {}...)", self.summarize_author())
    }
}

struct Article {
    title: String,
    author: String,
}

// Uses the default summarize()
impl Summary for Article {
    fn summarize_author(&self) -> String {
        self.author.clone()
    }
}

struct Tweet {
    username: String,
    content: String,
}

// Overrides the default
impl Summary for Tweet {
    fn summarize_author(&self) -> String {
        format!("@{}", self.username)
    }

    fn summarize(&self) -> String {
        format!("{}: {}", self.summarize_author(), self.content)
    }
}

fn main() {
    let article = Article { title: "News".into(), author: "Alice".into() };
    let tweet = Tweet { username: "bob".into(), content: "Hello!".into() };

    println!("{}", article.summarize());  // (Read more from Alice...)
    println!("{}", tweet.summarize());    // @bob: Hello!
}
```

---

## 5. Traits as Parameters

Use traits to accept any type that implements the trait:

```rust
trait Summary {
    fn summarize(&self) -> String;
}

// Syntax 1: impl Trait (most common for simple cases)
fn notify(item: &impl Summary) {
    println!("Breaking: {}", item.summarize());
}

// Syntax 2: Trait bound (more flexible)
fn notify_bound<T: Summary>(item: &T) {
    println!("Breaking: {}", item.summarize());
}

// Multiple parameters must be same type with trait bound:
fn compare<T: Summary>(a: &T, b: &T) {
    println!("A: {}", a.summarize());
    println!("B: {}", b.summarize());
}

// Different types OK with impl Trait:
fn notify_two(a: &impl Summary, b: &impl Summary) {
    println!("{}", a.summarize());
    println!("{}", b.summarize());
}
```

---

## 6. Returning Types That Implement Traits

```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct Tweet { username: String, content: String }

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("@{}: {}", self.username, self.content)
    }
}

// Return type implements Summary (but must be ONE concrete type)
fn create_tweet() -> impl Summary {
    Tweet {
        username: "rustlang".into(),
        content: "Hello from Rust!".into(),
    }
}

fn main() {
    let item = create_tweet();
    println!("{}", item.summarize());
}
```

⚠️ **Limitation**: `impl Trait` return can only return ONE concrete type. You can't conditionally return different types:

```rust
// ❌ Won't compile — returns different types
// fn pick(flag: bool) -> impl Summary {
//     if flag { Tweet { ... } }
//     else { Article { ... } }
// }

// ✅ Use Box<dyn Summary> for this (Lesson 39+)
```

---

## 7. The Orphan Rule

You can implement a trait for a type **only if** either the trait or the type is defined in your crate:

```rust
// ✅ Your trait on a foreign type
trait Greet {
    fn greet(&self) -> String;
}

impl Greet for String {
    fn greet(&self) -> String {
        format!("Hello, {self}!")
    }
}

// ✅ Foreign trait on your type
struct MyType;

impl std::fmt::Display for MyType {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "MyType")
    }
}

// ❌ Foreign trait on foreign type — NOT ALLOWED
// impl std::fmt::Display for Vec<i32> { ... }
```

### Workaround — the Newtype pattern:

```rust
struct Wrapper(Vec<String>);

impl std::fmt::Display for Wrapper {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "[{}]", self.0.join(", "))
    }
}

fn main() {
    let w = Wrapper(vec!["hello".into(), "world".into()]);
    println!("{w}");  // [hello, world]
}
```

---

## 8. Multiple Traits on One Type

A type can implement many traits:

```rust
use std::fmt;

trait Greet {
    fn greet(&self) -> String;
}

trait Farewell {
    fn farewell(&self) -> String;
}

struct Person {
    name: String,
}

impl Greet for Person {
    fn greet(&self) -> String {
        format!("Hello, I'm {}!", self.name)
    }
}

impl Farewell for Person {
    fn farewell(&self) -> String {
        format!("Goodbye from {}!", self.name)
    }
}

impl fmt::Display for Person {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Person({})", self.name)
    }
}

fn main() {
    let p = Person { name: "Alice".into() };
    println!("{}", p.greet());
    println!("{}", p.farewell());
    println!("{p}");
}
```

---

## 9. Trait Inheritance (Supertraits)

A trait can require another trait:

```rust
use std::fmt;

// PrintableSummary requires Display
trait PrintableSummary: fmt::Display {
    fn print_summary(&self) {
        println!("Summary: {self}");  // can use Display because it's required
    }
}

struct Report {
    title: String,
    pages: u32,
}

impl fmt::Display for Report {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "'{}' ({} pages)", self.title, self.pages)
    }
}

impl PrintableSummary for Report {}  // uses default print_summary

fn main() {
    let r = Report { title: "Annual".into(), pages: 42 };
    r.print_summary();  // Summary: 'Annual' (42 pages)
}
```

---

## 10. Summary Cheat Sheet

```
DEFINING TRAITS
────────────────────────────────────────────────────────────
trait MyTrait {
    fn required(&self);              required method
    fn default(&self) { ... }        default implementation
}

IMPLEMENTING TRAITS
────────────────────────────────────────────────────────────
impl MyTrait for MyType {
    fn required(&self) { ... }       must implement required methods
}

TRAITS AS PARAMETERS
────────────────────────────────────────────────────────────
fn f(item: &impl Trait)              impl Trait syntax
fn f<T: Trait>(item: &T)             trait bound syntax
fn f(a: &impl A, b: &impl B)        different types OK

RETURNING TRAITS
────────────────────────────────────────────────────────────
fn f() -> impl Trait                 one concrete type only

ORPHAN RULE
────────────────────────────────────────────────────────────
Can implement if: your trait OR your type (or both)
Workaround: newtype pattern

SUPERTRAITS
────────────────────────────────────────────────────────────
trait Child: Parent                  Child requires Parent
```

---

## What's Next?

**Lesson 39 — Trait Bounds** — Constrain generic types with traits. Master `T: Trait` syntax, multiple bounds, `where` clauses, and conditional implementations.

## Further Reading
- [The Rust Book — Ch 10.2: Traits](https://doc.rust-lang.org/book/ch10-02-traits.html)
- [Rust by Example — Traits](https://doc.rust-lang.org/rust-by-example/trait.html)

---

*Traits: the heart of Rust's polymorphism! 🦀*
