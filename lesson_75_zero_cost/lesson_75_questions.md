# 🧪 Lesson 75 — Questions: Zero-Cost Abstractions (P2)

> **Lesson:** [lesson_75_zero_cost.md](./lesson_75_zero_cost.md)  
> **Answers:** [lesson_75_answers.md](./lesson_75_answers.md)

---

## Section A — Conceptual

### Q1
What does "zero-cost abstraction" mean? State the two-part principle.

### Q2
What is monomorphization? Give an example of a generic function and what the compiler generates.

---

## Section B — Write It Yourself

### Q3 — Iterator vs loop comparison (Roadmap Practice Task)
Write an iterator chain version and a manual loop version that: filter even numbers from a `Vec<i32>`, square them, and sum the result. Time both.

### Q4 — Static vs dynamic dispatch
Write a trait `Greet` with a method `fn hello(&self) -> String`. Create two structs. Compare calling via `impl Greet` (static) vs `&dyn Greet` (dynamic). Explain the performance difference.

---

## Section C — True or False?

### Q5
1. Iterator chains in Rust have the same runtime cost as hand-written loops.
2. Monomorphization increases binary size.
3. `dyn Trait` is a zero-cost abstraction.
4. Closures that capture no variables are zero-sized.
5. `#[inline(always)]` guarantees the function will be inlined.
6. `Option<&T>` and `&T` have the same size due to null pointer optimization.

---

*Zero-cost: high-level elegance, low-level speed! 🦀*
