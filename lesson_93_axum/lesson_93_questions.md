# 🧪 Lesson 93 — Questions: Web Server with Axum (RW5)

> **Lesson:** [lesson_93_axum.md](./lesson_93_axum.md)  
> **Answers:** [lesson_93_answers.md](./lesson_93_answers.md)

---

## Section A — Conceptual

### Q1
What is an extractor in Axum? Name 4 built-in extractors.

### Q2
Why use `Arc<Mutex<T>>` for shared state instead of plain `T`?

---

## Section B — Write It Yourself

### Q3 — Todo CRUD API (Roadmap Practice Task)
Build a REST API with: `GET /todos` (list), `POST /todos` (create), `PUT /todos/:id` (update), `DELETE /todos/:id` (remove). Use in-memory `Vec` with shared state.

### Q4 — Custom middleware
Write a logging middleware that prints method, path, status code, and elapsed time for every request.

---

## Section C — True or False?

### Q5
1. Axum handlers are async functions.
2. `Path(id): Path<u32>` extracts a URL parameter as `u32`.
3. `Json(body): Json<T>` requires `T: Serialize`.
4. `.with_state()` attaches shared state to a router.
5. Axum is built on top of the `tower` middleware ecosystem.
6. `Router::new().nest("/api", sub)` prefixes all sub-routes with `/api`.

---

*Axum: type-safe web servers! 🦀*
