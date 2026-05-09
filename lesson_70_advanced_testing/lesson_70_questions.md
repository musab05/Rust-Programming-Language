# 🧪 Lesson 70 — Questions: Mocking & Property-Based Testing (TE2)

> **Lesson:** [lesson_70_advanced_testing.md](./lesson_70_advanced_testing.md)  
> **Answers:** [lesson_70_answers.md](./lesson_70_answers.md)

---

## Section A — Conceptual

### Q1
What is the difference between a mock and a stub?

### Q2
Why is property-based testing valuable compared to example-based testing?

---

## Section B — Write It Yourself

### Q3 — Manual mock (Roadmap Practice Task)
Define a `Database` trait with `fn get(&self, key: &str) -> Option<String>` and `fn set(&mut self, key: &str, value: &str)`. Create a `MockDatabase` using a `HashMap`. Write a `Cache` struct that uses the trait and test it.

### Q4 — Property-based test
Write a property test that verifies: for any `Vec<i32>`, sorting the vec and then reversing it produces a descending-order vec.

### Q5 — Doc test
Write a function `fn clamp(val: i32, min: i32, max: i32) -> i32` with doc tests that demonstrate normal use and edge cases.

---

## Section C — True or False?

### Q6
1. `#[automock]` generates a mock struct for any trait.
2. Property-based tests only test a single specific input.
3. `proptest!` automatically shrinks failing inputs to minimal examples.
4. Doc tests (in `///` comments) are compiled and run by `cargo test`.
5. Manual mocks give you more control than auto-generated mocks.
6. `mockall` can verify that a method was called exactly N times.

---

*70 lessons down — you're a Rust expert now! 🦀🎉*
