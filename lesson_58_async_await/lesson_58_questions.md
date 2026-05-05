# 🧪 Lesson 58 — Questions: Async/Await Basics (AS1)

> **Lesson:** [lesson_58_async_await.md](./lesson_58_async_await.md)  
> **Answers:** [lesson_58_answers.md](./lesson_58_answers.md)

---

## Section A — Conceptual

### Q1
What does `async fn fetch() -> String` actually return?

### Q2
True or false: Calling an `async fn` immediately executes its body.

### Q3
Where can `.await` be used?

---

## Section B — Write It Yourself

### Q4 — Compose async functions (Roadmap Practice Task)
Write three async functions: `fetch_user(id)`, `fetch_profile(user)`, and `fetch_avatar(profile)`. Chain them sequentially using `.await`. Describe what a runtime would need to do to execute them.

### Q5 — async blocks
Write an async block that captures a `Vec<i32>`, computes the sum, and returns it. Explain why `async move` is needed.

---

## Section C — True or False?

### Q6
1. Rust futures are eagerly executed (like JavaScript promises).
2. `async fn` can only be called from other `async fn` or `async` blocks.
3. The standard library includes a built-in async runtime.
4. `Poll::Pending` means the future is done.
5. You can have millions of async tasks on a single thread.
6. `Pin` prevents a future from being moved in memory.
7. `.await` blocks the entire OS thread.

### Q7
In your own words, explain the difference between threads and async for handling 10,000 concurrent HTTP connections.

---

*Async: the art of waiting efficiently! 🦀*
