# ✅ Lesson 42 — Answers: impl Trait & dyn Trait (T5)

---

## Section A

### A1 — ❌ Won't compile
`impl Trait` in return position can only return ONE concrete type. The `if/else` returns `Cat` or `Dog` — two different types.

### A2 — ✅ Compiles
`Box<dyn Speak>` allows returning different concrete types at runtime via dynamic dispatch.

### A3 — ❌ Won't compile
`Factory` has a method `create() -> Self` which makes it NOT object-safe. `Self` is unsized behind `dyn`, so the compiler can't construct it.

---

## Section B

### A4
```rust
trait Summary { fn summarize(&self) -> String; }

struct Article { title: String }
struct Tweet { user: String, text: String }
struct Video { title: String, duration: u32 }

impl Summary for Article {
    fn summarize(&self) -> String { format!("📰 {}", self.title) }
}
impl Summary for Tweet {
    fn summarize(&self) -> String { format!("🐦 @{}: {}", self.user, self.text) }
}
impl Summary for Video {
    fn summarize(&self) -> String { format!("🎬 {} ({}min)", self.title, self.duration) }
}

fn main() {
    let feed: Vec<Box<dyn Summary>> = vec![
        Box::new(Article { title: "Rust Tips".into() }),
        Box::new(Tweet { user: "ferris".into(), text: "Hello!".into() }),
        Box::new(Video { title: "Intro to Rust".into(), duration: 15 }),
    ];
    for item in &feed { println!("{}", item.summarize()); }
}
```

### A5
```rust
trait Shape { fn area(&self) -> f64; fn name(&self) -> &str; }
struct Circle { r: f64 }
struct Rect { w: f64, h: f64 }
impl Shape for Circle {
    fn area(&self) -> f64 { std::f64::consts::PI * self.r * self.r }
    fn name(&self) -> &str { "Circle" }
}
impl Shape for Rect {
    fn area(&self) -> f64 { self.w * self.h }
    fn name(&self) -> &str { "Rectangle" }
}

fn create_shape(kind: &str) -> Box<dyn Shape> {
    match kind {
        "circle" => Box::new(Circle { r: 5.0 }),
        _ => Box::new(Rect { w: 4.0, h: 6.0 }),
    }
}

fn main() {
    let s = create_shape("circle");
    println!("{}: {:.2}", s.name(), s.area());
}
```

### A6
```rust
trait Plugin {
    fn name(&self) -> &str;
    fn execute(&self);
}

struct Logger;
impl Plugin for Logger {
    fn name(&self) -> &str { "Logger" }
    fn execute(&self) { println!("[Logger] Writing log entry..."); }
}

struct Notifier;
impl Plugin for Notifier {
    fn name(&self) -> &str { "Notifier" }
    fn execute(&self) { println!("[Notifier] Sending notification..."); }
}

fn main() {
    let plugins: Vec<Box<dyn Plugin>> = vec![
        Box::new(Logger),
        Box::new(Notifier),
    ];
    for p in &plugins {
        println!("Running plugin: {}", p.name());
        p.execute();
    }
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `dyn Trait` stores a vtable pointer for runtime method lookup |
| 2 | **False** | `impl Trait` return can only return one concrete type |
| 3 | **False** | Returning `Self` makes it NOT object-safe |
| 4 | **True** | `Box<dyn Trait>` is a fat pointer: data ptr + vtable ptr |
| 5 | **False** | Dynamic dispatch is needed for heterogeneous collections/plugins |
| 6 | **False** | `&dyn Trait` borrows existing data — no heap allocation needed |

### A8
Use `enum` when the set of types is **fixed and known**. Example: a shape system with only `Circle`, `Rectangle`, `Triangle` — use an enum with `match`. Benefits: no vtable overhead, exhaustive pattern matching, no `Box` allocation. Use `dyn Trait` when types are **open-ended** (plugins, user-defined types).

---

## 🏆 Lesson 42 Complete!

**Next up:** [Lesson 43 — Standard Traits](../lesson_43_standard_traits/lesson_43_standard_traits.md) 🦀
