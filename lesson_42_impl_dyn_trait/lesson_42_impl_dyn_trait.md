# 📘 Lesson 42 — impl Trait & dyn Trait (T5)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T5 · Category: 🔷 Traits  
> **Previous:** [Lesson 41 — Generics in Structs & Enums](../lesson_41_generic_structs/lesson_41_generic_structs.md)  
> **Next:** [Lesson 43 — Standard Traits](../lesson_43_standard_traits/lesson_43_standard_traits.md)  
> **Practice:** [Questions](./lesson_42_questions.md) · [Answers](./lesson_42_answers.md)  
> **Practice Task:** Vec\<Box\<dyn Summary\>\> polymorphic collection

---

## Table of Contents

1. [Static vs Dynamic Dispatch](#1-static-vs-dynamic-dispatch)
2. [impl Trait — Static Dispatch](#2-impl-trait--static-dispatch)
3. [dyn Trait — Dynamic Dispatch](#3-dyn-trait--dynamic-dispatch)
4. [Trait Objects with Box](#4-trait-objects-with-box)
5. [Polymorphic Collections](#5-polymorphic-collections)
6. [Object Safety](#6-object-safety)
7. [&dyn Trait vs Box\<dyn Trait\>](#7-dyn-trait-vs-boxdyn-trait)
8. [When to Use Which](#8-when-to-use-which)
9. [Real-World Example](#9-real-world-example)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Static vs Dynamic Dispatch

When you call a method through a trait, Rust needs to know which concrete implementation to run. There are two strategies:

| | Static Dispatch (`impl Trait`) | Dynamic Dispatch (`dyn Trait`) |
|---|---|---|
| Resolution | Compile time | Runtime |
| Mechanism | Monomorphization | Vtable lookup |
| Performance | Fastest (inlined) | Slight overhead |
| Flexibility | One concrete type | Multiple types at runtime |
| Binary size | Larger (copies per type) | Smaller (shared code) |

---

## 2. impl Trait — Static Dispatch

The compiler generates specialized code for each concrete type:

```rust
trait Greet {
    fn hello(&self) -> String;
}

struct Dog { name: String }
struct Cat { name: String }

impl Greet for Dog {
    fn hello(&self) -> String { format!("Woof! I'm {}", self.name) }
}
impl Greet for Cat {
    fn hello(&self) -> String { format!("Meow! I'm {}", self.name) }
}

// Static dispatch — compiler generates two versions of this function
fn greet_animal(animal: &impl Greet) {
    println!("{}", animal.hello());
}

// Equivalent explicit generic:
fn greet_animal_v2<T: Greet>(animal: &T) {
    println!("{}", animal.hello());
}

fn main() {
    let dog = Dog { name: "Rex".into() };
    let cat = Cat { name: "Whiskers".into() };

    greet_animal(&dog);  // calls greet_animal_Dog
    greet_animal(&cat);  // calls greet_animal_Cat
}
```

### Returning impl Trait:

```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct Article { title: String }
impl Summary for Article {
    fn summarize(&self) -> String { format!("Article: {}", self.title) }
}

// Returns ONE concrete type (decided at compile time)
fn make_article() -> impl Summary {
    Article { title: "Rust is great".into() }
}

fn main() {
    let item = make_article();
    println!("{}", item.summarize());
}
```

⚠️ **Limitation**: `impl Trait` in return position can only return ONE concrete type:

```rust
// ❌ Won't compile — different types in branches
// fn make_item(is_article: bool) -> impl Summary {
//     if is_article { Article { ... } }
//     else { Tweet { ... } }
// }
```

---

## 3. dyn Trait — Dynamic Dispatch

Use `dyn Trait` when you need to work with **different types at runtime**:

```rust
trait Greet {
    fn hello(&self) -> String;
}

struct Dog { name: String }
struct Cat { name: String }

impl Greet for Dog {
    fn hello(&self) -> String { format!("Woof! I'm {}", self.name) }
}
impl Greet for Cat {
    fn hello(&self) -> String { format!("Meow! I'm {}", self.name) }
}

// Dynamic dispatch — ONE function handles all types via vtable
fn greet_any(animal: &dyn Greet) {
    println!("{}", animal.hello());
}

fn main() {
    let dog = Dog { name: "Rex".into() };
    let cat = Cat { name: "Whiskers".into() };

    greet_any(&dog as &dyn Greet);
    greet_any(&cat);  // coercion happens automatically
}
```

### How vtables work:

```
┌─────────────┐     ┌──────────────────┐
│ &dyn Greet  │────→│ data pointer     │──→ actual Dog/Cat data
│             │     │ vtable pointer   │──→ ┌─────────────────┐
└─────────────┘     └──────────────────┘    │ hello: fn ptr   │
                                            │ drop:  fn ptr   │
                                            └─────────────────┘
```

A `dyn Trait` reference is a **fat pointer** — it stores both a pointer to the data and a pointer to the vtable (method lookup table).

---

## 4. Trait Objects with Box

`dyn Trait` is unsized — you can't hold it directly. Use `Box`, `&`, or `Arc`:

```rust
trait Shape {
    fn area(&self) -> f64;
    fn name(&self) -> &str;
}

struct Circle { radius: f64 }
struct Rectangle { width: f64, height: f64 }

impl Shape for Circle {
    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
    fn name(&self) -> &str { "Circle" }
}

impl Shape for Rectangle {
    fn area(&self) -> f64 { self.width * self.height }
    fn name(&self) -> &str { "Rectangle" }
}

// Return different types at runtime!
fn make_shape(kind: &str) -> Box<dyn Shape> {
    match kind {
        "circle" => Box::new(Circle { radius: 5.0 }),
        "rect" => Box::new(Rectangle { width: 4.0, height: 6.0 }),
        _ => Box::new(Circle { radius: 1.0 }),
    }
}

fn main() {
    let shape = make_shape("circle");
    println!("{}: area = {:.2}", shape.name(), shape.area());

    let shape2 = make_shape("rect");
    println!("{}: area = {:.2}", shape2.name(), shape2.area());
}
```

---

## 5. Polymorphic Collections

Store different types in the same collection:

```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct Article { title: String, author: String }
struct Tweet { username: String, content: String }
struct Podcast { title: String, episode: u32 }

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("📰 {} by {}", self.title, self.author)
    }
}
impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("🐦 @{}: {}", self.username, self.content)
    }
}
impl Summary for Podcast {
    fn summarize(&self) -> String {
        format!("🎙️ {} (ep. {})", self.title, self.episode)
    }
}

fn main() {
    // Vec of different types behind a common trait
    let feed: Vec<Box<dyn Summary>> = vec![
        Box::new(Article {
            title: "Rust 2024".into(),
            author: "Ferris".into(),
        }),
        Box::new(Tweet {
            username: "rustlang".into(),
            content: "v1.80 released!".into(),
        }),
        Box::new(Podcast {
            title: "Rustacean Station".into(),
            episode: 100,
        }),
    ];

    println!("=== Your Feed ===");
    for item in &feed {
        println!("  {}", item.summarize());
    }
}
```

---

## 6. Object Safety

Not all traits can be used as `dyn Trait`. A trait is **object-safe** if:

1. All methods have a receiver (`self`, `&self`, `&mut self`, etc.)
2. No method returns `Self`
3. No method has generic type parameters

```rust
// ✅ Object-safe
trait Drawable {
    fn draw(&self);
    fn area(&self) -> f64;
}

// ❌ NOT object-safe — returns Self
trait Clonable {
    fn clone_it(&self) -> Self;
}

// ❌ NOT object-safe — generic method
trait Converter {
    fn convert<T>(&self) -> T;
}

// ❌ NOT object-safe — no receiver (associated function)
trait Factory {
    fn create() -> Self;
}
```

### Why?
With `dyn Trait`, the compiler doesn't know the concrete type at compile time, so it can't know the size of `Self` or which generic instantiation to use.

### Workaround — use `where Self: Sized`:

```rust
trait MyTrait {
    fn normal_method(&self) -> String;

    // This method is excluded from the vtable
    fn not_object_safe(&self) -> Self where Self: Sized;
}

// Now MyTrait IS object-safe — non_object_safe just can't be called on dyn MyTrait
fn use_trait(item: &dyn MyTrait) {
    println!("{}", item.normal_method());
    // item.not_object_safe();  // ❌ can't call this on dyn
}
```

---

## 7. &dyn Trait vs Box\<dyn Trait\>

```rust
trait Speak {
    fn speak(&self) -> String;
}

struct Dog;
impl Speak for Dog {
    fn speak(&self) -> String { "Woof!".into() }
}

// &dyn Trait — borrowed, no allocation
fn borrow_speaker(s: &dyn Speak) {
    println!("{}", s.speak());
}

// Box<dyn Trait> — owned, heap allocated
fn own_speaker(s: Box<dyn Speak>) {
    println!("{}", s.speak());
}

// Return owned trait object
fn make_speaker() -> Box<dyn Speak> {
    Box::new(Dog)
}

fn main() {
    let dog = Dog;
    borrow_speaker(&dog);     // no allocation
    own_speaker(Box::new(Dog)); // heap allocated

    let speaker = make_speaker();
    println!("{}", speaker.speak());
}
```

| | `&dyn Trait` | `Box<dyn Trait>` |
|---|---|---|
| Ownership | Borrowed | Owned |
| Allocation | None | Heap |
| Use case | Temporary use | Store, return, collections |
| Lifetime | Needs `'a` | No lifetime needed |

---

## 8. When to Use Which

```
Do you know the concrete type at compile time?
├── Yes → Use impl Trait (static dispatch)
│         ✅ Fastest, inlined
│         ✅ Best for single-type scenarios
│
└── No  → Do you need different types at runtime?
          ├── Yes → Use dyn Trait (dynamic dispatch)
          │         ✅ Collections of mixed types
          │         ✅ Factory functions returning different types
          │         ✅ Plugin systems
          │
          └── Use enum if types are known and fixed
              ✅ No vtable overhead
              ✅ Pattern matching
```

---

## 9. Real-World Example

```rust
use std::fmt;

trait Logger: fmt::Debug {
    fn log(&self, message: &str);
    fn level(&self) -> &str;
}

#[derive(Debug)]
struct ConsoleLogger;

#[derive(Debug)]
struct FileLogger { path: String }

impl Logger for ConsoleLogger {
    fn log(&self, message: &str) { println!("[CONSOLE] {message}"); }
    fn level(&self) -> &str { "debug" }
}

impl Logger for FileLogger {
    fn log(&self, message: &str) {
        println!("[FILE:{}] {message}", self.path);
    }
    fn level(&self) -> &str { "info" }
}

struct App {
    loggers: Vec<Box<dyn Logger>>,
}

impl App {
    fn new() -> Self { App { loggers: vec![] } }

    fn add_logger(&mut self, logger: Box<dyn Logger>) {
        self.loggers.push(logger);
    }

    fn log_all(&self, message: &str) {
        for logger in &self.loggers {
            logger.log(message);
        }
    }
}

fn main() {
    let mut app = App::new();
    app.add_logger(Box::new(ConsoleLogger));
    app.add_logger(Box::new(FileLogger { path: "app.log".into() }));

    app.log_all("Application started");
    app.log_all("Processing data...");
}
```

---

## 10. Summary Cheat Sheet

```
STATIC DISPATCH (impl Trait)
────────────────────────────────────────────────────────────
fn f(x: &impl Trait)            parameter position
fn f() -> impl Trait             return position (one type only)
✅ Zero overhead, inlined
❌ Can't mix types

DYNAMIC DISPATCH (dyn Trait)
────────────────────────────────────────────────────────────
fn f(x: &dyn Trait)             borrowed trait object
fn f(x: Box<dyn Trait>)         owned trait object
fn f() -> Box<dyn Trait>        return different types
✅ Mix types in collections
❌ Vtable overhead, no inlining

OBJECT SAFETY RULES
────────────────────────────────────────────────────────────
✅ Methods with &self / &mut self / self
❌ Methods returning Self
❌ Methods with generic parameters
❌ Associated functions without self
Fix: add `where Self: Sized` to exclude methods

CHOOSING
────────────────────────────────────────────────────────────
Known type at compile time  → impl Trait
Different types at runtime  → dyn Trait
Fixed set of types          → enum (best performance)
```

---

## What's Next?

**Lesson 43 — Standard Traits** — Master the traits the standard library uses everywhere: `Display`, `Debug`, `Clone`, `Copy`, `Default`, `PartialEq`, `Ord`, `Hash`, and `From/Into`.

## Further Reading
- [The Rust Book — Ch 17.2: Trait Objects](https://doc.rust-lang.org/book/ch17-02-trait-objects.html)
- [Rust Reference — Object Safety](https://doc.rust-lang.org/reference/items/traits.html#object-safety)

---

*Static or dynamic — Rust gives you the choice and the control! 🦀*
