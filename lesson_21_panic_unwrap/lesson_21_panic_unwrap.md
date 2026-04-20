# 📘 Lesson 21 — panic! & unwrap (E1)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** E1 · Category: ⚠ Error Handling  
> **Previous:** [Lesson 20 — HashMap\<K,V\>](../lesson_20_hashmap/lesson_20_hashmap.md)  
> **Next:** [Lesson 22 — Result\<T,E\>](../lesson_22_result/lesson_22_result.md)  
> **Practice:** [Questions](./lesson_21_questions.md) · [Answers](./lesson_21_answers.md)  
> **Practice Task:** Intentionally trigger a panic; read the backtrace

---

## Table of Contents

1. [Two Kinds of Errors in Rust](#1-two-kinds-of-errors-in-rust)
2. [What Is a Panic?](#2-what-is-a-panic)
3. [The `panic!` Macro](#3-the-panic-macro)
4. [Reading a Backtrace](#4-reading-a-backtrace)
5. [`unwrap()` — The Quick-and-Dirty Extractor](#5-unwrap--the-quick-and-dirty-extractor)
6. [`expect()` — unwrap with a Message](#6-expect--unwrap-with-a-message)
7. [`assert!`, `assert_eq!`, `assert_ne!`](#7-assert-assert_eq-assert_ne)
8. [When Should You Panic?](#8-when-should-you-panic)
9. [Panic vs Return an Error](#9-panic-vs-return-an-error)
10. [Configuring Panic Behaviour](#10-configuring-panic-behaviour)
11. [Common Panic Sources](#11-common-panic-sources)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. Two Kinds of Errors in Rust

Rust splits errors into two categories:

| | Unrecoverable | Recoverable |
|---|---|---|
| Mechanism | `panic!` | `Result<T, E>` |
| What happens | Program crashes (unwinds the stack) | Caller decides what to do |
| When to use | Bugs, violated invariants, impossible states | Expected failures (file not found, bad input) |
| Example | Index out of bounds | File open failing |

This lesson covers the **unrecoverable** side. The next two lessons cover `Result<T, E>` and `?`.

---

## 2. What Is a Panic?

A **panic** is Rust's way of saying: *"Something went so wrong that the program cannot continue safely."*

When a panic occurs:
1. Rust prints an error message
2. The stack **unwinds** — Rust walks back up the call stack, running destructors (`drop`) for every value
3. The program exits with a non-zero exit code

```
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', src/main.rs:3:5
```

---

## 3. The `panic!` Macro

You can trigger a panic intentionally with `panic!`:

```rust
fn main() {
    panic!("Something terrible happened!");
}
```

**Output:**
```
thread 'main' panicked at 'Something terrible happened!', src/main.rs:2:5
```

### With format arguments (just like `println!`):

```rust
fn main() {
    let user_id = 42;
    let max_id = 30;
    panic!("User ID {user_id} exceeds maximum allowed ({max_id})");
}
```

**Output:**
```
thread 'main' panicked at 'User ID 42 exceeds maximum allowed (30)', src/main.rs:4:5
```

### Panic inside a function:

```rust
fn divide(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Division by zero! a={a}, b={b}");
    }
    a / b
}

fn main() {
    println!("{}", divide(10, 2));  // 5
    println!("{}", divide(10, 0));  // 💥 PANIC
}
```

---

## 4. Reading a Backtrace

A **backtrace** shows you the chain of function calls that led to the panic. Enable it with the `RUST_BACKTRACE` environment variable:

```bash
# Show a short backtrace
RUST_BACKTRACE=1 cargo run

# Show a full backtrace
RUST_BACKTRACE=full cargo run
```

On Windows PowerShell:
```powershell
$env:RUST_BACKTRACE=1; cargo run
```

**Example backtrace (simplified):**
```
thread 'main' panicked at 'Division by zero!', src/main.rs:3:9
stack backtrace:
   0: std::panicking::begin_panic
   1: playground::divide        ← YOUR function
   2: playground::main           ← called from main
   3: std::rt::lang_start
```

**How to read it:**
- Start from the **bottom** and read upward
- Look for lines mentioning **your** file names
- The topmost line with your code is where the panic occurred

---

## 5. `unwrap()` — The Quick-and-Dirty Extractor

`Option<T>` and `Result<T, E>` both have an `unwrap()` method that either:
- Returns the inner value if it's `Some(v)` or `Ok(v)`
- **Panics** if it's `None` or `Err(e)`

### On `Option<T>`:

```rust
fn main() {
    let some_value: Option<i32> = Some(42);
    let none_value: Option<i32> = None;

    println!("{}", some_value.unwrap()); // 42
    println!("{}", none_value.unwrap()); // 💥 PANIC: called `Option::unwrap()` on a `None` value
}
```

### On `Result<T, E>`:

```rust
fn main() {
    let ok_value: Result<i32, String> = Ok(100);
    let err_value: Result<i32, String> = Err(String::from("something failed"));

    println!("{}", ok_value.unwrap());  // 100
    println!("{}", err_value.unwrap()); // 💥 PANIC: called `Result::unwrap()` on an `Err` value: "something failed"
}
```

### Real-world example — parsing:

```rust
fn main() {
    let number: i32 = "42".parse().unwrap();     // ✅ Works
    println!("{number}");

    let bad: i32 = "hello".parse().unwrap();      // 💥 PANIC
    println!("{bad}");
}
```

### ⚠️ When to use `unwrap()`

- ✅ Quick prototyping / experimenting
- ✅ Examples and tests where failure is a bug
- ✅ When you **know** the value is `Some` / `Ok` and can prove it logically
- ❌ **Never** in production code where failure is possible
- ❌ **Never** on user input

---

## 6. `expect()` — unwrap with a Message

`expect()` works exactly like `unwrap()`, but you provide a custom panic message:

```rust
fn main() {
    let config_port: u16 = "8080".parse()
        .expect("PORT must be a valid u16");  // ✅ 8080

    let bad_port: u16 = "not_a_port".parse()
        .expect("PORT must be a valid u16");  // 💥 PANIC with YOUR message
}
```

**Panic output:**
```
thread 'main' panicked at 'PORT must be a valid u16: ParseIntError { kind: InvalidDigit }'
```

### `expect` vs `unwrap`:

| | `unwrap()` | `expect("msg")` |
|---|---|---|
| On success | Returns inner value | Returns inner value |
| On failure | Panics with generic message | Panics with **your** message |
| Readability | Poor — "what was I unwrapping?" | Good — message explains intent |
| Use when | Quick experiments | You need context on failure |

**Best practice:** Prefer `expect()` over `unwrap()` — the message serves as documentation for why you believe the value should always be present:

```rust
fn main() {
    // Good — explains the assumption
    let home = std::env::var("HOME")
        .expect("HOME environment variable must be set");

    // Bad — no context
    let home = std::env::var("HOME").unwrap();
}
```

---

## 7. `assert!`, `assert_eq!`, `assert_ne!`

These macros panic if a condition is not met. They're primarily used in **tests** but can be used anywhere.

### `assert!` — asserts a condition is true

```rust
fn main() {
    let age = 25;
    assert!(age >= 18, "User must be an adult, got age={age}");
    println!("Access granted"); // ✅ Runs
}
```

If `age` were `15`:
```
thread 'main' panicked at 'User must be an adult, got age=15'
```

### `assert_eq!` — asserts two values are equal

```rust
fn main() {
    let result = 2 + 2;
    assert_eq!(result, 4);                      // ✅
    assert_eq!(result, 5, "Math is broken!");   // 💥 PANIC
}
```

**Panic output:**
```
thread 'main' panicked at 'assertion `left == right` failed: Math is broken!
  left: 4
 right: 5'
```

### `assert_ne!` — asserts two values are NOT equal

```rust
fn main() {
    let password = "admin";
    assert_ne!(password, "password123", "Password too common!");
    println!("Password accepted"); // ✅
}
```

### Debug-only asserts — `debug_assert!`

```rust
fn main() {
    let index = 5;
    // Only checked in debug builds (cargo build), NOT in release (cargo build --release)
    debug_assert!(index < 100, "Index out of range");
}
```

| Macro | Checked in debug? | Checked in release? |
|---|---|---|
| `assert!` | ✅ Yes | ✅ Yes |
| `debug_assert!` | ✅ Yes | ❌ No (compiled out) |

---

## 8. When Should You Panic?

**Panic when:**

| Situation | Example |
|---|---|
| A bug is detected | Invariant violated that "can't happen" |
| Unrecoverable state | Corrupted data structure |
| Tests | Expected values not matching |
| Prototyping | Quick and dirty experiments |
| Setup/config failure | Missing required environment variable |

**Don't panic when:**

| Situation | Use instead |
|---|---|
| File not found | `Result<T, E>` |
| User gave bad input | `Result<T, E>` |
| Network request failed | `Result<T, E>` |
| Parse error | `Result<T, E>` |

### Rule of thumb:

> If the error is **expected** and the caller can do something about it → `Result`.  
> If the error is a **bug** and continuing would be worse → `panic!`.

---

## 9. Panic vs Return an Error

```rust
// ❌ BAD — panics on expected input
fn parse_age_bad(input: &str) -> u32 {
    input.parse().unwrap() // panics on "abc"
}

// ✅ GOOD — returns Result so caller can handle it
fn parse_age_good(input: &str) -> Result<u32, std::num::ParseIntError> {
    input.parse()
}

fn main() {
    // The caller decides what to do with errors
    match parse_age_good("25") {
        Ok(age) => println!("Age: {age}"),
        Err(e) => println!("Invalid age: {e}"),
    }

    match parse_age_good("abc") {
        Ok(age) => println!("Age: {age}"),
        Err(e) => println!("Invalid age: {e}"), // handled gracefully
    }
}
```

**Output:**
```
Age: 25
Invalid age: invalid digit found in string
```

---

## 10. Configuring Panic Behaviour

### Unwind vs Abort

By default, Rust **unwinds** the stack on panic (running destructors). You can change this to **abort** (immediately terminate) in `Cargo.toml`:

```toml
[profile.release]
panic = "abort"    # smaller binary, faster termination, but no cleanup
```

| | Unwind (default) | Abort |
|---|---|---|
| Destructors run? | ✅ Yes | ❌ No |
| Binary size | Larger (unwind tables) | Smaller |
| Can catch panics? | ✅ Yes (`catch_unwind`) | ❌ No |
| Use case | Applications | Embedded / WASM |

### Catching panics (advanced)

You can catch a panic with `std::panic::catch_unwind`:

```rust
use std::panic;

fn main() {
    let result = panic::catch_unwind(|| {
        panic!("oh no!");
    });

    match result {
        Ok(_) => println!("No panic occurred"),
        Err(_) => println!("Caught a panic!"),  // ← this runs
    }

    println!("Program continues after caught panic");
}
```

> ⚠️ `catch_unwind` is **not** for general error handling. It's for FFI boundaries and thread pools. Use `Result` for normal error flow.

---

## 11. Common Panic Sources

These standard library operations **panic** under certain conditions:

```rust
fn main() {
    // 1. Index out of bounds
    let v = vec![1, 2, 3];
    // let x = v[99]; // 💥 PANIC: index out of bounds

    // Safe alternative:
    let x = v.get(99); // Returns None instead of panicking
    println!("{:?}", x); // None

    // 2. Integer overflow in debug mode
    // let x: u8 = 255 + 1; // 💥 PANIC in debug, wraps in release

    // 3. Slice out of range
    let s = "hello";
    // let sub = &s[0..99]; // 💥 PANIC: byte index 99 is out of bounds

    // 4. unwrap() on None / Err
    let opt: Option<i32> = None;
    // opt.unwrap(); // 💥 PANIC

    // 5. Division by zero (integers only)
    // let x = 10 / 0; // 💥 PANIC

    // 6. String slicing at non-char boundary
    let emoji = "🦀hello";
    // let bad = &emoji[0..1]; // 💥 PANIC: byte index 1 is not a char boundary

    println!("All safe alternatives used!");
}
```

### Safe alternatives cheat sheet:

| Panicking operation | Safe alternative |
|---|---|
| `v[i]` | `v.get(i)` → `Option<&T>` |
| `s[range]` | `s.get(range)` → `Option<&str>` |
| `.unwrap()` | `.unwrap_or(default)` or `match` |
| `a / b` | Check `b != 0` first |
| `.parse().unwrap()` | `.parse()` → handle `Result` |

---

## 12. Summary Cheat Sheet

```
TRIGGERING PANICS
────────────────────────────────────────────────────────────
panic!("msg")              explicitly crash with a message
panic!("x={x}, y={y}")    with formatted arguments

EXTRACTING VALUES (or panicking)
────────────────────────────────────────────────────────────
opt.unwrap()               Some(v) → v   |  None → PANIC
opt.expect("msg")          Some(v) → v   |  None → PANIC with msg
res.unwrap()               Ok(v)  → v    |  Err(e) → PANIC
res.expect("msg")          Ok(v)  → v    |  Err(e) → PANIC with msg

SAFE ALTERNATIVES
────────────────────────────────────────────────────────────
opt.unwrap_or(default)     Some(v) → v   |  None → default
opt.unwrap_or_else(|| f()) Some(v) → v   |  None → f()
opt.unwrap_or_default()    Some(v) → v   |  None → Default::default()

ASSERTIONS
────────────────────────────────────────────────────────────
assert!(cond)              panics if cond is false
assert!(cond, "msg")       panics with message
assert_eq!(a, b)           panics if a != b
assert_ne!(a, b)           panics if a == b
debug_assert!(cond)        debug builds only

BACKTRACE
────────────────────────────────────────────────────────────
RUST_BACKTRACE=1 cargo run     short backtrace
RUST_BACKTRACE=full cargo run  full backtrace

WHEN TO PANIC
────────────────────────────────────────────────────────────
✅ Bugs / violated invariants / impossible states
✅ Tests and prototyping
❌ Expected failures → use Result<T, E> instead
```

---

## What's Next?

**Lesson 22 — Result\<T,E\>** — The recoverable error type. Learn `Ok`, `Err`, `match`, `map`, `and_then`, and how to write functions that can fail gracefully.

## Further Reading
- [The Rust Book — Ch 9.1: Unrecoverable Errors with panic!](https://doc.rust-lang.org/book/ch09-01-unrecoverable-errors-with-panic.html)
- [std::panic module](https://doc.rust-lang.org/std/panic/index.html)

---

*Don't panic! …unless you really should. 🦀*
