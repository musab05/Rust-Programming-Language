# 📘 Lesson 69 — Testing: Unit & Integration (TE1)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** TE1 · Category: 🧪 Testing  
> **Previous:** [Lesson 68 — State & Strategy Patterns](../lesson_68_state_strategy/lesson_68_state_strategy.md)  
> **Next:** [Lesson 70 — Testing: Mocking & Property-Based](../lesson_70_advanced_testing/lesson_70_advanced_testing.md)  
> **Practice:** [Questions](./lesson_69_questions.md) · [Answers](./lesson_69_answers.md)  
> **Practice Task:** Write unit tests for a calculator module and integration tests for a library crate

---

## Table of Contents

1. [Testing in Rust](#1-testing-in-rust)
2. [Unit Tests](#2-unit-tests)
3. [Assert Macros](#3-assert-macros)
4. [Testing for Panics](#4-testing-for-panics)
5. [Testing Results](#5-testing-results)
6. [Test Organization](#6-test-organization)
7. [Integration Tests](#7-integration-tests)
8. [Test Configuration](#8-test-configuration)
9. [Running Tests](#9-running-tests)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Testing in Rust

Rust has a built-in test framework — no external library needed:

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2, 3), 5);
    }
}
```

```bash
$ cargo test
running 1 test
test tests::test_add ... ok
test result: ok. 1 passed; 0 failed
```

---

## 2. Unit Tests

Unit tests live in the same file as the code, inside a `tests` module:

```rust
pub fn is_even(n: i32) -> bool { n % 2 == 0 }
pub fn factorial(n: u32) -> u64 {
    (1..=n as u64).product()
}
fn internal_helper(x: i32) -> i32 { x * 2 }  // private

#[cfg(test)]
mod tests {
    use super::*;  // import everything from parent module

    #[test]
    fn test_is_even() {
        assert!(is_even(4));
        assert!(!is_even(7));
        assert!(is_even(0));
        assert!(is_even(-2));
    }

    #[test]
    fn test_factorial() {
        assert_eq!(factorial(0), 1);
        assert_eq!(factorial(1), 1);
        assert_eq!(factorial(5), 120);
        assert_eq!(factorial(10), 3628800);
    }

    #[test]
    fn test_private_function() {
        // Unit tests CAN access private functions!
        assert_eq!(internal_helper(5), 10);
    }
}
```

---

## 3. Assert Macros

| Macro | Purpose | Example |
|---|---|---|
| `assert!(expr)` | Assert truthy | `assert!(x > 0)` |
| `assert_eq!(a, b)` | Assert equal | `assert_eq!(add(2,3), 5)` |
| `assert_ne!(a, b)` | Assert not equal | `assert_ne!(x, 0)` |
| All accept custom messages | Debug context | `assert!(x > 0, "x was {x}")` |

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn test_with_messages() {
        let result = 2 + 2;
        assert_eq!(result, 4, "Basic math failed: 2 + 2 = {result}");

        let name = "Alice";
        assert!(!name.is_empty(), "Name should not be empty");
        assert_ne!(name, "Bob", "Name should not be Bob");
    }

    #[test]
    fn test_floating_point() {
        let result = 0.1 + 0.2;
        // Don't use assert_eq! for floats — use epsilon comparison
        assert!((result - 0.3).abs() < f64::EPSILON * 4.0,
            "Float comparison: {result} ≈ 0.3");
    }
}
```

---

## 4. Testing for Panics

```rust
fn divide(a: i32, b: i32) -> i32 {
    if b == 0 { panic!("Division by zero!"); }
    a / b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn test_divide_by_zero() {
        divide(10, 0);  // should panic
    }

    #[test]
    #[should_panic(expected = "Division by zero")]
    fn test_divide_by_zero_message() {
        divide(10, 0);  // panic message must contain expected text
    }

    #[test]
    fn test_normal_divide() {
        assert_eq!(divide(10, 2), 5);
    }
}
```

---

## 5. Testing Results

Return `Result` from test functions:

```rust
use std::num::ParseIntError;

fn parse_port(s: &str) -> Result<u16, ParseIntError> {
    s.parse::<u16>()
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_parse_valid() -> Result<(), ParseIntError> {
        let port = parse_port("8080")?;  // use ? operator!
        assert_eq!(port, 8080);
        Ok(())
    }

    #[test]
    fn test_parse_invalid() {
        assert!(parse_port("abc").is_err());
        assert!(parse_port("-1").is_err());  // u16 can't be negative
        assert!(parse_port("99999").is_err());  // too large for u16
    }

    #[test]
    fn test_parse_error_type() {
        let err = parse_port("xyz").unwrap_err();
        assert_eq!(err.to_string(), "invalid digit found in string");
    }
}
```

---

## 6. Test Organization

### Project structure:

```
my_project/
├── src/
│   ├── lib.rs          ← library root
│   ├── math.rs         ← module with unit tests
│   └── utils.rs        ← module with unit tests
├── tests/              ← integration tests
│   ├── math_test.rs
│   └── utils_test.rs
└── Cargo.toml
```

### Module with tests:

```rust
// src/math.rs
pub fn gcd(mut a: u64, mut b: u64) -> u64 {
    while b != 0 {
        let temp = b;
        b = a % b;
        a = temp;
    }
    a
}

pub fn lcm(a: u64, b: u64) -> u64 {
    a / gcd(a, b) * b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_gcd() {
        assert_eq!(gcd(12, 8), 4);
        assert_eq!(gcd(54, 24), 6);
        assert_eq!(gcd(7, 13), 1);  // coprime
        assert_eq!(gcd(0, 5), 5);
    }

    #[test]
    fn test_lcm() {
        assert_eq!(lcm(4, 6), 12);
        assert_eq!(lcm(3, 5), 15);
    }
}
```

---

## 7. Integration Tests

Integration tests are external — they test your library's public API:

```rust
// tests/math_test.rs
use my_project::math;  // only public API available

#[test]
fn test_gcd_integration() {
    assert_eq!(math::gcd(100, 75), 25);
}

#[test]
fn test_lcm_integration() {
    assert_eq!(math::lcm(12, 18), 36);
}
```

### Shared test helpers:

```rust
// tests/common/mod.rs  ← shared helpers
pub fn setup() -> Vec<i32> {
    vec![1, 2, 3, 4, 5]
}

// tests/feature_test.rs
mod common;

#[test]
fn test_with_setup() {
    let data = common::setup();
    assert_eq!(data.len(), 5);
}
```

---

## 8. Test Configuration

### Ignoring tests:

```rust
#[test]
#[ignore]  // skip during normal `cargo test`
fn expensive_test() {
    // This takes 30 seconds to run
    std::thread::sleep(std::time::Duration::from_secs(30));
}
// Run with: cargo test -- --ignored
```

### Conditional compilation:

```rust
#[cfg(test)]
mod tests {
    // This entire module only exists during testing
    // Not included in release builds!
}
```

### Test setup/teardown:

```rust
struct TestDb { data: Vec<String> }

impl TestDb {
    fn setup() -> Self { TestDb { data: vec!["test".into()] } }
}

impl Drop for TestDb {
    fn drop(&mut self) { println!("Cleaning up test DB"); }
}

#[test]
fn test_with_db() {
    let db = TestDb::setup();
    assert_eq!(db.data.len(), 1);
    // db.drop() called automatically
}
```

---

## 9. Running Tests

```bash
# Run all tests
cargo test

# Run tests with a name filter
cargo test test_gcd

# Run tests in a specific module
cargo test math::tests

# Run ignored tests
cargo test -- --ignored

# Run all tests including ignored
cargo test -- --include-ignored

# Show println! output (normally captured)
cargo test -- --nocapture

# Run tests sequentially (default is parallel)
cargo test -- --test-threads=1

# Run only integration tests
cargo test --test math_test

# Run only doc tests
cargo test --doc
```

---

## 10. Summary Cheat Sheet

```
UNIT TESTS
────────────────────────────────────────────────────────────
#[cfg(test)]
mod tests {
    use super::*;
    #[test]
    fn test_name() { assert_eq!(f(), expected); }
}

ASSERT MACROS
────────────────────────────────────────────────────────────
assert!(expr)              bool assertion
assert_eq!(a, b)           equality
assert_ne!(a, b)           inequality
assert!(x, "msg {}", val)  custom message

PANICS & RESULTS
────────────────────────────────────────────────────────────
#[should_panic]             expect panic
#[should_panic(expected)]   expect specific message
fn test() -> Result<()>     use ? in tests

INTEGRATION TESTS
────────────────────────────────────────────────────────────
tests/my_test.rs            separate files
use my_crate::*;            public API only

RUNNING
────────────────────────────────────────────────────────────
cargo test                  all tests
cargo test name_filter      filter by name
cargo test -- --nocapture   show output
cargo test -- --ignored     ignored tests only
```

---

## What's Next?

**Lesson 70 — Testing: Mocking & Property-Based** — Mock dependencies, use `mockall`, and discover property-based testing with `proptest`.

## Further Reading
- [The Rust Book — Ch 11: Testing](https://doc.rust-lang.org/book/ch11-00-testing.html)
- [Rust by Example — Testing](https://doc.rust-lang.org/rust-by-example/testing.html)

---

*Testing: if it compiles AND the tests pass, it probably works! 🦀*
