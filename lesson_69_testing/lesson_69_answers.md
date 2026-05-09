# ✅ Lesson 69 — Answers: Testing (TE1)

---

## Section A

### A1 — ✅ Passes
Integer division by zero panics with `attempt to divide by zero` in Rust, which contains `"by zero"`. The `expected` substring matches.

### A2 — ✅ Passes
`"42".parse::<i32>()` succeeds, `x` is 42, `assert_eq!` passes, `Ok(())` returned.

---

## Section B

### A3
```rust
mod calculator {
    pub fn add(a: f64, b: f64) -> f64 { a + b }
    pub fn subtract(a: f64, b: f64) -> f64 { a - b }
    pub fn multiply(a: f64, b: f64) -> f64 { a * b }
    pub fn divide(a: f64, b: f64) -> Result<f64, String> {
        if b == 0.0 { Err("Division by zero".into()) }
        else { Ok(a / b) }
    }
}

#[cfg(test)]
mod tests {
    use super::calculator::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2.0, 3.0), 5.0);
        assert_eq!(add(-1.0, 1.0), 0.0);
        assert_eq!(add(0.0, 0.0), 0.0);
    }

    #[test]
    fn test_subtract() {
        assert_eq!(subtract(5.0, 3.0), 2.0);
        assert_eq!(subtract(3.0, 5.0), -2.0);
    }

    #[test]
    fn test_multiply() {
        assert_eq!(multiply(3.0, 4.0), 12.0);
        assert_eq!(multiply(0.0, 100.0), 0.0);
        assert_eq!(multiply(-2.0, 3.0), -6.0);
    }

    #[test]
    fn test_divide_ok() {
        assert_eq!(divide(10.0, 2.0).unwrap(), 5.0);
        assert_eq!(divide(-6.0, 3.0).unwrap(), -2.0);
    }

    #[test]
    fn test_divide_by_zero() {
        let err = divide(10.0, 0.0).unwrap_err();
        assert_eq!(err, "Division by zero");
    }
}
```

### A4
```rust
// tests/process_test.rs
use my_crate::process;

#[test]
fn test_process_normal() {
    assert_eq!(process("hello"), "HELLO");  // example behavior
}

#[test]
fn test_process_empty() {
    assert_eq!(process(""), "");
}

#[test]
fn test_process_special_chars() {
    let result = process("hello, world!");
    assert!(!result.is_empty());
}
```

### A5
```rust
#[test]
#[ignore]
fn slow_network_test() {
    std::thread::sleep(std::time::Duration::from_secs(10));
    assert!(true);
}
// Run with: cargo test -- --ignored
// Or: cargo test -- --include-ignored
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Unit tests in the same file/module can access private items via `use super::*` |
| 2 | **True** | Integration tests go in the `tests/` directory at crate root |
| 3 | **False** | `#[cfg(test)]` is ONLY compiled during `cargo test` |
| 4 | **True** | `--nocapture` disables output capture, showing println output |
| 5 | **True** | Tests run in parallel by default (control with `--test-threads`) |
| 6 | **True** | `#[should_panic]` marks the test as expected to panic |

---

## 🏆 Lesson 69 Complete!

**Next up:** [Lesson 70 — Testing: Mocking & Property-Based](../lesson_70_advanced_testing/lesson_70_advanced_testing.md) 🦀
