# ✅ Lesson 98 — Answers: SIMD & Intrinsics (P5)

---

## Section A

### A1
**SIMD** = Single Instruction, Multiple Data. It processes multiple values (e.g., 4 floats) in **one CPU instruction** using wide registers (128/256/512-bit).

**Difference from multithreading:**
- **SIMD** = one core, one instruction, multiple data lanes. No thread overhead, no synchronization.
- **Multithreading** = multiple cores, each running independent instruction streams. Requires synchronization (Mutex, channels).
- SIMD is **data parallelism within a single thread**; multithreading is **task parallelism across cores**.

### A2
**Auto-vectorization** is when the compiler automatically converts scalar loops into SIMD instructions without you writing intrinsics.

Three ways to help:
1. **Use simple loop bodies** — avoid branches, early returns, and complex control flow inside loops
2. **Use `#[target_feature(enable = "avx2")]`** on the function to let the compiler use wider instructions
3. **Use slices with known alignment** and avoid aliasing (the compiler needs to prove independence of memory regions)

Bonus: Compile with `-C target-cpu=native` to enable all features your CPU supports.

### A3
| Feature | `std::arch` (stable) | `std::simd` (portable) |
|---------|---------------------|----------------------|
| Stability | ✅ Stable | ❌ Nightly only |
| Portability | ❌ Platform-specific (x86, ARM, etc.) | ✅ Cross-platform |
| Control | Full — exact intrinsics | Abstract — compiler chooses instructions |
| Safety | `unsafe` required | Safe API |
| Performance | Maximum (hand-tuned) | Good (compiler decides) |
| Complexity | High — must handle each architecture | Low — write once |

---

## Section B

### A4
```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

// Scalar version
fn dot_scalar(a: &[f32], b: &[f32]) -> f32 {
    a.iter().zip(b.iter()).map(|(x, y)| x * y).sum()
}

// SSE2 version — processes 4 floats at a time
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

    // Horizontal sum: extract all 4 floats and add
    let mut result = [0.0f32; 4];
    _mm_storeu_ps(result.as_mut_ptr(), sum);
    let mut total: f32 = result.iter().sum();

    // Handle remaining elements (0–3)
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
        if is_x86_feature_detected!("sse2") {
            let simd = unsafe { dot_sse2(&a, &b) };
            println!("SSE2:   {simd}");
            assert!((scalar - simd).abs() < 1.0, "Results differ!");
        }
    }
}
```

### A5
```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

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
    // Scalar fallback
    data.iter().sum()
}

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "avx2")]
unsafe fn sum_avx2(data: &[f32]) -> f32 {
    let chunks = data.len() / 8;
    let mut acc = _mm256_setzero_ps();
    for i in 0..chunks {
        let v = _mm256_loadu_ps(data.as_ptr().add(i * 8));
        acc = _mm256_add_ps(acc, v);
    }
    // Horizontal sum: 8 floats → 1
    let mut buf = [0.0f32; 8];
    _mm256_storeu_ps(buf.as_mut_ptr(), acc);
    let mut total: f32 = buf.iter().sum();
    for i in (chunks * 8)..data.len() {
        total += data[i];
    }
    total
}

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "sse2")]
unsafe fn sum_sse2(data: &[f32]) -> f32 {
    let chunks = data.len() / 4;
    let mut acc = _mm_setzero_ps();
    for i in 0..chunks {
        let v = _mm_loadu_ps(data.as_ptr().add(i * 4));
        acc = _mm_add_ps(acc, v);
    }
    let mut buf = [0.0f32; 4];
    _mm_storeu_ps(buf.as_mut_ptr(), acc);
    let mut total: f32 = buf.iter().sum();
    for i in (chunks * 4)..data.len() {
        total += data[i];
    }
    total
}

fn main() {
    let data: Vec<f32> = (1..=1000).map(|i| i as f32).collect();
    let result = sum_fastest(&data);
    println!("Sum: {result}");  // 500500.0
}
```

### A6
```rust
#[cfg(target_arch = "x86_64")]
use std::arch::x86_64::*;

#[cfg(target_arch = "x86_64")]
#[target_feature(enable = "sse2")]
unsafe fn add_arrays_sse2(a: &[f32], b: &[f32], out: &mut [f32]) {
    let len = a.len().min(b.len()).min(out.len());
    let chunks = len / 4;

    for i in 0..chunks {
        let va = _mm_loadu_ps(a.as_ptr().add(i * 4));
        let vb = _mm_loadu_ps(b.as_ptr().add(i * 4));
        let vc = _mm_add_ps(va, vb);
        _mm_storeu_ps(out.as_mut_ptr().add(i * 4), vc);
    }

    // Handle remaining elements
    for i in (chunks * 4)..len {
        out[i] = a[i] + b[i];
    }
}

fn main() {
    let a = vec![1.0f32, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0];
    let b = vec![10.0f32, 20.0, 30.0, 40.0, 50.0, 60.0, 70.0];
    let mut c = vec![0.0f32; 7];

    #[cfg(target_arch = "x86_64")]
    {
        if is_x86_feature_detected!("sse2") {
            unsafe { add_arrays_sse2(&a, &b, &mut c); }
            println!("{:?}", c);
            // [11.0, 22.0, 33.0, 44.0, 55.0, 66.0, 77.0]
        }
    }
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | That's the definition of SIMD — one instruction, many data |
| 2 | **False** | It's a **runtime** check — queries CPUID at execution time |
| 3 | **True** | Intrinsics can cause UB on CPUs that lack the feature |
| 4 | **False** | It only enables AVX2 for that **one function** |
| 5 | **False** | `std::simd` is nightly-only (unstable) |
| 6 | **True** | The `u` in `loadu` means "unaligned" — works with any pointer |

### A8
`#[target_feature(enable = "sse2")]` makes a function unsafe to call because calling it on a CPU that **lacks** that feature is **undefined behavior** — the CPU will fault on illegal instructions.

Runtime feature detection solves this by:
1. `is_x86_feature_detected!("sse2")` queries the CPU at runtime (via `cpuid`)
2. Only if the feature exists do you call the `unsafe` function
3. You provide a safe scalar fallback for CPUs without the feature
4. The outer dispatch function can be **safe** because it guarantees the invariant

Pattern:
```rust
fn safe_dispatch(data: &[f32]) -> f32 {
    if is_x86_feature_detected!("sse2") {
        unsafe { sse2_impl(data) }  // safe: we just checked
    } else {
        scalar_impl(data)
    }
}
```

---

## 🏆 Lesson 98 Complete!

**Next up:** [Lesson 99 — #![no_std]](../lesson_99_no_std/lesson_99_no_std.md) 🦀
