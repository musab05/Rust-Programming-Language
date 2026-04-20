# ✅ Lesson 27 — Answers: Testing (B2)

---

## Section A

### A1 — ✅ Pass
All assertions are correct: 2+2=4, 2+2≠5, true is true.

### A2 — ❌ Fail
`"Hello, Rust!"` has **12** characters, not 11. `assert_eq!(greeting.len(), 11)` fails. Fix: `assert_eq!(greeting.len(), 12)`.

### A3 — ✅ Pass
`v[10]` on a 3-element vec panics with "index out of bounds". `#[should_panic]` expects a panic → test passes.

### A4 — ❌ Fail
The panic message is `"something went wrong"`, but `expected = "divide by zero"`. The expected string is not found in the panic message, so the `#[should_panic]` test fails.

---

## Section B

### A5 — ✅ Yes
Tests can return `Result<(), E>`. The `?` operator works inside tests. This compiles and passes.

### A6 — ❌ No
`assert_eq!` requires `Debug + PartialEq`. `Point` only derives `PartialEq`, not `Debug`. Fix: `#[derive(Debug, PartialEq)]`.

### A7 — ✅ Yes
Private functions are accessible from child modules (including `#[cfg(test)] mod tests`). `use super::*` imports `private_fn`. Compiles and passes.

---

## Section C

### A8
```rust
fn is_even(n: i32) -> bool {
    n % 2 == 0
}

#[cfg(test)]
mod tests {
    use super::*;  // fix: import parent items

    #[test]
    fn test_is_even() {
        assert!(is_even(4));
        assert!(!is_even(3));
    }
}
```

### A9
```rust
fn divide(a: i32, b: i32) -> i32 {
    a / b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_divide_normal() {           // fix: removed #[should_panic]
        assert_eq!(divide(10, 2), 5);
    }

    #[test]
    #[should_panic]                      // fix: added #[should_panic]
    fn test_divide_by_zero() {
        divide(10, 0);
    }
}
```

---

## Section D

### A10 — Vec utilities with full coverage
```rust
fn sum(v: &[i32]) -> i32 {
    v.iter().sum()
}

fn mean(v: &[f64]) -> Option<f64> {
    if v.is_empty() {
        None
    } else {
        Some(v.iter().sum::<f64>() / v.len() as f64)
    }
}

fn median(v: &mut Vec<f64>) -> Option<f64> {
    if v.is_empty() {
        return None;
    }
    v.sort_by(|a, b| a.partial_cmp(b).unwrap());
    let mid = v.len() / 2;
    if v.len() % 2 == 0 {
        Some((v[mid - 1] + v[mid]) / 2.0)
    } else {
        Some(v[mid])
    }
}

fn min_max(v: &[i32]) -> Option<(i32, i32)> {
    if v.is_empty() {
        return None;
    }
    let min = *v.iter().min().unwrap();
    let max = *v.iter().max().unwrap();
    Some((min, max))
}

fn contains_duplicates(v: &[i32]) -> bool {
    let mut seen = std::collections::HashSet::new();
    for item in v {
        if !seen.insert(item) {
            return true;
        }
    }
    false
}

#[cfg(test)]
mod tests {
    use super::*;

    // sum tests
    #[test]
    fn test_sum_normal() { assert_eq!(sum(&[1, 2, 3, 4, 5]), 15); }
    #[test]
    fn test_sum_empty() { assert_eq!(sum(&[]), 0); }
    #[test]
    fn test_sum_negative() { assert_eq!(sum(&[-1, -2, 3]), 0); }

    // mean tests
    #[test]
    fn test_mean_normal() { assert_eq!(mean(&[2.0, 4.0, 6.0]), Some(4.0)); }
    #[test]
    fn test_mean_empty() { assert_eq!(mean(&[]), None); }
    #[test]
    fn test_mean_single() { assert_eq!(mean(&[7.0]), Some(7.0)); }

    // median tests
    #[test]
    fn test_median_odd() {
        assert_eq!(median(&mut vec![3.0, 1.0, 2.0]), Some(2.0));
    }
    #[test]
    fn test_median_even() {
        assert_eq!(median(&mut vec![4.0, 1.0, 3.0, 2.0]), Some(2.5));
    }
    #[test]
    fn test_median_empty() {
        assert_eq!(median(&mut vec![]), None);
    }
    #[test]
    fn test_median_single() {
        assert_eq!(median(&mut vec![42.0]), Some(42.0));
    }

    // min_max tests
    #[test]
    fn test_min_max_normal() {
        assert_eq!(min_max(&[3, 1, 4, 1, 5, 9]), Some((1, 9)));
    }
    #[test]
    fn test_min_max_empty() { assert_eq!(min_max(&[]), None); }
    #[test]
    fn test_min_max_single() { assert_eq!(min_max(&[42]), Some((42, 42))); }

    // contains_duplicates tests
    #[test]
    fn test_duplicates_yes() { assert!(contains_duplicates(&[1, 2, 3, 2])); }
    #[test]
    fn test_duplicates_no() { assert!(!contains_duplicates(&[1, 2, 3, 4])); }
    #[test]
    fn test_duplicates_empty() { assert!(!contains_duplicates(&[])); }
}
```

### A11 — Calculator tests
```rust
fn add(a: f64, b: f64) -> f64 { a + b }
fn subtract(a: f64, b: f64) -> f64 { a - b }
fn multiply(a: f64, b: f64) -> f64 { a * b }

fn divide(a: f64, b: f64) -> Result<f64, String> {
    if b == 0.0 {
        Err(String::from("Division by zero"))
    } else {
        Ok(a / b)
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_add() {
        assert_eq!(add(2.0, 3.0), 5.0);
        assert_eq!(add(-1.0, 1.0), 0.0);
        assert_eq!(add(0.0, 0.0), 0.0);
    }

    #[test]
    fn test_subtract() {
        assert_eq!(subtract(10.0, 3.0), 7.0);
        assert_eq!(subtract(5.0, 5.0), 0.0);
        assert_eq!(subtract(0.0, 5.0), -5.0);
    }

    #[test]
    fn test_multiply() {
        assert_eq!(multiply(3.0, 4.0), 12.0);
        assert_eq!(multiply(-2.0, 3.0), -6.0);
        assert_eq!(multiply(5.0, 0.0), 0.0);
    }

    #[test]
    fn test_divide_success() {
        assert_eq!(divide(10.0, 2.0), Ok(5.0));
        assert_eq!(divide(-6.0, 3.0), Ok(-2.0));
    }

    #[test]
    fn test_divide_by_zero() {
        assert!(divide(10.0, 0.0).is_err());
        assert_eq!(
            divide(10.0, 0.0),
            Err(String::from("Division by zero"))
        );
    }
}
```

### A12 — should_panic tests
```rust
fn validate_age(age: i32) {
    if age < 0 {
        panic!("Age cannot be negative: {age}");
    }
    if age > 150 {
        panic!("Age cannot exceed 150: {age}");
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_valid_age() {
        validate_age(0);
        validate_age(25);
        validate_age(150);
        // No panic = test passes
    }

    #[test]
    #[should_panic]
    fn test_negative_age() {
        validate_age(-1);
    }

    #[test]
    #[should_panic(expected = "cannot exceed 150")]
    fn test_too_old() {
        validate_age(200);
    }
}
```

---

## Section E

### A13 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Tests can return `Result<(), E>` to use `?` |
| 2 | **True** | A test fails when the function panics (assertion failure, unwrap, etc.) |
| 3 | **True** | `assert_eq!` prints both values on failure, which requires `Debug` |
| 4 | **False** | `#[cfg(test)]` is only included when running `cargo test` |
| 5 | **False** | Integration tests (`tests/` dir) can only access the public API |
| 6 | **True** | `cargo test foo` runs all tests whose names contain "foo" |
| 7 | **True** | `#[should_panic]` tests fail if no panic occurs |
| 8 | **False** | `#[ignore]` tests can be run with `--ignored` or `--include-ignored` |
| 9 | **True** | `--show-output` displays captured stdout for all tests including passing |
| 10 | **False** | Test functions take no parameters — they must be `fn()` or `fn() -> Result` |

### A14 — Explanation
Rust allows testing private functions because the test module (`mod tests`) is a **child** of the module containing the private function. In Rust's visibility rules, child modules can access parent items.

**When to test private functions:**
- When the private function has complex logic worth verifying independently
- When bugs in the private function would be hard to diagnose through the public API

**When to test only the public API:**
- When private functions are simple helpers
- When testing the public API naturally exercises the private code
- For integration tests (which can only see the public API by design)

Both approaches are valid. A common practice: test the public API thoroughly, and test complex private functions individually.

---

## 🏆 Lesson 27 Complete!

✅ `#[test]` attribute for marking tests  
✅ `assert!`, `assert_eq!`, `assert_ne!` with custom messages  
✅ `#[should_panic]` for testing panic behaviour  
✅ `Result`-returning tests with `?`  
✅ `cargo test` options and filtering  
✅ Unit tests (`#[cfg(test)]`) vs integration tests (`tests/`)  
✅ `#[ignore]` for slow tests  
✅ Testing private functions  

**Up next: Lesson 28 — Clippy & rustfmt 🦀**
