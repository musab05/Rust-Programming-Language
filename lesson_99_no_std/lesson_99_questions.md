# 🧪 Lesson 99 — Questions & ✅ Answers: #![no_std] (EM1)

> **Lesson:** [lesson_99_no_std.md](./lesson_99_no_std.md)

---

## Q1 — What's the difference between `core`, `alloc`, and `std`?
**A:** `core` = no OS, no heap (primitives, Option, Result). `alloc` = adds heap (Vec, String, Box). `std` = full OS support (files, network, threads).

## Q2 — no_std library (Roadmap Practice Task)
Write a `#![no_std]` ring buffer. See lesson for full implementation.

## Q3 — True or False?
| # | Statement | Answer |
|---|-----------|--------|
| 1 | `#![no_std]` means you can't use `Option` or `Result` | **False** — they're in `core` |
| 2 | `#![no_std]` libraries need a `#[panic_handler]` | **False** — only binaries do |
| 3 | `Vec` requires `alloc`, not just `core` | **True** |
| 4 | `HashMap` is available in `core` | **False** — it's only in `std` |
| 5 | Tests (`#[cfg(test)]`) can still use `std` | **True** |

**Next:** [Lesson 100 — Embedded HAL](../lesson_100_embedded_hal/lesson_100_embedded_hal.md) 🦀
