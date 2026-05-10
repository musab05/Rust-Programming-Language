# 🧪 Lesson 97 — Questions & ✅ Answers: Proc Macros (MA3)

> **Lesson:** [lesson_97_proc_macros.md](./lesson_97_proc_macros.md)

---

## Q1 — What are the three kinds of proc macros?
**A:** Derive (`#[derive(X)]`), Attribute (`#[attr] fn ...`), Function-like (`my_macro!(...)`).

## Q2 — `#[log_calls]` attribute macro (Roadmap Practice Task)
Create an attribute that prints function name on entry/exit. See lesson for full implementation.

## Q3 — True or False?
| # | Statement | Answer |
|---|-----------|--------|
| 1 | Proc macros must live in a separate crate | **True** |
| 2 | `syn` parses TokenStreams into ASTs | **True** |
| 3 | `quote!` generates Rust code at runtime | **False** — at compile time |
| 4 | `#[proc_macro_attribute]` takes two TokenStream args | **True** (attrs + item) |
| 5 | Proc macros can access runtime values | **False** — compile-time only |

**Next:** [Lesson 98 — SIMD](../lesson_98_simd/lesson_98_simd.md) 🦀
