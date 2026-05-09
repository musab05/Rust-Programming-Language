# 🧪 Lesson 69 — Questions: Testing (TE1)

> **Lesson:** [lesson_69_testing.md](./lesson_69_testing.md)  
> **Answers:** [lesson_69_answers.md](./lesson_69_answers.md)

---

## Section A — Predict

### Q1
```rust
fn divide(a: i32, b: i32) -> i32 { a / b }

#[test]
#[should_panic(expected = "by zero")]
fn test_div_zero() { divide(1, 0); }
```
Does this test pass?

### Q2
```rust
#[test]
fn test_result() -> Result<(), String> {
    let x: i32 = "42".parse().map_err(|e| format!("{e}"))?;
    assert_eq!(x, 42);
    Ok(())
}
```
Does this test pass?

---

## Section B — Write It Yourself

### Q3 — Calculator tests (Roadmap Practice Task)
Write a `Calculator` module with `add`, `subtract`, `multiply`, `divide` (returns `Result`). Write comprehensive unit tests including edge cases and error cases.

### Q4 — Integration test
Given a library with a public `fn process(input: &str) -> String`, write an integration test file in `tests/` that tests multiple inputs.

### Q5 — Ignored test
Write a test marked `#[ignore]` that simulates a slow operation. Explain how to run it.

---

## Section C — True or False?

### Q6
1. Unit tests can access private functions.
2. Integration tests live in the `tests/` directory.
3. `#[cfg(test)]` code is included in release builds.
4. `cargo test -- --nocapture` shows `println!` output.
5. Tests run in parallel by default.
6. `#[should_panic]` passes if the test function panics.

---

*Testing: Rust's built-in quality assurance! 🦀*
