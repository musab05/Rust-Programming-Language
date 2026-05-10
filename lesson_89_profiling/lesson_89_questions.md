# 🧪 Lesson 89 — Questions: Profiling (P4)

> **Lesson:** [lesson_89_profiling.md](./lesson_89_profiling.md)  
> **Answers:** [lesson_89_answers.md](./lesson_89_answers.md)

---

## Section A — Conceptual

### Q1
What's the difference between benchmarking and profiling? When do you use each?

### Q2
How do you read a flamegraph? What do the X and Y axes represent?

---

## Section B — Write It Yourself

### Q3 — Manual profiling (Roadmap Practice Task)
Write a program with 3 phases (parse, sort, aggregate). Use `Instant::now()` to time each phase. Identify which phase is the bottleneck.

### Q4 — Allocation counting
Create a custom global allocator that counts allocations. Compare `Vec::new()` with push vs `Vec::with_capacity()`.

---

## Section C — True or False?

### Q5
1. You should profile release builds, not debug builds.
2. `[profile.release] debug = true` keeps debug symbols for profiling.
3. `cargo flamegraph` generates a PNG image.
4. Wide bars in a flamegraph indicate time-consuming functions.
5. `perf stat` shows CPU counters like cache misses.
6. Profiling and benchmarking should always be done together.

---

*Profile first, optimize second! 🦀*
