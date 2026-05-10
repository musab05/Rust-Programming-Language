# ✅ Lesson 92 — Answers: Observer / Event System (DP4)

---

## Section A

### A1
- **Decoupling**: Publishers don't know about subscribers — add/remove without changing publisher code.
- **Extensibility**: New observers can be added without modifying existing code.
- **Single Responsibility**: Each observer handles its own concern (logging, analytics, security).
- **Dynamic registration**: Observers can be added/removed at runtime.

---

## Section B

### A2
```rust
use std::any::{Any, TypeId};
use std::collections::HashMap;

struct Bus { h: HashMap<TypeId, Vec<Box<dyn Fn(&dyn Any)>>> }
impl Bus {
    fn new() -> Self { Bus { h: HashMap::new() } }
    fn on<E: 'static>(&mut self, f: impl Fn(&E) + 'static) {
        self.h.entry(TypeId::of::<E>()).or_default()
            .push(Box::new(move |a| { if let Some(e) = a.downcast_ref::<E>() { f(e); } }));
    }
    fn emit<E: 'static>(&self, e: E) {
        if let Some(hs) = self.h.get(&TypeId::of::<E>()) { for h in hs { h(&e); } }
    }
}

struct LoginEvent(String);
struct ErrorEvent(u32);

fn main() {
    let mut b = Bus::new();
    b.on(|e: &LoginEvent| println!("Login: {}", e.0));
    b.on(|e: &ErrorEvent| println!("Error: {}", e.0));
    b.emit(LoginEvent("alice".into()));
    b.emit(ErrorEvent(404));
}
```

### A3
```rust
use std::collections::HashMap;
struct Emitter { m: HashMap<String, Vec<Box<dyn Fn(&str)>>> }
impl Emitter {
    fn new() -> Self { Emitter { m: HashMap::new() } }
    fn on(&mut self, event: &str, f: impl Fn(&str) + 'static) {
        self.m.entry(event.into()).or_default().push(Box::new(f));
    }
    fn emit(&self, event: &str, data: &str) {
        if let Some(cbs) = self.m.get(event) { for cb in cbs { cb(data); } }
    }
}
fn main() {
    let mut e = Emitter::new();
    e.on("click", |d| println!("Clicked: {d}"));
    e.on("click", |_| println!("Analytics: click"));
    e.emit("click", "button_submit");
}
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Core benefit — publishers and subscribers are independent |
| 2 | **False** | broadcast delivers to **ALL** subscribers |
| 3 | **True** | TypeId provides a unique identifier per type |
| 4 | **True** | Closures avoid defining separate observer structs |
| 5 | **False** | Async observer works well with broadcast/mpsc channels |
| 6 | **True** | downcast_ref returns `Option<&T>` — safe and checked |

---

## 🏆 Lesson 92 Complete!

**Next up:** [Lesson 93 — Web Server with Axum](../lesson_93_axum/lesson_93_axum.md) 🦀
