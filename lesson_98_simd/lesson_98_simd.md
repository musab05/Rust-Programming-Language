# 📘 Lesson 98 — SIMD & Intrinsics (P5)

> **Series:** Rust From Zero · Expert Level (Gap Fill)  
> **Roadmap ID:** P5 · Category: ⚡ Performance  
> **Previous:** [Lesson 97 — Proc Macros](../lesson_97_proc_macros/lesson_97_proc_macros.md)  
> **Next:** [Lesson 99 — #![no_std]](../lesson_99_no_std/lesson_99_no_std.md)  
> **Practice:** [Questions](./lesson_98_questions.md) · [Answers](./lesson_98_answers.md)  
> **Practice Task:** SIMD dot product vs scalar comparison

---

## Table of Contents

1. [What Is SIMD?](#1-what-is-simd)
2. [Auto-Vectorization](#2-auto-vectorization)
3. [Portable SIMD (std::simd)](#3-portable-simd-stdsimd)
4. [x86 Intrinsics](#4-x86-intrinsics)
5. [Runtime Feature Detection](#5-runtime-feature-detection)
6. [SIMD Dot Product](#6-simd-dot-product)
7. [Summary Cheat Sheet](#7-summary-cheat-sheet)

---

## 1. What Is SIMD?

**Single Instruction, Multiple Data** — process multiple values in one CPU instruction:

```
Scalar:  a1 + b1 → c1   (1 operation per instruction)

SIMD:    [a1, a2, a3, a4] + [b1, b2, b3, b4] → [c1, c2, c3, c4]
         (4 operations in ONE instruction!)
```

| Instruction Set | Width | Platform |
|---|---|---|
| SSE2 | 128-bit (4× f32) | x86 (baseline) |
| AVX2 | 256-bit (8× f32) | Modern x86 |
| AVX-512 | 512-bit (16× f32) | Server CPUs |
| NEON | 128-bit (4× f32) | ARM (M1/M2) |

---

## 2. Auto-Vectorization

The compiler often vectorizes automatically — check with cargo-show-asm:

```rust
// The compiler auto-vectorizes this to SIMD!
pub fn sum(data: &[f32]) -> f32 {
    data.iter().sum()
}

pub fn add_arrays(a: &[f32], b: &[f32], out: &mut [f32]) {
    for i in 0..a.len().min(b.len()).min(out.len()) {
        out[i] = a[i] + b[i];
    }
}

fn main() {
    let a = vec![1.0f32; 1000];
    let b = vec![2.0f32; 1000];
    let mut c = vec![0.0f32; 1000];
    add_arrays(&a, &b, &mut c);
    println!("c[0] = {}", c[0]); // 3.0
}
```

Tips for helping auto-vectorization:
- Use slices instead of iterators with complex closures
- Avoid early returns and branches in loops
- Use `#[target_feature(enable = "avx2")]` on functions

---

## 3. Portable SIMD (std::simd)

Nightly-only portable SIMD API:

```rust
#![feature(portable_simd)]
use std::simd::f32x4;

fn main() {
    let a = f32x4::from_array([1.0, 2.0, 3.0, 4.0]);
    let b = f32x4::from_array([5.0, 6.0, 7.0, 8.0]);
    let c = a + b;
    println!("{:?}", c.to_array()); // [6.0, 8.0, 10.0, 12.0]

    let d = a * b;
    println!("{:?}", d.to_array()); // [5.0, 12.0, 21.0, 32.0]

    // Horizontal sum
    let sum: f32 = c.to_array().iter().sum();
    println!("Sum: {sum}"); // 34.0
}
```

---

## 4. x86 Intrinsics

Stable API via `std::arch`:

```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

#[target_feature(enable = "sse2")]
unsafe fn add_sse(a: &[f32; 4], b: &[f32; 4]) -> [f32; 4] {
    let va = _mm_loadu_ps(a.as_ptr());
    let vb = _mm_loadu_ps(b.as_ptr());
    let vc = _mm_add_ps(va, vb);
    let mut result = [0.0f32; 4];
    _mm_storeu_ps(result.as_mut_ptr(), vc);
    result
}

fn main() {
    let a = [1.0f32, 2.0, 3.0, 4.0];
    let b = [5.0f32, 6.0, 7.0, 8.0];
    let c = unsafe { add_sse(&a, &b) };
    println!("{:?}", c); // [6.0, 8.0, 10.0, 12.0]
}
```

### Common SSE/AVX intrinsics:

| Intrinsic | Operation |
|---|---|
| `_mm_add_ps` | 4× f32 add |
| `_mm_mul_ps` | 4× f32 multiply |
| `_mm_loadu_ps` | Load 4× f32 (unaligned) |
| `_mm_storeu_ps` | Store 4× f32 (unaligned) |
| `_mm256_add_ps` | 8× f32 add (AVX) |

---

## 5. Runtime Feature Detection

```rust
fn sum_fastest(data: &[f32]) -> f32 {
    #[cfg(target_arch = "x86_64")]
    {
        if is_x86_feature_detected!("avx2") {
            return unsafe { sum_avx2(data) };
        }
        if is_x86_feature_detected!("sse2") {
            return unsafe { sum_sse2(data) };
        }
    }
    // Fallback: scalar
    data.iter().sum()
}

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "avx2")]
unsafe fn sum_avx2(data: &[f32]) -> f32 {
    // AVX2 implementation
    data.iter().sum() // simplified
}

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "sse2")]
unsafe fn sum_sse2(data: &[f32]) -> f32 {
    data.iter().sum() // simplified
}

fn main() {
    let data: Vec<f32> = (0..1000).map(|i| i as f32).collect();
    println!("Sum: {}", sum_fastest(&data));

    #[cfg(target_arch = "x86_64")]
    {
        println!("SSE2:    {}", is_x86_feature_detected!("sse2"));
        println!("AVX2:    {}", is_x86_feature_detected!("avx2"));
        println!("AVX-512: {}", is_x86_feature_detected!("avx512f"));
    }
}
```

---

## 6. SIMD Dot Product

```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

// Scalar version
fn dot_scalar(a: &[f32], b: &[f32]) -> f32 {
    a.iter().zip(b.iter()).map(|(x, y)| x * y).sum()
}

// SSE2 version — 4× faster for large inputs
#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "sse2")]
unsafe fn dot_sse2(a: &[f32], b: &[f32]) -> f32 {
    let len = a.len().min(b.len());
    let chunks = len / 4;
    let mut sum = _mm_setzero_ps();

    for i in 0..chunks {
        let va = _mm_loadu_ps(a.as_ptr().add(i * 4));
        let vb = _mm_loadu_ps(b.as_ptr().add(i * 4));
        let prod = _mm_mul_ps(va, vb);
        sum = _mm_add_ps(sum, prod);
    }

    // Horizontal sum of 4 floats
    let mut result = [0.0f32; 4];
    _mm_storeu_ps(result.as_mut_ptr(), sum);
    let mut total: f32 = result.iter().sum();

    // Handle remaining elements
    for i in (chunks * 4)..len {
        total += a[i] * b[i];
    }
    total
}

fn main() {
    let a: Vec<f32> = (0..1000).map(|i| i as f32).collect();
    let b: Vec<f32> = (0..1000).map(|i| (i * 2) as f32).collect();

    let scalar = dot_scalar(&a, &b);
    println!("Scalar: {scalar}");

    #[cfg(target_arch = "x86_64")]
    {
        let simd = unsafe { dot_sse2(&a, &b) };
        println!("SSE2:   {simd}");
    }
}
```

---

## 7. Summary Cheat Sheet

```
SIMD BASICS
────────────────────────────────────────────────────────────
Process multiple values per instruction
SSE2: 128-bit  AVX2: 256-bit  AVX-512: 512-bit

AUTO-VECTORIZATION
────────────────────────────────────────────────────────────
Compiler does it automatically for simple loops
Verify with: cargo asm --lib my_crate::func

INTRINSICS (stable)
────────────────────────────────────────────────────────────
use std::arch::x86_64::*;
#[target_feature(enable = "sse2")]
unsafe fn f() { _mm_add_ps(a, b); }

FEATURE DETECTION
────────────────────────────────────────────────────────────
is_x86_feature_detected!("avx2")   runtime check
#[target_feature(enable = "avx2")]  compile hint

PORTABLE SIMD (nightly)
────────────────────────────────────────────────────────────
#![feature(portable_simd)]
use std::simd::f32x4;
```

---

## What's Next?

**Lesson 99 — #![no_std]** — Rust without the standard library for bare-metal and embedded.

---

*SIMD: parallelism at the instruction level! 🦀*
