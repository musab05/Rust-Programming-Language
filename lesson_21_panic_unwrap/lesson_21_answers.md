# ✅ Lesson 21 — Answers: panic! & unwrap (E1)

---

## Section A

### A1
```
20
0
None
```
`v.get(1)` → `Some(&20)` → unwrap → 20. `v.get(5)` → `None` → `unwrap_or(&0)` → 0. `v.get(99)` → `None`.

### A2
```
42
0
42
```
`x` is `Some(42)` → `unwrap_or` returns inner value. `y` is `None` → returns default `0`. `x.expect(...)` succeeds → 42.

### A3
```
ok: 1
err: bad
ok: 3
```
Simple match on each `Result`. No panics — we handle both arms.

### A4
```
123 3.14 99 -1
```
All parse successfully except `"abc"` → `unwrap_or(-1)` returns `-1`.

---

## Section B

### A5 — ✅ Compiles, ❌ Panics at runtime
`v[0]` on an empty vec panics with `index out of bounds: the len is 0 but the index is 0`. Fix: use `v.get(0)`.

### A6 — ✅ Compiles, ❌ Panics at runtime
`"hello"` has 5 chars. `nth(10)` returns `None`. `.unwrap()` panics. Fix: use `.unwrap_or('?')` or handle `Option`.

### A7 — ✅ Compiles, ✅ Runs
All assertions pass: 2+2==4, "rust"≠"python", true is true. Output: `All assertions passed!`

### A8 — ✅ Compiles, ✅ Runs
```
Error occurred: oops
val = -1
```
`unwrap_or_else` calls the closure on `Err`. The closure prints the error message and returns `-1`.

---

## Section C

### A9 — Remove all panics
```rust
fn main() {
    let numbers = vec![10, 20, 30];

    let first = numbers.get(0).copied().unwrap_or(0);
    let tenth = numbers.get(9).copied().unwrap_or(0);
    let parsed: i32 = "abc".parse().unwrap_or(0);

    println!("{first} {tenth} {parsed}"); // 10 0 0
}
```

### A10 — Use expect instead of unwrap
```rust
use std::collections::HashMap;

fn main() {
    let mut scores: HashMap<&str, i32> = HashMap::new();
    scores.insert("Alice", 95);

    let alice_score = scores.get("Alice")
        .expect("Alice should be in the scores map");

    let parsed_score: i32 = "88".parse()
        .expect("'88' should be a valid i32");

    let env_val = std::env::var("MY_APP_CONFIG")
        .expect("MY_APP_CONFIG environment variable must be set");

    println!("{alice_score} {parsed_score} {env_val}");
}
```

### A11 — Fix the assertion
```rust
fn double(x: i32) -> i32 {
    x + x
}

fn main() {
    assert_eq!(double(5), 10, "double(5) should be 10");  // fix: 11 → 10
    assert_eq!(double(0), 0);
    assert_eq!(double(-3), -6);
    println!("All tests passed!");
}
```
Bug: `double(5)` returns 10 but the assertion expected 11. Fix the expected value to 10.

---

## Section D

### A12 — Safe division function
```rust
fn safe_divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None
    } else {
        Some(a / b)
    }
}

fn main() {
    // Using unwrap_or
    println!("{}", safe_divide(10.0, 3.0).unwrap_or(0.0));  // 3.333...
    println!("{}", safe_divide(10.0, 0.0).unwrap_or(0.0));  // 0.0

    // Using match
    match safe_divide(100.0, 7.0) {
        Some(result) => println!("100 / 7 = {result:.4}"),
        None => println!("Cannot divide by zero"),
    }

    match safe_divide(42.0, 0.0) {
        Some(result) => println!("42 / 0 = {result}"),
        None => println!("Cannot divide by zero"),  // ← this runs
    }
}
```

### A13 — Panic-free Vec accessor
```rust
fn get_or_default(v: &[i32], index: usize, default: i32) -> i32 {
    v.get(index).copied().unwrap_or(default)
    // Or equivalently:
    // match v.get(index) {
    //     Some(&val) => val,
    //     None => default,
    // }
}

fn main() {
    let empty: Vec<i32> = vec![];
    let nums = vec![10, 20, 30];

    println!("{}", get_or_default(&empty, 0, -1));   // -1
    println!("{}", get_or_default(&nums, 1, -1));    // 20
    println!("{}", get_or_default(&nums, 100, -1));  // -1
    println!("{}", get_or_default(&nums, 2, 0));     // 30
}
```

### A14 — Intentional panic reader
```rust
fn main() {
    // === Panic 1: Index out of bounds ===
    // let v = vec![1, 2, 3];
    // let x = v[99];
    // Panic message: "index out of bounds: the len is 3 but the index is 99"

    // Safe alternative:
    let v = vec![1, 2, 3];
    let x = v.get(99).unwrap_or(&0);
    println!("Safe index: {x}");  // 0

    // === Panic 2: unwrap on None ===
    // let opt: Option<i32> = None;
    // let val = opt.unwrap();
    // Panic message: "called `Option::unwrap()` on a `None` value"

    // Safe alternative:
    let opt: Option<i32> = None;
    let val = opt.unwrap_or(0);
    println!("Safe unwrap: {val}");  // 0

    // === Panic 3: Parse failure ===
    // let num: i32 = "hello".parse().unwrap();
    // Panic message: "called `Result::unwrap()` on an `Err` value: ParseIntError { kind: InvalidDigit }"

    // Safe alternative:
    let num: i32 = "hello".parse().unwrap_or(-1);
    println!("Safe parse: {num}");  // -1
}
```

### A15 — Validate function inputs with assert
```rust
fn create_percentage(value: f64) -> f64 {
    assert!(value >= 0.0, "Percentage cannot be negative, got {value}");
    assert!(value <= 100.0, "Percentage cannot exceed 100, got {value}");
    value
}

fn safe_create_percentage(value: f64) -> Option<f64> {
    if value >= 0.0 && value <= 100.0 {
        Some(value)
    } else {
        None
    }
}

fn main() {
    // Panic version — works for valid input
    println!("Panic version: {}", create_percentage(85.5));  // 85.5

    // create_percentage(-5.0);  // 💥 PANIC: "Percentage cannot be negative, got -5"
    // create_percentage(150.0); // 💥 PANIC: "Percentage cannot exceed 100, got 150"

    // Safe version
    println!("{:?}", safe_create_percentage(85.5));   // Some(85.5)
    println!("{:?}", safe_create_percentage(-5.0));    // None
    println!("{:?}", safe_create_percentage(150.0));   // None
    println!("{:?}", safe_create_percentage(0.0));     // Some(0.0)
    println!("{:?}", safe_create_percentage(100.0));   // Some(100.0)
}
```

---

## Section E

### A16 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `unwrap()` on `Some(value)` returns the value successfully — no panic |
| 2 | **True** | `expect("msg")` uses the provided message when panicking |
| 3 | **False** | `panic!` can be caught with `std::panic::catch_unwind` (unless `panic = "abort"`) |
| 4 | **True** | `assert_eq!` panics when the two values are not equal |
| 5 | **False** | `debug_assert!` is only checked in debug builds, not release |
| 6 | **True** | `.get(i)` returns `Option<&T>` — `None` for out of bounds, never panics |
| 7 | **False** | `v[i]` returns `T` directly and **panics** if `i` is out of bounds |
| 8 | **False** | `unwrap_or(default)` returns the default on `None` — it never panics |
| 9 | **True** | `panic = "abort"` skips stack unwinding and terminates immediately |
| 10 | **False** | `catch_unwind` is for FFI/thread pool boundaries, not general error handling — use `Result` |

### A17 — Explain the difference

1. **`unwrap()` vs `expect()`**: Both panic on `None`/`Err`. `expect` lets you provide a custom message explaining *why* you expected a value, making debugging easier. Prefer `expect` when the assumption is important.

2. **`unwrap()` vs `unwrap_or()`**: `unwrap` panics on `None`, while `unwrap_or` returns a fallback value. Use `unwrap_or` when you have a sensible default and the absence of a value is expected.

3. **`assert!()` vs `debug_assert!()`**: Both panic if the condition is false. `assert!` is checked in all builds (debug + release). `debug_assert!` is only checked in debug builds and compiled out in release. Use `debug_assert!` for expensive invariant checks that you trust in production.

4. **`panic!()` vs returning `Result::Err`**: `panic!` crashes the program — use for bugs/invariant violations. `Result::Err` returns an error value to the caller — use for expected failures that the caller can handle.

5. **`v[i]` vs `v.get(i)`**: `v[i]` panics if `i` is out of bounds and returns `T` directly. `v.get(i)` returns `Option<&T>` and never panics. Use `v.get(i)` when the index might be invalid.

### A18 — Big debug challenge
```rust
// Fix 1: Return Option instead of panicking
fn lookup_score(name: &str) -> Option<i32> {
    let scores = vec![("Alice", 95), ("Bob", 80), ("Cara", 88)];
    for (n, s) in scores {
        if n == name {
            return Some(s);
        }
    }
    None  // Fix 1: return None instead of panicking
}

fn main() {
    let name = "Dave";
    // Fix 2: handle the None case
    let score = lookup_score(name).unwrap_or(0);

    let numbers = vec![10, 20, 30];
    // Fix 3: use .get() for safe access, index 3 doesn't exist (only 0,1,2)
    let sum = numbers.get(0).unwrap_or(&0)
            + numbers.get(1).unwrap_or(&0)
            + numbers.get(2).unwrap_or(&0);  // fix: changed index 3 → 2

    let input = "not_a_number";
    // Fix 4: use unwrap_or instead of unwrap
    let parsed: i32 = input.parse().unwrap_or(0);

    // Fix 5: 2+2=4, not 5
    assert_eq!(2 + 2, 4);

    println!("Score: {score}, Sum: {sum}, Parsed: {parsed}");
    // Output: Score: 0, Sum: 60, Parsed: 0
}
```

---

## 🏆 Lesson 21 Complete!

✅ `panic!("msg")` — unrecoverable crash with a message  
✅ `unwrap()` — extract value or panic  
✅ `expect("msg")` — extract value or panic with a custom message  
✅ `assert!` / `assert_eq!` / `assert_ne!` — panic on failed conditions  
✅ `debug_assert!` — debug-only assertions  
✅ Safe alternatives: `unwrap_or`, `unwrap_or_else`, `v.get(i)`, `match`  
✅ When to panic vs when to return `Result`  
✅ Reading backtraces with `RUST_BACKTRACE=1`  

**Up next: Lesson 22 — Result\<T,E\> 🦀**
