# 🧪 Lesson 94 — Questions: Database with SQLx (RW6)

> **Lesson:** [lesson_94_sqlx.md](./lesson_94_sqlx.md)  
> **Answers:** [lesson_94_answers.md](./lesson_94_answers.md)

---

## Section A — Conceptual

### Q1
What makes `sqlx::query!()` different from `sqlx::query()`?

### Q2
What happens if a transaction is dropped without calling `.commit()`?

---

## Section B — Write It Yourself

### Q3 — User CRUD (Roadmap Practice Task)
Write `create_user`, `get_user`, `list_users`, `update_user`, and `delete_user` functions using SQLx with SQLite.

### Q4 — Axum + SQLx integration
Build a minimal Axum server with `GET /users` and `POST /users` backed by SQLite via SQLx.

---

## Section C — True or False?

### Q5
1. `sqlx::query!()` checks SQL syntax at compile time.
2. SQLx is an ORM like Diesel.
3. `#[derive(FromRow)]` maps database rows to Rust structs.
4. `pool.begin()` starts a database transaction.
5. SQLx supports PostgreSQL, MySQL, and SQLite.
6. Connection pools must be manually closed before program exit.

---

*SQLx: type-safe SQL with async superpowers! 🦀*
