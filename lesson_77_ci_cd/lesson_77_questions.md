# 🧪 Lesson 77 — Questions: CI/CD with GitHub Actions (B8)

> **Lesson:** [lesson_77_ci_cd.md](./lesson_77_ci_cd.md)  
> **Answers:** [lesson_77_answers.md](./lesson_77_answers.md)

---

## Section A — Conceptual

### Q1
What is the difference between CI and CD?

### Q2
Why test on multiple toolchains (stable, beta, nightly, MSRV)?

---

## Section B — Write It Yourself

### Q3 — Rust CI workflow (Roadmap Practice Task)
Write a GitHub Actions workflow that: runs `cargo fmt --check`, `cargo clippy`, and `cargo test` on stable, nightly, and MSRV 1.75.0.

### Q4 — Caching
Add dependency caching to a workflow using `Swatinem/rust-cache@v2`.

---

## Section C — True or False?

### Q5
1. GitHub Actions workflows are defined in `.github/workflows/`.
2. `cargo clippy -- -D warnings` treats clippy warnings as errors.
3. Matrix strategies run jobs sequentially, one after another.
4. `Swatinem/rust-cache` caches the `target/` directory and cargo registry.
5. Release workflows can trigger on git tag pushes.
6. `cargo fmt -- --check` modifies files to fix formatting.

---

*CI/CD: automate quality, ship with confidence! 🦀*
