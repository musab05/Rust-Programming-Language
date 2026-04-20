# 📘 Lesson 27 — Testing (B2)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** B2 · Category: 🛠 Tooling  
> **Previous:** [Lesson 26 — Crates & Cargo](../lesson_26_crates_cargo/lesson_26_crates_cargo.md)  
> **Next:** [Lesson 28 — Clippy & rustfmt](../lesson_28_clippy_rustfmt/lesson_28_clippy_rustfmt.md)  
> **Practice:** [Questions](./lesson_27_questions.md) · [Answers](./lesson_27_answers.md)  
> **Practice Task:** 100% unit-test coverage on your Vec utilities

---

## Table of Contents

1. [Why Test?](#1-why-test)
2. [Your First Test](#2-your-first-test)
3. [`assert!`, `assert_eq!`, `assert_ne!`](#3-assert-assert_eq-assert_ne)
4. [Custom Failure Messages](#4-custom-failure-messages)
5. [Testing for Panics: `#[should_panic]`](#5-testing-for-panics-should_panic)
6. [Tests That Return `Result`](#6-tests-that-return-result)
7. [Running Tests with `cargo test`](#7-running-tests-with-cargo-test)
8. [Test Organisation](#8-test-organisation)
9. [Ignoring Tests](#9-ignoring-tests)
10. [Testing Private Functions](#10-testing-private-functions)
11. [Test Helpers & Setup](#11-test-helpers--setup)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. Why Test?

Tests are code that verifies your other code works correctly. They catch bugs before they reach users.

**Rust's testing philosophy:**
- Tests are first-class citizens — built into the language and toolchain
- `cargo test` runs all tests automatically
- Tests fail if they **panic** (assertion failure, `unwrap` on `None`, etc.)
- Tests pass if they **return normally**

---

## 2. Your First Test

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]        // only compile this module during testing
mod tests {
    use super::*;   // import everything from parent module

    #[test]         // marks this function as a test
    fn test_add() {
        let result = add(2, 3);
        assert_eq!(result, 5);
    }
}
```

Run it:
```bash
$ cargo test
running 1 test
test tests::test_add ... ok

test result: ok. 1 passed; 0 failed; 0 ignored
```

### Anatomy:
- `#[cfg(test)]` — module is only compiled during `cargo test`
- `#[test]` — marks a function as a test case
- `use super::*` — imports everything from the parent scope
- `assert_eq!` — panics (= test fails) if two values aren't equal

---

## 3. `assert!`, `assert_eq!`, `assert_ne!`

### `assert!` — asserts a boolean is true

```rust
#[test]
fn test_is_positive() {
    let x = 42;
    assert!(x > 0);
    assert!(x.is_positive());
}
```

### `assert_eq!` — asserts two values are equal

```rust
#[test]
fn test_multiply() {
    assert_eq!(3 * 4, 12);
    assert_eq!("hello".len(), 5);
}
```

### `assert_ne!` — asserts two values are NOT equal

```rust
#[test]
fn test_not_equal() {
    assert_ne!(1 + 1, 3);
    assert_ne!("rust", "python");
}
```

### Important: `assert_eq!` requires `PartialEq + Debug`

The types must implement `PartialEq` (for comparison) and `Debug` (for printing on failure). Most standard types do. For custom types, derive them:

```rust
#[derive(Debug, PartialEq)]
struct Point { x: i32, y: i32 }

#[test]
fn test_point() {
    let p = Point { x: 1, y: 2 };
    assert_eq!(p, Point { x: 1, y: 2 });
}
```

---

## 4. Custom Failure Messages

Add context to make failures easier to debug:

```rust
#[test]
fn test_greeting() {
    let name = "Alice";
    let greeting = format!("Hello, {name}!");
    assert!(
        greeting.contains("Alice"),
        "Greeting should contain the name, got: '{greeting}'"
    );
}

#[test]
fn test_range() {
    let value = 42;
    assert!(
        value >= 1 && value <= 100,
        "Expected value in range 1..=100, got {value}"
    );
}
```

**On failure:**
```
thread 'test_range' panicked at 'Expected value in range 1..=100, got 150'
```

---

## 5. Testing for Panics: `#[should_panic]`

Sometimes you want to verify that code **does** panic:

```rust
fn divide(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("Division by zero!");
    }
    a / b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn test_divide_by_zero() {
        divide(10, 0);  // should panic — test passes!
    }

    #[test]
    #[should_panic(expected = "Division by zero")]
    fn test_divide_by_zero_message() {
        divide(10, 0);  // passes only if panic message contains this text
    }

    #[test]
    fn test_divide_normal() {
        assert_eq!(divide(10, 2), 5);  // no panic — test passes normally
    }
}
```

### `expected` parameter
- Without `expected`: test passes if *any* panic occurs
- With `expected`: test only passes if the panic message **contains** the expected string

---

## 6. Tests That Return `Result`

Instead of using `assert!`, tests can return `Result<(), E>`:

```rust
#[test]
fn test_parse() -> Result<(), String> {
    let number: i32 = "42".parse().map_err(|e| format!("{e}"))?;
    if number != 42 {
        return Err(format!("Expected 42, got {number}"));
    }
    Ok(())  // test passes
}
```

This lets you use `?` in tests. The test:
- **Passes** if it returns `Ok(())`
- **Fails** if it returns `Err(msg)`

> ⚠️ You can't use `#[should_panic]` with Result-returning tests. Use `assert!` for panic tests.

---

## 7. Running Tests with `cargo test`

```bash
# Run all tests
cargo test

# Run tests with a name containing "add"
cargo test add

# Run a specific test by full name
cargo test tests::test_add

# Run tests in a specific module
cargo test tests::

# Show println! output (normally captured)
cargo test -- --show-output

# Run tests on a single thread (useful for shared state)
cargo test -- --test-threads=1

# List all tests without running them
cargo test -- --list

# Run ignored tests
cargo test -- --ignored

# Run ALL tests (including ignored)
cargo test -- --include-ignored
```

### Test output:
```
running 4 tests
test tests::test_add ... ok
test tests::test_subtract ... ok
test tests::test_multiply ... ok
test tests::test_divide ... FAILED

failures:

---- tests::test_divide stdout ----
thread 'tests::test_divide' panicked at 'assertion `left == right` failed
  left: 3
 right: 4'

failures:
    tests::test_divide

test result: FAILED. 3 passed; 1 failed; 0 ignored
```

---

## 8. Test Organisation

### Unit tests — test individual functions

Convention: put them in a `tests` module inside each file:

```rust
// src/lib.rs or src/main.rs

pub fn factorial(n: u64) -> u64 {
    (1..=n).product()
}

pub fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => {
            let (mut a, mut b) = (0u64, 1u64);
            for _ in 2..=n {
                let temp = b;
                b = a + b;
                a = temp;
            }
            b
        }
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_factorial_zero() {
        assert_eq!(factorial(0), 1);
    }

    #[test]
    fn test_factorial_five() {
        assert_eq!(factorial(5), 120);
    }

    #[test]
    fn test_fibonacci_base() {
        assert_eq!(fibonacci(0), 0);
        assert_eq!(fibonacci(1), 1);
    }

    #[test]
    fn test_fibonacci_sequence() {
        assert_eq!(fibonacci(10), 55);
        assert_eq!(fibonacci(20), 6765);
    }
}
```

### Integration tests — test the public API

Place in a `tests/` directory at the project root:

```
my_project/
├── Cargo.toml
├── src/
│   └── lib.rs
└── tests/
    └── integration_test.rs
```

```rust
// tests/integration_test.rs
use my_project::{factorial, fibonacci};

#[test]
fn test_factorial_large() {
    assert_eq!(factorial(10), 3628800);
}

#[test]
fn test_fibonacci_large() {
    assert_eq!(fibonacci(30), 832040);
}
```

Integration tests:
- Each file in `tests/` is compiled as a separate crate
- They can only use your **public** API
- No `#[cfg(test)]` needed — Cargo knows they're tests
- Run with `cargo test --test integration_test`

---

## 9. Ignoring Tests

Mark slow or environment-specific tests with `#[ignore]`:

```rust
#[test]
fn quick_test() {
    assert_eq!(2 + 2, 4);
}

#[test]
#[ignore]  // skipped by default
fn expensive_test() {
    // This takes 30 seconds...
    std::thread::sleep(std::time::Duration::from_secs(30));
    assert!(true);
}
```

```bash
cargo test                  # runs quick_test, skips expensive_test
cargo test -- --ignored     # runs ONLY ignored tests
cargo test -- --include-ignored  # runs ALL tests
```

---

## 10. Testing Private Functions

Unlike most languages, Rust allows testing private functions directly:

```rust
fn internal_helper(x: i32) -> i32 {
    x * 2 + 1
}

pub fn public_api(x: i32) -> i32 {
    internal_helper(x) + 10
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_private_helper() {
        // ✅ This works! Tests can access private functions
        // because the test module is a child of the containing module
        assert_eq!(internal_helper(5), 11);
    }

    #[test]
    fn test_public_api() {
        assert_eq!(public_api(5), 21);
    }
}
```

This is valid because child modules can access their parent's private items.

---

## 11. Test Helpers & Setup

### Shared setup function:

```rust
#[cfg(test)]
mod tests {
    use super::*;

    fn setup() -> Vec<i32> {
        vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
    }

    #[test]
    fn test_sum() {
        let data = setup();
        let sum: i32 = data.iter().sum();
        assert_eq!(sum, 55);
    }

    #[test]
    fn test_length() {
        let data = setup();
        assert_eq!(data.len(), 10);
    }

    #[test]
    fn test_contains() {
        let data = setup();
        assert!(data.contains(&5));
        assert!(!data.contains(&99));
    }
}
```

### Approximate equality for floats:

```rust
fn approx_eq(a: f64, b: f64, epsilon: f64) -> bool {
    (a - b).abs() < epsilon
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_float_math() {
        let result = 0.1 + 0.2;
        // assert_eq!(result, 0.3); // ❌ fails! floating point imprecision
        assert!(approx_eq(result, 0.3, 1e-10)); // ✅
    }
}
```

---

## 12. Summary Cheat Sheet

```
TEST ATTRIBUTES
────────────────────────────────────────────────────────────
#[test]                    marks a function as a test
#[should_panic]            test passes if it panics
#[should_panic(expected)]  must panic with specific message
#[ignore]                  skip this test by default
#[cfg(test)]               only compile during testing

ASSERTIONS
────────────────────────────────────────────────────────────
assert!(expr)              true or panic
assert!(expr, "msg")       true or panic with message
assert_eq!(a, b)           a == b or panic (shows both values)
assert_ne!(a, b)           a != b or panic

RUNNING TESTS
────────────────────────────────────────────────────────────
cargo test                 all tests
cargo test name            tests matching "name"
cargo test -- --show-output  see println! output
cargo test -- --ignored    run ignored tests
cargo test -- --test-threads=1  single-threaded

TEST ORGANISATION
────────────────────────────────────────────────────────────
Unit tests     #[cfg(test)] mod tests { }   inside src files
Integration    tests/test_name.rs           separate crate
               uses only pub API

RESULT-RETURNING TESTS
────────────────────────────────────────────────────────────
#[test]
fn test() -> Result<(), String> {
    let x = parse()?;   // use ? in tests
    Ok(())
}
```

---

## What's Next?

**Lesson 28 — Clippy & rustfmt** — The Rust linter and formatter. Let the tools make your code idiomatic and consistent.

## Further Reading
- [The Rust Book — Ch 11: Writing Automated Tests](https://doc.rust-lang.org/book/ch11-00-testing.html)
- [Rust by Example — Testing](https://doc.rust-lang.org/rust-by-example/testing.html)

---

*Tests are your safety net — write them, trust them! 🦀*
