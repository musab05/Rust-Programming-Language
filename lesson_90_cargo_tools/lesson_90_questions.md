# 🧪 Lesson 90 — Questions: cargo-expand & cargo-asm (B6)

> **Lesson:** [lesson_90_cargo_tools.md](./lesson_90_cargo_tools.md)  
> **Answers:** [lesson_90_answers.md](./lesson_90_answers.md)

---

## Section A — Conceptual

### Q1
What is the difference between `cargo expand` and `cargo asm`?

### Q2
Why does `cargo expand` require nightly Rust?

---

## Section B — Write It Yourself

### Q3 — Derive expansion (Roadmap Practice Task)
Create a struct with `#[derive(Debug, Clone, PartialEq)]`. Describe what code each derive generates (without actually running cargo expand — use your knowledge from this lesson).

### Q4 — Assembly verification
Write two versions of a sum function (iterator vs loop). Explain why their assembly output should be similar.

---

## Section C — True or False?

### Q5
1. `cargo expand` shows the code after macro expansion.
2. `cargo asm` shows LLVM IR.
3. `cargo expand` requires nightly Rust.
4. Iterators and manual loops typically produce different assembly.
5. `godbolt.org` can show Rust assembly interactively.
6. `call core::panicking::panic_bounds_check` in assembly means a bounds check was NOT elided.

---

*See the code behind the code! 🦀*
