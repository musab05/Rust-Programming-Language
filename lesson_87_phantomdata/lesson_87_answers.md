# ✅ Lesson 87 — Answers: PhantomData (AT5)

---

## Section A

### A1 — ❌ Won't compile
Error: "parameter `T` is never used". Fix: add `_marker: PhantomData<T>`.

---

## Section B

### A2
```rust
use std::marker::PhantomData;
use std::ops::Add;

struct Meters; struct Kilograms; struct Seconds;

#[derive(Debug, Clone, Copy)]
struct Quantity<U> { value: f64, _u: PhantomData<U> }
impl<U> Quantity<U> { fn new(v: f64) -> Self { Quantity { value: v, _u: PhantomData } } }
impl<U> Add for Quantity<U> {
    type Output = Self;
    fn add(self, rhs: Self) -> Self { Quantity::new(self.value + rhs.value) }
}

fn main() {
    let d = Quantity::<Meters>::new(10.0) + Quantity::<Meters>::new(5.0);
    println!("{:.1}m", d.value);  // 15.0m
    // Quantity::<Meters>::new(1.0) + Quantity::<Seconds>::new(2.0);  // ❌ compile error
}
```

### A3
```rust
use std::marker::PhantomData;
struct Disconnected; struct Connected; struct Authenticated;

struct Connection<S> { host: String, _s: PhantomData<S> }

impl Connection<Disconnected> {
    fn new(host: &str) -> Self { Connection { host: host.into(), _s: PhantomData } }
    fn connect(self) -> Connection<Connected> {
        println!("Connected to {}", self.host);
        Connection { host: self.host, _s: PhantomData }
    }
}
impl Connection<Connected> {
    fn authenticate(self, _token: &str) -> Connection<Authenticated> {
        println!("Authenticated");
        Connection { host: self.host, _s: PhantomData }
    }
}
impl Connection<Authenticated> {
    fn query(&self, sql: &str) -> String { format!("Result of: {sql}") }
}

fn main() {
    let conn = Connection::<Disconnected>::new("db.example.com")
        .connect()
        .authenticate("token123");
    println!("{}", conn.query("SELECT 1"));
    // conn_disconnected.query("...");  // ❌ can't query without auth
}
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | PhantomData is zero-sized — no runtime cost |
| 2 | **True** | Rust requires all type params to be used (or marked with PhantomData) |
| 3 | **True** | Affects drop check and variance |
| 4 | **True** | `PhantomData<&'a T>` binds lifetime `'a` to the struct |
| 5 | **True** | PhantomData holds the state type parameter |
| 6 | **False** | PhantomData works with `?Sized` types too |

---

## 🏆 Lesson 87 Complete!

**Next up:** [Lesson 88 — Benchmarking with Criterion](../lesson_88_benchmarking/lesson_88_benchmarking.md) 🦀
