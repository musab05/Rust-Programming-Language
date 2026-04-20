# 🧪 Lesson 27 — Questions: Testing (B2)

> **Lesson:** [lesson_27_testing.md](./lesson_27_testing.md)  
> **Answers:** [lesson_27_answers.md](./lesson_27_answers.md)

---

## Section A — Predict the Result (Pass or Fail?)

### Q1
```rust
#[test]
fn test_basic() {
    assert_eq!(2 + 2, 4);
    assert_ne!(2 + 2, 5);
    assert!(true);
}
```

### Q2
```rust
#[test]
fn test_string() {
    let greeting = String::from("Hello, Rust!");
    assert!(greeting.contains("Rust"));
    assert_eq!(greeting.len(), 11);  // is this right?
}
```

### Q3
```rust
#[test]
#[should_panic]
fn test_out_of_bounds() {
    let v = vec![1, 2, 3];
    let _ = v[10];
}
```

### Q4
```rust
#[test]
#[should_panic(expected = "divide by zero")]
fn test_wrong_panic_message() {
    panic!("something went wrong");
}
```

---

## Section B — Will It Compile?

### Q5
```rust
#[test]
fn test_result() -> Result<(), String> {
    let n: i32 = "42".parse().map_err(|e| format!("{e}"))?;
    assert_eq!(n, 42);
    Ok(())
}
```

### Q6
```rust
#[derive(PartialEq)]
struct Point { x: i32, y: i32 }

#[test]
fn test_point() {
    assert_eq!(Point { x: 1, y: 2 }, Point { x: 1, y: 2 });
}
```

### Q7
```rust
fn private_fn() -> i32 { 42 }

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_private() {
        assert_eq!(private_fn(), 42);
    }
}
```

---

## Section C — Fix the Bugs

### Q8
```rust
fn is_even(n: i32) -> bool {
    n % 2 == 0
}

#[cfg(test)]
mod tests {
    #[test]
    fn test_is_even() {
        assert!(is_even(4));     // BUG: is_even not in scope
        assert!(!is_even(3));
    }
}
```

### Q9
```rust
fn divide(a: i32, b: i32) -> i32 {
    a / b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn test_divide_normal() {    // BUG: this test shouldn't expect a panic
        assert_eq!(divide(10, 2), 5);
    }

    #[test]
    fn test_divide_by_zero() {   // BUG: this test should expect a panic
        divide(10, 0);
    }
}
```

---

## Section D — Write It Yourself

### Q10 — Vec utilities with 100% coverage (Roadmap Practice Task)
Write the following functions and test each one thoroughly:
```rust
fn sum(v: &[i32]) -> i32
fn mean(v: &[f64]) -> Option<f64>           // None for empty
fn median(v: &mut Vec<f64>) -> Option<f64>  // None for empty
fn min_max(v: &[i32]) -> Option<(i32, i32)> // None for empty
fn contains_duplicates(v: &[i32]) -> bool
```
Write at least 2 tests per function covering normal cases and edge cases (empty vec, single element, even/odd length for median).

### Q11 — Test a calculator
Write `add`, `subtract`, `multiply`, `divide` functions (where divide returns `Result<f64, String>`). Write tests for:
- Normal operations
- Division by zero (should return Err)
- Edge cases (zero, negative numbers)

### Q12 — should_panic tests
Write a function `fn validate_age(age: i32)` that panics if age < 0 or age > 150. Write:
- A test that passes for valid age (no panic)
- A `#[should_panic]` test for negative age
- A `#[should_panic(expected = "...")]` test for age > 150

---

## Section E — Deep Understanding

### Q13 — True or False?
1. `#[test]` functions must return `()`.
2. A test fails only when it panics.
3. `assert_eq!` requires both values to implement `Debug`.
4. `#[cfg(test)]` code is included in release builds.
5. Integration tests can test private functions.
6. `cargo test name` runs only tests whose names contain "name".
7. `#[should_panic]` tests fail if the code does NOT panic.
8. `#[ignore]` permanently disables a test.
9. `cargo test -- --show-output` shows `println!` output even for passing tests.
10. Test functions can take parameters.

### Q14 — Explain
Why does Rust allow testing private functions? Is this a good practice? When would you test private functions vs only testing the public API?

---

*Write tests early, test often, ship with confidence! 🦀*
