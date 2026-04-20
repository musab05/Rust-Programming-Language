# ✅ Lesson 28 — Answers: Clippy & rustfmt (B5)

---

## Section A

### A1 — Redundant bool
```rust
// Clippy: `if x { true } else { false }` can be simplified
fn is_adult(age: u32) -> bool {
    age >= 18  // direct bool expression
}
```

### A2 — Use `is_empty()`
```rust
fn main() {
    let names = vec!["Alice", "Bob", "Cara"];
    if !names.is_empty() {  // instead of len() > 0
        println!("Have names");
    }
}
```

### A3 — Needless return
```rust
fn add(a: i32, b: i32) -> i32 {
    a + b  // implicit return, no `return` keyword needed
}
```

### A4 — Single char push
```rust
fn main() {
    let mut s = String::new();
    s.push('!');  // push(char) instead of push_str("!")
}
```

### A5 — Clone on Copy type
```rust
fn main() {
    let x = 42;
    let y = x;  // i32 is Copy — clone is unnecessary
    println!("{y}");
}
```

### A6 — Manual loop → iterator
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let sum: i32 = v.iter().sum();  // iterator instead of manual loop
    println!("{sum}");
}
```

---

## Section B

### A7 — Fixed program
```rust
fn get_status(code: i32) -> String {
    // Fix: remove unnecessary returns and dead code
    match code {
        200 => String::from("OK"),
        404 => String::from("Not Found"),
        500 => String::from("Server Error"),
        _ => String::from("Unknown"),
    }
}

fn is_valid(s: &str) -> bool {
    // Fix: is_empty() instead of len() == 0
    // Fix: direct bool instead of if/else true/false
    !s.is_empty() && s.contains('@')  // '@' as char, not "&"
}

fn process(items: &[i32]) -> Vec<i32> {
    // Fix: &[i32] instead of &Vec<i32>
    // Fix: iterator instead of index loop
    items.iter()
        .filter(|&&x| x > 0)
        .map(|&x| x * 2)
        .collect()
}

fn main() {
    let status = get_status(200);
    println!("{status}");

    let valid = is_valid("user@example.com");
    println!("Valid: {valid}");

    let nums = vec![1, -2, 3, -4, 5];
    let doubled = process(&nums);
    println!("{doubled:?}");
}
```

**Changes made:**
1. `get_status`: replaced if/else chain with `match`, removed unnecessary `return` and dead `result` variable
2. `is_valid`: `len() == 0` → `is_empty()`, `if ... { true } else { false }` → direct bool, `"@"` → `'@'`
3. `process`: `&Vec<i32>` → `&[i32]` (more general), index loop → iterator chain
4. `{:?}` → `{doubled:?}` (inline format variable)

---

## Section C

### A8 — Formatted code
```rust
fn main() {
    let x = 5;
    let y = 10;
    if x > 3 {
        println!("{}", x + y);
    } else {
        println!("small");
    }
    let v = vec![1, 2, 3];
    for i in &v {
        println!("{}", i);
    }
}
```

### A9 — rustfmt.toml
```toml
max_width = 80
tab_spaces = 2
imports_granularity = "Crate"
```

---

## Section D

### A10 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `cargo fmt` writes formatted code directly to the source files |
| 2 | **False** | `cargo clippy --fix` can fix many but not all warnings |
| 3 | **False** | `#[allow(...)]` only applies to the item it's attached to (or module with `#!`) |
| 4 | **False** | Some are style suggestions or micro-optimizations, not bugs |
| 5 | **True** | `--check` returns non-zero exit code if files would change |
| 6 | **False** | `pedantic` lints are opt-in — you must enable them explicitly |
| 7 | **True** | `[lints.clippy]` in Cargo.toml is a supported way to configure lints |
| 8 | **True** | `_x` tells the compiler the variable is intentionally unused |

### A11 — Fix vs Suppress

1. **`needless_return`** — **Fix it.** Remove the explicit `return`. Even complex match expressions work as implicit returns. This is idiomatic Rust.

2. **`too_many_arguments`** — **Fix it.** 8 parameters suggests the function should be refactored — use a config struct to bundle related parameters together.

3. **`cast_possible_truncation`** — **Suppress it** with `#[allow(clippy::cast_possible_truncation)]` and a comment explaining why the range is verified. The compiler can't know your runtime invariant.

4. **`unwrap_used`** in tests — **Suppress it.** Tests are supposed to panic on unexpected values. `unwrap()` is perfectly appropriate in test code. Use `#![allow(clippy::unwrap_used)]` at the test module level.

---

## 🏆 Lesson 28 Complete!

✅ `cargo fmt` — automatic code formatting  
✅ `rustfmt.toml` — formatting configuration  
✅ `cargo clippy` — catches common mistakes and non-idiomatic code  
✅ Common clippy warnings and how to fix them  
✅ `#[allow(clippy::...)]` — suppressing lints when appropriate  
✅ Lint levels: correctness, style, complexity, perf, pedantic  
✅ Integration into workflow and CI  

**Up next: Lesson 29 — File I/O & std::io 🦀**
