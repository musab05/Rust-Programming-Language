# 🧪 Lesson 101 — Questions & ✅ Answers: MMIO & SVD2Rust (EM3)

> **Lesson:** [lesson_101_mmio.md](./lesson_101_mmio.md)

---

## Q1 — Why must register access use volatile operations?
**A:** Without volatile, the compiler may optimize away reads (caching) or writes (dead store elimination). Hardware registers can change independently, and writes have side effects — volatile prevents these optimizations.

## Q2 — What does SVD2Rust do?
**A:** Converts an SVD (XML) file describing a chip's registers into a type-safe Rust PAC crate. Instead of magic hex addresses and bit manipulation, you get named registers with methods like `.modify(|r, w| w.moder5().output())`.

## Q3 — True or False?
| # | Statement | Answer |
|---|-----------|--------|
| 1 | Memory-mapped registers are at fixed addresses | **True** |
| 2 | `read_volatile` may be optimized away by the compiler | **False** — that's the whole point of volatile |
| 3 | SVD files are provided by chip vendors | **True** |
| 4 | PACs implement embedded-hal traits | **False** — HALs do; PACs provide register access |
| 5 | `.modify()` performs a read-modify-write operation | **True** |

---

## 🎉🏆 ROADMAP COMPLETE! ALL 101 LESSONS DONE! 🦀

**From Hello World to bare-metal registers — congratulations!**
