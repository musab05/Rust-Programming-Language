# ✅ Lesson 75 — Answers: Zero-Cost Abstractions (P2)

---

## Section A

### A1
1. **"What you don't use, you don't pay for."** — Unused features add zero overhead.
2. **"What you do use, you couldn't hand-code any better."** — Abstractions compile to optimal machine code.

### A2
Monomorphization: the compiler generates a **separate specialized version** of a generic function for each concrete type used.

```rust
fn max<T: PartialOrd>(a: T, b: T) -> T { if a >= b { a } else { b } }

// If you call max(1, 2) and max(1.0, 2.0), compiler generates:
// fn max_i32(a: i32, b: i32) -> i32 { ... }
// fn max_f64(a: f64, b: f64) -> f64 { ... }
```

No runtime dispatch — the exact function is called directly.

---

## Section B

### A3
```rust
use std::time::Instant;

fn iter_version(data: &[i32]) -> i32 {
    data.iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .sum()
}

fn loop_version(data: &[i32]) -> i32 {
    let mut sum = 0;
    for &x in data {
        if x % 2 == 0 { sum += x * x; }
    }
    sum
}

fn main() {
    let data: Vec<i32> = (0..100_000).collect();

    let t = Instant::now();
    let r1 = iter_version(&data);
    let t1 = t.elapsed();

    let t = Instant::now();
    let r2 = loop_version(&data);
    let t2 = t.elapsed();

    assert_eq!(r1, r2);
    println!("Iterator: {r1} in {:?}", t1);
    println!("Loop:     {r2} in {:?}", t2);
    // Nearly identical performance!
}
```

### A4
```rust
trait Greet { fn hello(&self) -> String; }

struct English;
impl Greet for English { fn hello(&self) -> String { "Hello!".into() } }

struct French;
impl Greet for French { fn hello(&self) -> String { "Bonjour!".into() } }

fn greet_static(g: &impl Greet) { println!("{}", g.hello()); }
fn greet_dynamic(g: &dyn Greet) { println!("{}", g.hello()); }

fn main() {
    greet_static(&English);  // monomorphized → direct call, inlined
    greet_dynamic(&French);  // vtable lookup → indirect call

    // Static: compiler knows the exact type → inlines hello()
    // Dynamic: compiler uses vtable pointer → indirect function call
    // Static is faster but generates more code (one version per type)
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Iterator chains are optimized to loops by LLVM |
| 2 | **True** | Each type gets its own copy of the generic function |
| 3 | **False** | `dyn Trait` uses vtable lookup — it has runtime cost |
| 4 | **True** | No captures = zero-sized struct |
| 5 | **True** | It's a strong directive (compiler almost always obeys) |
| 6 | **True** | Null pointer optimization: `None` uses the null pointer |

---

## 🏆 Lesson 75 Complete!

**Next up:** [Lesson 76 — Cargo Features & Profiles](../lesson_76_cargo_features/lesson_76_cargo_features.md) 🦀
