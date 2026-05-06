# 🧪 Lesson 63 — Questions: Procedural Macros (MC2)

> **Lesson:** [lesson_63_proc_macros.md](./lesson_63_proc_macros.md)  
> **Answers:** [lesson_63_answers.md](./lesson_63_answers.md)

---

## Section A — Conceptual

### Q1
What are the three types of procedural macros? Give the syntax for each.

### Q2
What do the `syn` and `quote` crates do?

### Q3
Why must proc macros live in a separate crate with `proc-macro = true`?

---

## Section B — Write It Yourself

### Q4 — Using derives (Roadmap Practice Task)
Create a struct `Config` with `host`, `port`, and `debug` fields. Apply appropriate standard derives AND serde derives. Serialize it to JSON and deserialize it back.

### Q5 — Derive macro structure
Write the skeleton (not full implementation) of a derive macro called `Summary` that would add a `fn summary(&self) -> String` method. Show the proc-macro crate structure.

---

## Section C — True or False?

### Q6
1. `#[derive(Debug)]` is a procedural macro.
2. Proc macros can run arbitrary Rust code at compile time.
3. Derive macros can only implement existing standard library traits.
4. `quote!` generates Rust source code as a `TokenStream`.
5. Attribute macros receive both the attribute arguments AND the item they decorate.
6. You can define proc macros in the same crate that uses them.

---

*Proc macros: compile-time code generation at its finest! 🦀*
