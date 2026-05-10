# 🧪 Lesson 81 — Questions: HTTP Client with reqwest (RW4)

> **Lesson:** [lesson_81_reqwest.md](./lesson_81_reqwest.md)  
> **Answers:** [lesson_81_answers.md](./lesson_81_answers.md)

---

## Section A — Conceptual

### Q1
Why should you reuse a `reqwest::Client` instead of creating a new one per request?

### Q2
What does `error_for_status()` do?

---

## Section B — Write It Yourself

### Q3 — Weather app (Roadmap Practice Task)
Write an async program that fetches a JSON response from a public API (e.g., `jsonplaceholder.typicode.com/users`), deserializes it into a struct, and prints selected fields.

### Q4 — Concurrent fetches
Fetch 5 URLs concurrently using `join_all` and print results with timing.

---

## Section C — True or False?

### Q5
1. `reqwest::get()` is a convenience function that creates a temporary client.
2. `.json::<T>()` deserializes the response body into type `T`.
3. reqwest is synchronous by default.
4. `Client::builder().timeout()` sets a per-request timeout.
5. `.bearer_auth("token")` adds an `Authorization: Bearer` header.
6. POST requests can send JSON with `.json(&data)`.

---

*reqwest: Rust talks to the web! 🦀*
