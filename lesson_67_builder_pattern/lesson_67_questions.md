# 🧪 Lesson 67 — Questions: Builder Pattern (DP1)

> **Lesson:** [lesson_67_builder_pattern.md](./lesson_67_builder_pattern.md)  
> **Answers:** [lesson_67_answers.md](./lesson_67_answers.md)

---

## Section A — Conceptual

### Q1
Why use the Builder pattern instead of a constructor with many parameters?

### Q2
What is the difference between a basic builder (`self`) and a fluent builder (`&mut self`)?

---

## Section B — Write It Yourself

### Q3 — Config builder (Roadmap Practice Task)
Build a `DatabaseConfig` builder with required fields (`host`, `port`, `database`) and optional fields (`username`, `password`, `pool_size`, `timeout_ms`). Validate that port > 0 and pool_size > 0 in `build()`.

### Q4 — Type-safe builder
Write a builder where `build()` only compiles if BOTH `name` and `email` have been set. Use PhantomData.

---

## Section C — True or False?

### Q5
1. The Builder pattern is useful when a struct has many optional fields.
2. `fn field(mut self, v: T) -> Self` consumes the builder on each call.
3. Type-safe builders catch missing required fields at runtime.
4. `derive_builder` auto-generates builder code via a proc macro.
5. Builders can return `Result` from `build()` for validation.

---

*Builder: complex construction made simple! 🦀*
