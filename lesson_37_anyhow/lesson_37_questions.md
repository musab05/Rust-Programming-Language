# 🧪 Lesson 37 — Questions: anyhow & error-stack (E5)

> **Lesson:** [lesson_37_anyhow.md](./lesson_37_anyhow.md)  
> **Answers:** [lesson_37_answers.md](./lesson_37_answers.md)

---

## Section A — Conceptual

### Q1
What is the difference between `anyhow::Result<T>` and `Result<T, Box<dyn std::error::Error>>`?

### Q2
Explain the difference between `.context("msg")` and `.with_context(|| format!("msg {}", var))`.

---

## Section B — Write It Yourself

### Q3 — Rewrite with anyhow (Roadmap Practice Task)
Rewrite this function using `anyhow`, adding meaningful context at each step:
```rust
fn process() -> Result<(), Box<dyn std::error::Error>> {
    let config = std::fs::read_to_string("config.txt")?;
    let port: u16 = config.trim().parse()?;
    let _addr = format!("0.0.0.0:{port}");
    Ok(())
}
```

### Q4 — bail! and ensure!
Write a `validate_email(email: &str) -> anyhow::Result<()>` that uses `ensure!` to check:
1. Contains exactly one `@`
2. Has content before the `@`
3. Has content after the `@`
4. The domain part contains a `.`

### Q5 — Downcasting
Given a custom `ApiError` enum, demonstrate creating it, wrapping in `anyhow::Error`, and downcasting to recover the original type.

---

## Section C — Decision Making

### Q6 — Choose the right tool
For each scenario, choose: **manual Error impl**, **thiserror**, or **anyhow**:
1. A public library crate for parsing CSV files
2. A CLI tool that reads config files and makes HTTP requests
3. A learning exercise to understand Rust error handling
4. An internal module in a large application
5. A one-off script that processes log files

### Q7 — True or False?
1. `anyhow::Error` preserves the original error type for downcasting.
2. `bail!("msg")` is equivalent to `return Err(anyhow!("msg"))`.
3. `anyhow` should be used in library crates.
4. `.context()` replaces the original error.
5. `ensure!(cond, "msg")` panics if the condition is false.

---

*Error handling: choose the right tool for each layer of your application! 🦀*
