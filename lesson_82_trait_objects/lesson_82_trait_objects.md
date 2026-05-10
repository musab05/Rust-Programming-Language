# 📘 Lesson 82 — Trait Objects & Object Safety (T9)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** T9 · Category: 🧬 Traits  
> **Previous:** [Lesson 81 — HTTP Client with reqwest](../lesson_81_reqwest/lesson_81_reqwest.md)  
> **Next:** [Lesson 83 — Cow\<T\>](../lesson_83_cow/lesson_83_cow.md)  
> **Practice:** [Questions](./lesson_82_questions.md) · [Answers](./lesson_82_answers.md)  
> **Practice Task:** Design a plugin system using Box\<dyn\> trait objects

---

## Table of Contents

1. [Static vs Dynamic Dispatch](#1-static-vs-dynamic-dispatch)
2. [Trait Objects](#2-trait-objects)
3. [The Vtable](#3-the-vtable)
4. [Object Safety Rules](#4-object-safety-rules)
5. [Fixing Non-Object-Safe Traits](#5-fixing-non-object-safe-traits)
6. [&dyn vs Box\<dyn\>](#6-dyn-vs-boxdyn)
7. [Multiple Trait Bounds](#7-multiple-trait-bounds)
8. [Plugin System Example](#8-plugin-system-example)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Static vs Dynamic Dispatch

```rust
trait Greet { fn hello(&self) -> String; }
struct English;
impl Greet for English { fn hello(&self) -> String { "Hello!".into() } }
struct Spanish;
impl Greet for Spanish { fn hello(&self) -> String { "¡Hola!".into() } }

// STATIC dispatch — compiler generates separate function per type
fn greet_static(g: &impl Greet) { println!("{}", g.hello()); }

// DYNAMIC dispatch — resolved at runtime via vtable
fn greet_dynamic(g: &dyn Greet) { println!("{}", g.hello()); }

fn main() {
    greet_static(&English);  // monomorphized: greet_static_English
    greet_static(&Spanish);  // monomorphized: greet_static_Spanish

    greet_dynamic(&English); // single function, vtable lookup
    greet_dynamic(&Spanish); // same function, different vtable
}
```

| | Static (`impl Trait`) | Dynamic (`dyn Trait`) |
|---|---|---|
| Resolved at | Compile time | Runtime |
| Performance | Faster (inlined) | Slower (vtable) |
| Binary size | Larger (duplicated) | Smaller (shared) |
| Heterogeneous | ❌ One type only | ✅ Mixed types |

---

## 2. Trait Objects

A trait object is a fat pointer: **data pointer + vtable pointer**:

```rust
trait Shape {
    fn area(&self) -> f64;
    fn name(&self) -> &str;
}

struct Circle { radius: f64 }
impl Shape for Circle {
    fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius }
    fn name(&self) -> &str { "Circle" }
}

struct Rectangle { w: f64, h: f64 }
impl Shape for Rectangle {
    fn area(&self) -> f64 { self.w * self.h }
    fn name(&self) -> &str { "Rectangle" }
}

struct Triangle { base: f64, height: f64 }
impl Shape for Triangle {
    fn area(&self) -> f64 { 0.5 * self.base * self.height }
    fn name(&self) -> &str { "Triangle" }
}

fn main() {
    // Heterogeneous collection — only possible with dyn
    let shapes: Vec<Box<dyn Shape>> = vec![
        Box::new(Circle { radius: 5.0 }),
        Box::new(Rectangle { w: 4.0, h: 6.0 }),
        Box::new(Triangle { base: 3.0, height: 8.0 }),
    ];

    let total: f64 = shapes.iter().map(|s| s.area()).sum();
    for s in &shapes {
        println!("{}: area = {:.2}", s.name(), s.area());
    }
    println!("Total area: {total:.2}");

    // Size of trait object pointers
    println!("\n&dyn Shape:      {} bytes", std::mem::size_of::<&dyn Shape>());      // 16 (2 pointers)
    println!("Box<dyn Shape>:  {} bytes", std::mem::size_of::<Box<dyn Shape>>());    // 16
    println!("&Circle:         {} bytes", std::mem::size_of::<&Circle>());            // 8 (1 pointer)
}
```

---

## 3. The Vtable

```
trait object = data pointer + vtable pointer

┌──────────────┐
│ data pointer ─────→ [actual struct data on heap]
├──────────────┤
│ vtable ptr   ─────→ ┌─────────────────────┐
└──────────────┘      │ drop fn pointer     │
                      │ size                │
                      │ alignment           │
                      │ area() fn pointer   │
                      │ name() fn pointer   │
                      └─────────────────────┘
```

Each concrete type gets its OWN vtable. The vtable is created once at compile time and shared by all instances of that type.

---

## 4. Object Safety Rules

A trait is **object-safe** (can be used as `dyn Trait`) if ALL methods:

| Rule | Allowed | Not Allowed |
|---|---|---|
| `Self` in return | ❌ | `fn clone(&self) -> Self` |
| Generic parameters | ❌ | `fn process<T>(&self, t: T)` |
| `Sized` bound | ❌ | `fn f() where Self: Sized` (excluded from dyn) |
| `&self` or `&mut self` | ✅ | `fn f(self)` (consumes) |
| No `Self` in params | ✅ varies | `fn eq(&self, other: &Self)` |

```rust
// ✅ Object-safe
trait Drawable {
    fn draw(&self);
    fn name(&self) -> String;
}

// ❌ NOT object-safe — returns Self
trait Clonable {
    fn clone_me(&self) -> Self;
}

// ❌ NOT object-safe — generic method
trait Processor {
    fn process<T>(&self, item: T);
}

// ❌ NOT object-safe — requires Sized
trait SizedOnly: Sized {
    fn something(&self);
}
```

---

## 5. Fixing Non-Object-Safe Traits

### Return Self → use Box:

```rust
// ❌ Not object-safe
trait Animal {
    fn baby(&self) -> Self;
}

// ✅ Fixed: return Box<dyn Animal>
trait Animal {
    fn baby(&self) -> Box<dyn Animal>;
}
```

### Generic method → exclude from dyn:

```rust
trait Processor {
    fn name(&self) -> &str;

    // Exclude from trait object — still callable on concrete types
    fn process<T>(&self, item: T) where Self: Sized;
}

// Now dyn Processor works (process() is just not available through dyn)
fn use_processor(p: &dyn Processor) {
    println!("Processor: {}", p.name());
    // p.process(42);  // ❌ can't call — it requires Sized
}
```

### Self in parameters → use trait object:

```rust
trait Comparable {
    fn equals(&self, other: &dyn Comparable) -> bool;
}
```

---

## 6. &dyn vs Box\<dyn\>

```rust
trait Logger { fn log(&self, msg: &str); }

struct ConsoleLogger;
impl Logger for ConsoleLogger {
    fn log(&self, msg: &str) { println!("[CONSOLE] {msg}"); }
}

// &dyn — borrowed, no ownership
fn log_message(logger: &dyn Logger, msg: &str) {
    logger.log(msg);
}

// Box<dyn> — owned, on the heap
fn create_logger() -> Box<dyn Logger> {
    Box::new(ConsoleLogger)
}

// Stored in structs
struct App {
    logger: Box<dyn Logger>,  // owns the logger
}

fn main() {
    let logger = ConsoleLogger;
    log_message(&logger, "borrowed");

    let owned = create_logger();
    owned.log("owned");

    let app = App { logger: Box::new(ConsoleLogger) };
    app.logger.log("from app");
}
```

| | `&dyn Trait` | `Box<dyn Trait>` |
|---|---|---|
| Ownership | Borrowed | Owned |
| Lifetime | Must outlive reference | No lifetime constraint |
| Storage | Can't store easily | Store in structs |
| Cost | No allocation | Heap allocation |

---

## 7. Multiple Trait Bounds

```rust
use std::fmt;

trait Describable: fmt::Display + fmt::Debug {
    fn describe(&self) -> String;
}

#[derive(Debug)]
struct Item { name: String, price: f64 }

impl fmt::Display for Item {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "{} (${:.2})", self.name, self.price)
    }
}

impl Describable for Item {
    fn describe(&self) -> String { format!("Item: {self}") }
}

// Multiple trait object bounds
fn print_info(item: &(dyn Describable + Send + Sync)) {
    println!("Display:  {item}");
    println!("Debug:    {item:?}");
    println!("Describe: {}", item.describe());
}

fn main() {
    let item = Item { name: "Book".into(), price: 29.99 };
    print_info(&item);
}
```

---

## 8. Plugin System Example

```rust
trait Plugin: Send + Sync {
    fn name(&self) -> &str;
    fn version(&self) -> &str;
    fn execute(&self, input: &str) -> String;
}

struct UpperPlugin;
impl Plugin for UpperPlugin {
    fn name(&self) -> &str { "uppercase" }
    fn version(&self) -> &str { "1.0" }
    fn execute(&self, input: &str) -> String { input.to_uppercase() }
}

struct ReversePlugin;
impl Plugin for ReversePlugin {
    fn name(&self) -> &str { "reverse" }
    fn version(&self) -> &str { "1.0" }
    fn execute(&self, input: &str) -> String { input.chars().rev().collect() }
}

struct CountPlugin;
impl Plugin for CountPlugin {
    fn name(&self) -> &str { "word-count" }
    fn version(&self) -> &str { "1.0" }
    fn execute(&self, input: &str) -> String {
        format!("{} words", input.split_whitespace().count())
    }
}

struct PluginManager {
    plugins: Vec<Box<dyn Plugin>>,
}

impl PluginManager {
    fn new() -> Self { PluginManager { plugins: vec![] } }

    fn register(&mut self, plugin: Box<dyn Plugin>) {
        println!("📦 Registered: {} v{}", plugin.name(), plugin.version());
        self.plugins.push(plugin);
    }

    fn run_all(&self, input: &str) {
        println!("\nInput: \"{input}\"");
        for p in &self.plugins {
            println!("  [{}] → {}", p.name(), p.execute(input));
        }
    }
}

fn main() {
    let mut mgr = PluginManager::new();
    mgr.register(Box::new(UpperPlugin));
    mgr.register(Box::new(ReversePlugin));
    mgr.register(Box::new(CountPlugin));
    mgr.run_all("hello world from rust");
}
```

---

## 9. Summary Cheat Sheet

```
TRAIT OBJECTS
────────────────────────────────────────────────────────────
&dyn Trait           borrowed trait object (16 bytes)
Box<dyn Trait>       owned trait object (16 bytes)
= data pointer + vtable pointer

OBJECT SAFETY RULES
────────────────────────────────────────────────────────────
✅ Methods with &self / &mut self
✅ Return types that aren't Self
✅ No generic type parameters
❌ fn method() -> Self
❌ fn method<T>()
❌ trait: Sized

FIXES
────────────────────────────────────────────────────────────
Return Self      → return Box<dyn Trait>
Generic method   → add `where Self: Sized` to exclude
Self in params   → use &dyn Trait instead

WHEN TO USE
────────────────────────────────────────────────────────────
impl Trait   → one type, fast, static dispatch
dyn Trait    → multiple types, flexible, dynamic dispatch
Box<dyn>     → store in structs, return from functions
&dyn         → borrow, pass to functions
```

---

## What's Next?

**Lesson 83 — Cow\<T\> (Clone on Write)** — Avoid allocations by conditionally borrowing or owning data.

## Further Reading
- [The Rust Book — Ch 17.2: Trait Objects](https://doc.rust-lang.org/book/ch17-02-trait-objects.html)
- [Object Safety RFC](https://doc.rust-lang.org/reference/items/traits.html#object-safety)

---

*Trait objects: runtime polymorphism, Rust-style! 🦀*
