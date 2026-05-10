# 🧪 Lesson 100 — Questions & ✅ Answers: Embedded HAL (EM2)

> **Lesson:** [lesson_100_embedded_hal.md](./lesson_100_embedded_hal.md)

---

## Q1 — Why use traits instead of direct register access?
**A:** Portability. A driver written against `OutputPin` works on STM32, nRF52, RP2040, or any chip with an embedded-hal implementation.

## Q2 — LED driver (Roadmap Practice Task)
Write a portable `Led<P: OutputPin>` with `on()`, `off()`, `toggle()`. See lesson for implementation.

## Q3 — True or False?
| # | Statement | Answer |
|---|-----------|--------|
| 1 | embedded-hal defines traits, not implementations | **True** |
| 2 | PAC = Peripheral Access Crate (register-level) | **True** |
| 3 | Drivers using embedded-hal traits are chip-specific | **False** — they're portable |
| 4 | `DelayNs` provides `delay_ms()` and `delay_us()` | **True** |

**Next:** [Lesson 101 — MMIO & SVD2Rust](../lesson_101_mmio/lesson_101_mmio.md) — the FINAL lesson! 🎉🦀
