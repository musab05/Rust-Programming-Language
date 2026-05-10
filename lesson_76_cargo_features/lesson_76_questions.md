# 🧪 Lesson 76 — Questions: Cargo Features & Profiles (B1)

> **Lesson:** [lesson_76_cargo_features.md](./lesson_76_cargo_features.md)  
> **Answers:** [lesson_76_answers.md](./lesson_76_answers.md)

---

## Section A — Conceptual

### Q1
What is the difference between `#[cfg(feature = "x")]` and `cfg!(feature = "x")`?

### Q2
What does `lto = true` do in a release profile? What's the trade-off?

---

## Section B — Write It Yourself

### Q3 — Feature flag (Roadmap Practice Task)
Write a `Cargo.toml` with a `logging` feature (default off) that optionally depends on the `log` crate. Write code that uses `#[cfg(feature = "logging")]` to conditionally log.

### Q4 — Custom profiles
Configure `Cargo.toml` so dev builds use `opt-level = 1` and release builds use `lto = true`, `codegen-units = 1`, and `panic = "abort"`.

---

## Section C — True or False?

### Q5
1. `cargo build` uses `[profile.dev]` by default.
2. Features can enable optional dependencies.
3. `--no-default-features` disables all features including explicitly requested ones.
4. `#[cfg(feature = "x")]` removes code entirely at compile time.
5. `opt-level = 3` produces the fastest binary but slowest compile.
6. Features are resolved at runtime.

---

*Cargo: your build system is your superpower! 🦀*
