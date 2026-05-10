# 🧪 Lesson 98 — Questions & ✅ Answers: SIMD (P5)

> **Lesson:** [lesson_98_simd.md](./lesson_98_simd.md)

---

## Q1 — What does SIMD stand for and why is it faster?
**A:** Single Instruction, Multiple Data. One CPU instruction operates on 4/8/16 values simultaneously instead of one.

## Q2 — SIMD dot product (Roadmap Practice Task)
Write scalar and SSE2 dot product. See lesson for full implementation.

## Q3 — True or False?
| # | Statement | Answer |
|---|-----------|--------|
| 1 | The compiler can auto-vectorize simple loops | **True** |
| 2 | `is_x86_feature_detected!` is a runtime check | **True** |
| 3 | SIMD intrinsics are safe to call without `unsafe` | **False** |
| 4 | `#[target_feature(enable = "avx2")]` enables AVX2 for one function | **True** |
| 5 | Portable SIMD (`std::simd`) is stable | **False** — nightly only |

**Next:** [Lesson 99 — #![no_std]](../lesson_99_no_std/lesson_99_no_std.md) 🦀
