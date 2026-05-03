# 🧪 Lesson 47 — Questions: Publishing to crates.io (M5)

> **Lesson:** [lesson_47_publishing.md](./lesson_47_publishing.md)  
> **Answers:** [lesson_47_answers.md](./lesson_47_answers.md)

---

## Section A — Conceptual

### Q1
What are the **required** fields in `Cargo.toml` for publishing to crates.io?

### Q2
What is the difference between `///` and `//!` doc comments?

### Q3
What does `cargo publish --dry-run` do?

---

## Section B — Write It Yourself

### Q4 — Document a function (Roadmap Practice Task)
Write a fully documented `pub fn is_palindrome(s: &str) -> bool` function with:
- Brief description
- `# Arguments` section
- `# Examples` section with a working doc test
- `# Returns` section

### Q5 — Cargo.toml metadata
Write a complete `[package]` section for a crate called `text-utils` that provides text processing utilities.

---

## Section C — True or False?

### Q6
1. Doc tests in `///` blocks are automatically run by `cargo test`.
2. `cargo yank` deletes a version from crates.io permanently.
3. Version `0.2.0 → 0.3.0` can include breaking changes (pre-1.0).
4. You need a GitHub account to publish to crates.io.
5. `cargo doc --open` builds documentation and opens it in a browser.
6. Examples go in an `examples/` directory and are run with `cargo run --example name`.

---

*Publishing: share your Rust library with 2 million developers! 🦀*
