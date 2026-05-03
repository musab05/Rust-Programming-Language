# ✅ Lesson 49 — Answers: Fn, FnMut, FnOnce (CL2)

---

## Section A

### A1
- `a = || println!("hello")` — **Fn** (no captures, just prints)
- `b = || { count += 1; }` — **FnMut** (mutates `count`)
- `c = || drop(name)` — **FnOnce** (consumes `name`)

### A2 — ❌ Won't compile
`FnOnce` takes `self` — after the first `f()`, the closure is consumed. The second `f()` can't work. Error: `use of moved value: f`.

### A3 — ✅ Compiles
`Fn` takes `&self` — the closure is borrowed each time. `println!("hi")` captures nothing, so it implements `Fn`. Output: `hi` printed twice.

---

## Section B

### A4
```rust
fn run_once(f: impl FnOnce() -> String) -> String { f() }

fn run_mut(mut f: impl FnMut(i32) -> i32, values: &[i32]) -> Vec<i32> {
    values.iter().map(|&v| f(v)).collect()
}

fn run_pure(f: impl Fn(i32) -> i32, x: i32) -> i32 { f(f(x)) }

fn main() {
    // FnOnce — consumes a captured String
    let name = String::from("world");
    let result = run_once(move || format!("Hello, {name}!"));
    println!("{result}");

    // FnMut — accumulates state
    let mut total = 0;
    let sums = run_mut(|x| { total += x; total }, &[1, 2, 3, 4]);
    println!("{:?}", sums);  // [1, 3, 6, 10]

    // Fn — pure function, no side effects
    let result = run_pure(|x| x * 2, 3);
    println!("{result}");  // 12
}
```

### A5
```rust
fn make_multiplier(factor: i32) -> impl Fn(i32) -> i32 {
    move |x| x * factor
}

fn main() {
    let double = make_multiplier(2);
    let triple = make_multiplier(3);
    println!("{}", double(5));   // 10
    println!("{}", triple(5));   // 15
}
```

### A6
```rust
struct EventBus {
    handlers: Vec<Box<dyn Fn(&str)>>,
}

impl EventBus {
    fn new() -> Self { EventBus { handlers: vec![] } }

    fn subscribe(&mut self, handler: impl Fn(&str) + 'static) {
        self.handlers.push(Box::new(handler));
    }

    fn emit(&self, event: &str) {
        for handler in &self.handlers {
            handler(event);
        }
    }
}

fn main() {
    let mut bus = EventBus::new();
    bus.subscribe(|e| println!("[Logger] Event: {e}"));
    bus.subscribe(|e| println!("[Audit]  Event: {e}"));
    bus.emit("user_login");
    bus.emit("page_view");
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `FnOnce` is the MOST PERMISSIVE — all closures implement it |
| 2 | **True** | `Fn ⊂ FnMut ⊂ FnOnce` — Fn implies both others |
| 3 | **True** | `call_mut(&mut self)` requires mutable access to the closure |
| 4 | **False** | Must use `-> impl Fn(...)` or `-> Box<dyn Fn(...)>` |
| 5 | **True** | `Box<dyn Fn(T)>` enables dynamic dispatch for closure storage |

### A8
`FnOnce` is the most **permissive** — it accepts ANY closure (Fn, FnMut, or FnOnce). If you only call the closure once, requiring `Fn` or `FnMut` unnecessarily restricts callers. For example, a closure that consumes a captured `String` couldn't be passed to `impl Fn()`, but it works fine with `impl FnOnce()`. Using `FnOnce` maximizes the types of closures your function can accept.

---

## 🏆 Lesson 49 Complete!

**Next up:** [Lesson 50 — Higher-Order Functions](../lesson_50_higher_order/lesson_50_higher_order.md) 🦀
