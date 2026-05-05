# 🧪 Lesson 59 — Questions: Tokio Runtime (AS2)

> **Lesson:** [lesson_59_tokio.md](./lesson_59_tokio.md)  
> **Answers:** [lesson_59_answers.md](./lesson_59_answers.md)

---

## Section A — Conceptual

### Q1
What does `#[tokio::main]` actually do?

### Q2
What is the difference between `tokio::join!` and `tokio::select!`?

### Q3
Why must a future passed to `tokio::spawn` be `Send + 'static`?

---

## Section B — Write It Yourself

### Q4 — Concurrent tasks (Roadmap Practice Task)
Write 3 async functions that simulate different delay times (100ms, 200ms, 300ms). Run them concurrently with `tokio::join!` and measure the total time. Verify it's ~300ms, not ~600ms.

### Q5 — Timeout
Write an async function that takes too long (2 seconds). Wrap it with `tokio::time::timeout` set to 500ms. Handle the timeout error.

### Q6 — Async channel
Use `tokio::sync::mpsc` with 2 producer tasks and 1 consumer. Each producer sends 3 messages. Print all messages in the consumer.

---

## Section C — True or False?

### Q7
1. `#[tokio::main]` creates a multi-threaded runtime by default.
2. `tokio::spawn` is like `thread::spawn` but for async tasks.
3. `tokio::join!` cancels tasks that fail.
4. `tokio::select!` runs all branches to completion.
5. `tokio::time::sleep` blocks the OS thread.
6. `tokio::sync::oneshot` is for single-use request/response patterns.

---

*Tokio: making async Rust practical and powerful! 🦀*
