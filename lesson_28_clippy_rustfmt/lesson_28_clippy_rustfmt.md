# 📘 Lesson 28 — Clippy & rustfmt (B5)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** B5 · Category: 🛠 Tooling  
> **Previous:** [Lesson 27 — Testing](../lesson_27_testing/lesson_27_testing.md)  
> **Next:** [Lesson 29 — File I/O & std::io](../lesson_29_file_io/lesson_29_file_io.md)  
> **Practice:** [Questions](./lesson_28_questions.md) · [Answers](./lesson_28_answers.md)  
> **Practice Task:** Fix all clippy warnings in an existing project

---

## Table of Contents

1. [Why Linting & Formatting?](#1-why-linting--formatting)
2. [rustfmt — The Formatter](#2-rustfmt--the-formatter)
3. [Configuring rustfmt](#3-configuring-rustfmt)
4. [Clippy — The Linter](#4-clippy--the-linter)
5. [Common Clippy Warnings](#5-common-clippy-warnings)
6. [Clippy Lint Levels](#6-clippy-lint-levels)
7. [Configuring Clippy](#7-configuring-clippy)
8. [Suppressing Warnings](#8-suppressing-warnings)
9. [Integrating into Your Workflow](#9-integrating-into-your-workflow)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Why Linting & Formatting?

| Tool | Purpose | What it does |
|---|---|---|
| `rustfmt` | **Formatting** | Makes code look consistent (indentation, spacing, line width) |
| `clippy` | **Linting** | Finds common mistakes, inefficiencies, and unidiomatic code |

Together they ensure your Rust code is:
- **Consistent** — everyone on the team writes code that looks the same
- **Idiomatic** — follows Rust community conventions
- **Correct** — catches subtle bugs and performance issues

---

## 2. rustfmt — The Formatter

```bash
# Format all files in the project
cargo fmt

# Check formatting without changing files (useful in CI)
cargo fmt -- --check

# Format a single file
rustfmt src/main.rs
```

### Before formatting:
```rust
fn main(){let x=5;let y=10;
if x>3{println!("big");
}else{println!("small");}
let v=vec![1,2,3,4,5];
for i in &v{println!("{}",i);}}
```

### After `cargo fmt`:
```rust
fn main() {
    let x = 5;
    let y = 10;
    if x > 3 {
        println!("big");
    } else {
        println!("small");
    }
    let v = vec![1, 2, 3, 4, 5];
    for i in &v {
        println!("{}", i);
    }
}
```

rustfmt handles:
- Indentation (4 spaces)
- Spacing around operators
- Brace placement
- Line width (default: 100 characters)
- Trailing commas
- Import ordering

---

## 3. Configuring rustfmt

Create a `rustfmt.toml` or `.rustfmt.toml` in your project root:

```toml
# rustfmt.toml
max_width = 100              # max line width (default: 100)
tab_spaces = 4               # spaces per indent (default: 4)
edition = "2021"             # Rust edition
use_small_heuristics = "Max" # pack more on each line

# Common options
imports_granularity = "Crate"   # merge use statements per crate
group_imports = "StdExternalCrate"  # group: std, external, crate
reorder_imports = true           # sort use statements
```

### Example with `imports_granularity = "Crate"`:

Before:
```rust
use std::collections::HashMap;
use std::collections::HashSet;
use std::io::Read;
use std::io::Write;
```

After:
```rust
use std::collections::{HashMap, HashSet};
use std::io::{Read, Write};
```

---

## 4. Clippy — The Linter

```bash
# Run clippy
cargo clippy

# Treat warnings as errors (useful in CI)
cargo clippy -- -D warnings

# Fix suggestions automatically where possible
cargo clippy --fix
```

### Example — clippy catches issues:

```rust
fn main() {
    // Warning: this if-else can be simplified to a bool expression
    let x = 5;
    let is_big = if x > 3 { true } else { false };
    //           ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ clippy says: simplify!

    // Warning: using `len()` to check emptiness
    let v = vec![1, 2, 3];
    if v.len() > 0 {
        println!("not empty");
    }

    // Warning: manual implementation of map
    let mut result = Vec::new();
    for item in &v {
        result.push(item * 2);
    }
}
```

### After fixing clippy warnings:

```rust
fn main() {
    let x = 5;
    let is_big = x > 3;  // simplified!

    let v = vec![1, 2, 3];
    if !v.is_empty() {  // use is_empty() instead of len() > 0
        println!("not empty");
    }

    let result: Vec<i32> = v.iter().map(|item| item * 2).collect();  // use iterator
}
```

---

## 5. Common Clippy Warnings

### Redundant patterns

```rust
// ❌ Clippy warning: redundant bool
let is_valid = if x > 0 { true } else { false };
// ✅ Fix
let is_valid = x > 0;

// ❌ Clippy warning: redundant clone
let s = String::from("hello");
let s2 = s.clone();  // warning if s is not used after this
// ✅ Fix: just move s
let s2 = s;
```

### Inefficient patterns

```rust
// ❌ Clippy: use `is_empty()` instead of `len() == 0`
if vec.len() == 0 { }
// ✅
if vec.is_empty() { }

// ❌ Clippy: manual loop can be iterator
let mut sum = 0;
for x in &numbers { sum += x; }
// ✅
let sum: i32 = numbers.iter().sum();

// ❌ Clippy: single-character string push
let mut s = String::new();
s.push_str("x");
// ✅
s.push('x');  // push a char instead
```

### Correctness issues

```rust
// ❌ Clippy: comparing floats with ==
let x: f64 = 0.1 + 0.2;
if x == 0.3 { }  // will be false due to float imprecision!
// ✅
if (x - 0.3).abs() < f64::EPSILON { }

// ❌ Clippy: using `unwrap()` in non-test code
let val = some_option.unwrap();
// ✅
let val = some_option.expect("option should be Some because...");
// or better:
let val = some_option.unwrap_or(default);
```

### Style issues

```rust
// ❌ Clippy: unnecessary `return` at end of function
fn add(a: i32, b: i32) -> i32 {
    return a + b;
}
// ✅
fn add(a: i32, b: i32) -> i32 {
    a + b  // implicit return
}

// ❌ Clippy: this `match` can be replaced with `if let`
match option {
    Some(val) => do_something(val),
    None => {},
}
// ✅
if let Some(val) = option {
    do_something(val);
}
```

---

## 6. Clippy Lint Levels

Clippy has hundreds of lints organised by category:

| Category | Description | Example |
|---|---|---|
| `clippy::correctness` | Almost certainly a bug | Float equality, infinite loop |
| `clippy::suspicious` | Likely a mistake | Shadowing with different type |
| `clippy::style` | Not idiomatic Rust | Unnecessary `return`, verbose bool |
| `clippy::complexity` | Unnecessarily complex | Match that could be `if let` |
| `clippy::perf` | Performance improvement | Unnecessary allocation |
| `clippy::pedantic` | Extra strict (opt-in) | Missing docs, wildcard imports |
| `clippy::nursery` | Experimental lints | Newer, potentially unstable |

```bash
# Run with pedantic lints (stricter)
cargo clippy -- -W clippy::pedantic

# Run with all lints
cargo clippy -- -W clippy::all
```

---

## 7. Configuring Clippy

### In code — per-item or per-file:

```rust
// Allow a specific lint for one item
#[allow(clippy::needless_return)]
fn my_function() -> i32 {
    return 42;
}

// Allow a lint for the entire file
#![allow(clippy::too_many_arguments)]

// Warn on a specific lint
#![warn(clippy::pedantic)]

// Deny a lint (make it an error)
#![deny(clippy::unwrap_used)]
```

### In `Cargo.toml`:

```toml
[lints.clippy]
pedantic = "warn"
unwrap_used = "deny"
needless_return = "allow"
```

### In `clippy.toml`:

```toml
# clippy.toml — project-wide configuration
too-many-arguments-threshold = 10   # default is 7
cognitive-complexity-threshold = 30 # default is 25
```

---

## 8. Suppressing Warnings

Sometimes clippy is wrong or you have a good reason to deviate:

```rust
// Suppress for a single expression/item
#[allow(clippy::redundant_clone)]
let s2 = s.clone();  // I know s is used later in another thread

// Suppress for a function
#[allow(clippy::cast_possible_truncation)]
fn as_u8(x: u32) -> u8 {
    x as u8  // I've verified x is always < 256
}

// Suppress for the entire module/file
#![allow(clippy::module_name_repetitions)]
```

### Compiler warnings (not clippy):

```rust
#[allow(dead_code)]       // suppress "unused function" warning
fn unused_helper() { }

#[allow(unused_variables)] // suppress "unused variable" warning
fn demo(x: i32) { }

// Or use underscore prefix (idiomatic)
fn demo2(_x: i32) { }     // no warning
let _unused = 42;          // no warning
```

---

## 9. Integrating into Your Workflow

### Pre-commit check:

```bash
# Run before every commit
cargo fmt -- --check && cargo clippy -- -D warnings && cargo test
```

### CI/CD pipeline:

```yaml
# GitHub Actions example
- name: Format check
  run: cargo fmt -- --check

- name: Clippy
  run: cargo clippy -- -D warnings

- name: Test
  run: cargo test
```

### Editor integration:
- **VS Code**: Install `rust-analyzer` — runs clippy and fmt automatically
- **IntelliJ**: Install the Rust plugin — built-in support
- Format on save: most editors support this with rust-analyzer

### Recommended workflow:

1. **Write code** normally
2. **`cargo fmt`** to fix formatting
3. **`cargo clippy`** to find issues
4. **Fix clippy warnings** or suppress with `#[allow(...)]` + comment
5. **`cargo test`** to verify correctness
6. **Commit**

---

## 10. Summary Cheat Sheet

```
RUSTFMT
────────────────────────────────────────────────────────────
cargo fmt                    format all files
cargo fmt -- --check         check without modifying (for CI)
rustfmt.toml                 configuration file

CLIPPY
────────────────────────────────────────────────────────────
cargo clippy                 run the linter
cargo clippy -- -D warnings  treat warnings as errors
cargo clippy --fix           auto-fix where possible
cargo clippy -- -W clippy::pedantic  enable stricter lints

SUPPRESSING LINTS
────────────────────────────────────────────────────────────
#[allow(clippy::lint_name)]  suppress for one item
#![allow(clippy::lint_name)] suppress for file/module
#[allow(dead_code)]          suppress compiler warning
_variable                    suppress unused variable warning

CONFIGURING
────────────────────────────────────────────────────────────
rustfmt.toml        formatting rules
clippy.toml         clippy thresholds
Cargo.toml [lints]  lint levels per project

COMMON FIXES
────────────────────────────────────────────────────────────
if x { true } else { false }  →  x
v.len() == 0                  →  v.is_empty()
v.len() > 0                   →  !v.is_empty()
return expr;                  →  expr (last expression)
for x in &v { sum += x }     →  v.iter().sum()
s.push_str("x")              →  s.push('x')
```

---

## What's Next?

**Lesson 29 — File I/O & std::io** — Read and write files, use `BufReader`/`BufWriter`, handle stdin/stdout, and build a CSV parser.

## Further Reading
- [Clippy Lint List](https://rust-lang.github.io/rust-clippy/master/)
- [rustfmt Configuration](https://rust-lang.github.io/rustfmt/)
- [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/)

---

*Let the tools work for you — format and lint on every save! 🦀*
