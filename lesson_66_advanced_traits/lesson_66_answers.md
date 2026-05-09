# ✅ Lesson 66 — Answers: Advanced Traits (AL2)

---

## Section A

### A1
```
S A B
```
`S.greet()` calls the inherent method. `A::greet(&S)` calls A's impl. `<S as B>::greet(&S)` calls B's impl.

---

## Section B

### A2
```rust
struct Celsius(f64);
struct Fahrenheit(f64);

impl From<Celsius> for Fahrenheit {
    fn from(c: Celsius) -> Self { Fahrenheit(c.0 * 9.0 / 5.0 + 32.0) }
}

impl std::fmt::Display for Celsius {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result { write!(f, "{:.1}°C", self.0) }
}

impl std::fmt::Display for Fahrenheit {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result { write!(f, "{:.1}°F", self.0) }
}

fn boil_water(temp: Celsius) { println!("Heating to {temp}"); }

fn main() {
    let c = Celsius(100.0);
    let f: Fahrenheit = c.into();
    println!("{f}");  // 212.0°F

    boil_water(Celsius(100.0));
    // boil_water(Fahrenheit(212.0));  // ❌ type mismatch!
}
```

### A3
```rust
trait VecExt {
    fn median(&self) -> f64;
}

impl VecExt for Vec<f64> {
    fn median(&self) -> f64 {
        let mut sorted = self.clone();
        sorted.sort_by(|a, b| a.partial_cmp(b).unwrap());
        let len = sorted.len();
        if len == 0 { return 0.0; }
        if len % 2 == 0 { (sorted[len / 2 - 1] + sorted[len / 2]) / 2.0 }
        else { sorted[len / 2] }
    }
}

fn main() {
    let v = vec![3.0, 1.0, 4.0, 1.0, 5.0];
    println!("Median: {}", v.median());  // 3.0
}
```

### A4
```rust
use std::marker::PhantomData;

struct Disconnected;
struct Connected;
struct Authenticated;

struct Connection<S> { addr: String, _state: PhantomData<S> }

impl Connection<Disconnected> {
    fn new(addr: &str) -> Self {
        Connection { addr: addr.into(), _state: PhantomData }
    }
    fn connect(self) -> Connection<Connected> {
        println!("Connected to {}", self.addr);
        Connection { addr: self.addr, _state: PhantomData }
    }
}

impl Connection<Connected> {
    fn authenticate(self, _token: &str) -> Connection<Authenticated> {
        println!("Authenticated");
        Connection { addr: self.addr, _state: PhantomData }
    }
}

impl Connection<Authenticated> {
    fn send_data(&self, data: &str) {
        println!("Sending to {}: {data}", self.addr);
    }
}

fn main() {
    let conn = Connection::<Disconnected>::new("server.example.com")
        .connect()
        .authenticate("secret");
    conn.send_data("hello");
    // Connection::<Disconnected>::new("x").send_data("y");  // ❌ compile error
}
```

### A5
```rust
trait Summarize {
    fn summary(&self) -> String;
}

impl<T: std::fmt::Debug> Summarize for T {
    fn summary(&self) -> String { format!("{:?}", self) }
}

fn main() {
    println!("{}", 42.summary());
    println!("{}", vec![1, 2, 3].summary());
    println!("{}", "hello".summary());
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `trait A: B` requires implementors to also implement B |
| 2 | **False** | Blanket impls have bounds — they apply to types matching those bounds |
| 3 | **False** | Newtypes are zero-cost — the wrapper is optimized away |
| 4 | **True** | The private supertrait prevents external implementations |
| 5 | **True** | That's the fully qualified disambiguation syntax |
| 6 | **True** | Extension traits add methods to any type (with the orphan rule) |

---

## 🏆 Lesson 66 Complete!

**Next up:** [Lesson 67 — Design Patterns: Builder](../lesson_67_builder_pattern/lesson_67_builder_pattern.md) 🦀
