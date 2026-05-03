# 🧪 Lesson 46 — Questions: Workspaces (M4)

> **Lesson:** [lesson_46_workspaces.md](./lesson_46_workspaces.md)  
> **Answers:** [lesson_46_answers.md](./lesson_46_answers.md)

---

## Section A — Conceptual

### Q1
What do all crates in a workspace share?

### Q2
What is the purpose of `{ workspace = true }` in a member's `Cargo.toml`?

### Q3
How do you reference another crate within the same workspace?

---

## Section B — Write It Yourself

### Q4 — Design a workspace (Roadmap Practice Task)
Design the directory structure and `Cargo.toml` files for a workspace with:
1. `math_core` — a library with basic math functions
2. `math_cli` — a CLI that uses `math_core`
3. `math_api` — a web API that uses `math_core`

Shared dependencies: `serde = "1"` and `anyhow = "1"`.

### Q5
Write the root `Cargo.toml` for a workspace with members `lib`, `server`, and `tools`, sharing `tokio = { version = "1", features = ["full"] }`.

---

## Section C — True or False?

### Q6
1. A workspace root `Cargo.toml` has a `[package]` section.
2. All crates in a workspace share one `Cargo.lock`.
3. You can run a specific binary with `cargo run -p crate_name`.
4. Each crate in a workspace has its own `target/` directory.
5. `{ path = "../sibling" }` allows one crate to depend on a sibling in the workspace.
6. Workspace members must all be libraries.

---

*Workspaces: one project, many crates, zero chaos! 🦀*
