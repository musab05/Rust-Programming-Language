# ✅ Lesson 48 — Answers: Closures (CL1)

---

## Section A

### A1 — ✅ Compiles
Closure borrows `x` by `&i32` (only reads it). After the closure runs, `x` is still available.  
Output: `15` then `x = 10`.

### A2 — ❌ Won't compile
`move` transfers ownership of `name` into the closure. The final `println!("{name}")` tries to use `name` after it was moved. Error: `value used after move`.

### A3 — ❌ Won't compile
The closure mutably borrows `v`. After calling `push()` twice, the `println!` tries to use `v`, but the mutable borrow by the closure is still active. Fix: drop the closure before using `v`, or restructure.

Actually — this **does compile** in modern Rust editions because the last use of `push` is before the `println!`, so the borrow ends. Output: `[1, 2, 3, 4, 4]`.

---

## Section B

### A4
```rust
fn filter_by<T, F>(items: &[T], predicate: F) -> Vec<&T>
where F: Fn(&T) -> bool,
{
    items.iter().filter(|item| predicate(item)).collect()
}

fn main() {
    let nums: Vec<i32> = (1..=10).collect();

    // 1. Even numbers
    let evens = filter_by(&nums, |x| *x % 2 == 0);
    println!("Evens: {:?}", evens);

    // 2. Words longer than 4 chars
    let words = vec!["hi", "hello", "hey", "greetings", "yo"];
    let long = filter_by(&words, |w| w.len() > 4);
    println!("Long: {:?}", long);

    // 3. With captured threshold
    let threshold = 7;
    let above = filter_by(&nums, |x| **x > threshold);
    println!("Above {threshold}: {:?}", above);
}
```

### A5
```rust
fn main() {
    let mut count = 0;
    let mut counter = || {
        count += 1;
        count
    };

    println!("{}", counter());  // 1
    println!("{}", counter());  // 2
    println!("{}", counter());  // 3
}
```

### A6
```rust
use std::thread;

fn main() {
    let data = vec![1, 2, 3, 4, 5];

    let handle = thread::spawn(move || {
        println!("Thread has: {:?}", data);
    });

    handle.join().unwrap();
}
```
`move` is necessary because the thread may outlive the current function scope. Without `move`, the closure would borrow `data`, but `data` might be dropped before the thread finishes — a dangling reference. `move` transfers ownership to the thread, ensuring the data lives long enough.

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | That's the key difference — closures capture, functions don't |
| 2 | **False** | `move` with `Copy` types copies them; they're still available outside |
| 3 | **True** | Once types are inferred from first use, they're fixed |
| 4 | **True** | `Copy` types are copied into the closure, not moved |
| 5 | **True** | Each closure has a unique anonymous type, even if signatures match |
| 6 | **False** | Closure types are anonymous — you must use `impl Fn(...)` or generics |

### A8
Rust automatically chooses the least restrictive capture mode:
1. **`&T` (immutable borrow)** — when the closure only reads the variable
2. **`&mut T` (mutable borrow)** — when the closure modifies the variable
3. **`T` (move/ownership)** — when the closure consumes the variable (e.g., drops it, moves it elsewhere)

Rust analyzes HOW the closure USES each variable to decide. The `move` keyword overrides this, forcing ownership for ALL captures.

---

## 🏆 Lesson 48 Complete!

**Next up:** [Lesson 49 — Fn, FnMut, FnOnce](../lesson_49_fn_traits/lesson_49_fn_traits.md) 🦀
