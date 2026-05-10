# ✅ Lesson 82 — Answers: Trait Objects & Object Safety (T9)

---

## Section A

### A1
- **A** ✅ Object-safe — method takes `&self`, returns `&str`
- **B** ❌ NOT safe — returns `Self` (compiler doesn't know size of `Self` through dyn)
- **C** ❌ NOT safe — generic type parameter `T` prevents vtable construction
- **D** ✅ Object-safe — both methods take `&self`/`&mut self`, no generics or Self return

---

## Section B

### A2
```rust
trait Plugin: Send {
    fn name(&self) -> &str;
    fn version(&self) -> &str;
    fn execute(&self, input: &str) -> String;
}

struct Upper; struct Lower; struct Reverse;
impl Plugin for Upper { fn name(&self) -> &str { "upper" } fn version(&self) -> &str { "1.0" } fn execute(&self, s: &str) -> String { s.to_uppercase() } }
impl Plugin for Lower { fn name(&self) -> &str { "lower" } fn version(&self) -> &str { "1.0" } fn execute(&self, s: &str) -> String { s.to_lowercase() } }
impl Plugin for Reverse { fn name(&self) -> &str { "rev" } fn version(&self) -> &str { "1.0" } fn execute(&self, s: &str) -> String { s.chars().rev().collect() } }

struct Manager { plugins: Vec<Box<dyn Plugin>> }
impl Manager {
    fn new() -> Self { Manager { plugins: vec![] } }
    fn add(&mut self, p: Box<dyn Plugin>) { self.plugins.push(p); }
    fn run(&self, input: &str) {
        for p in &self.plugins { println!("[{}] {}", p.name(), p.execute(input)); }
    }
}

fn main() {
    let mut m = Manager::new();
    m.add(Box::new(Upper)); m.add(Box::new(Lower)); m.add(Box::new(Reverse));
    m.run("Hello World");
}
```

### A3
```rust
use std::f64::consts::PI;
trait Shape { fn area(&self) -> f64; fn name(&self) -> &str; }
struct Circle(f64); struct Square(f64); struct Triangle(f64, f64);
impl Shape for Circle { fn area(&self) -> f64 { PI * self.0 * self.0 } fn name(&self) -> &str { "Circle" } }
impl Shape for Square { fn area(&self) -> f64 { self.0 * self.0 } fn name(&self) -> &str { "Square" } }
impl Shape for Triangle { fn area(&self) -> f64 { 0.5 * self.0 * self.1 } fn name(&self) -> &str { "Triangle" } }

fn main() {
    let shapes: Vec<Box<dyn Shape>> = vec![Box::new(Circle(5.0)), Box::new(Square(4.0)), Box::new(Triangle(6.0, 3.0))];
    let total: f64 = shapes.iter().map(|s| { println!("{}: {:.2}", s.name(), s.area()); s.area() }).sum();
    println!("Total: {total:.2}");
}
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Fat pointer = data ptr (8B) + vtable ptr (8B) = 16 bytes |
| 2 | **False** | Returning `Self` makes it non-object-safe |
| 3 | **True** | Box provides heap ownership of the trait object |
| 4 | **True** | Static dispatch is monomorphized and inlined |
| 5 | **True** | Compiler rejects `dyn NonObjectSafeTrait` |
| 6 | **False** | Multiple unrelated traits need a supertrait; only auto-traits like Send/Sync can be added |

---

## 🏆 Lesson 82 Complete!

**Next up:** [Lesson 83 — Cow\<T\>](../lesson_83_cow/lesson_83_cow.md) 🦀
