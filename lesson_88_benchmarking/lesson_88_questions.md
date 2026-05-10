# 🧪 Lesson 88 — Questions: Benchmarking (P3)

> **Lesson:** [lesson_88_benchmarking.md](./lesson_88_benchmarking.md)  
> **Answers:** [lesson_88_answers.md](./lesson_88_answers.md)

---

## Section A — Conceptual

### Q1
Why is `black_box()` necessary in benchmarks?

### Q2
What does `iter_batched` do and when would you use it instead of `iter`?

---

## Section B — Write It Yourself

### Q3 — Sorting benchmark (Roadmap Practice Task)
Write a Criterion benchmark that compares `Vec::sort()`, `Vec::sort_unstable()`, and a custom insertion sort on vectors of 1000, 5000, and 10000 elements.

---

## Section C — True or False?

### Q4
1. Criterion provides statistical analysis including confidence intervals.
2. `cargo bench` runs benchmarks defined in the `benches/` directory.
3. `black_box` prevents the compiler from optimizing away benchmark code.
4. Criterion can detect performance regressions across runs.
5. HTML reports are generated in `target/criterion/report/`.
6. `harness = false` means Criterion uses Rust's built-in test harness.

---

*Benchmark: proof beats assumption! 🦀*
