# 🧪 Lesson 98 — Questions: SIMD & Intrinsics (P5)

> **Lesson:** [lesson_98_simd.md](./lesson_98_simd.md)  
> **Answers:** [lesson_98_answers.md](./lesson_98_answers.md)

---

## Section A — Conceptual

### Q1
What does SIMD stand for? Explain how it achieves parallelism differently from multithreading.

### Q2
What is **auto-vectorization**? List three things you can do to help the compiler auto-vectorize your loops.

### Q3
Compare `std::arch` (stable intrinsics) vs `std::simd` (portable SIMD). What are the trade-offs of each approach?

---

## Section B — Write It Yourself

### Q4 — SIMD dot product (Roadmap Practice Task)
Write both a scalar and an SSE2 version of `dot_product(a: &[f32], b: &[f32]) -> f32`. The SSE2 version should:
1. Process 4 floats at a time using `_mm_loadu_ps`, `_mm_mul_ps`, `_mm_add_ps`
2. Handle the remaining elements with scalar fallback
3. Be marked with `#[target_feature(enable = "sse2")]`

### Q5 — Runtime feature detection
Write a `sum_fastest(data: &[f32]) -> f32` function that:
1. Checks for AVX2 at runtime — if available, call an AVX2 path
2. Falls back to SSE2 if AVX2 isn't available
3. Falls back to scalar as a final fallback
4. Uses `is_x86_feature_detected!` for the runtime checks

### Q6 — SIMD array addition
Write an SSE2 function that adds two `&[f32]` slices element-wise and stores the result in a `&mut [f32]` output slice. Process 4 elements at a time.

---

## Section C — True or False?

### Q7
1. SIMD processes multiple data elements with a single CPU instruction.
2. `is_x86_feature_detected!("avx2")` is a compile-time check.
3. SIMD intrinsics in `std::arch` require `unsafe`.
4. `#[target_feature(enable = "avx2")]` enables AVX2 for the entire program.
5. The portable SIMD API (`std::simd`) is stable in current Rust.
6. `_mm_loadu_ps` loads 4 `f32` values from an unaligned pointer.

### Q8
Why does `#[target_feature(enable = "sse2")]` make a function `unsafe` to call? How does runtime feature detection (`is_x86_feature_detected!`) solve this problem?

---

*Parallelism at the instruction level! 🦀*
