# ✅ Lesson 88 — Answers: Benchmarking (P3)

---

## Section A

### A1
`black_box()` prevents the compiler's optimizer from eliminating dead code. Without it, the compiler may realize the benchmark result is unused and skip the computation entirely, giving misleadingly fast results.

### A2
`iter_batched` creates fresh input for each iteration via a setup function. Use it when your benchmark **mutates** the input (e.g., sorting a vec in-place). With plain `iter`, the same input would be reused, and sorting an already-sorted vec is much faster.

---

## Section B

### A3
```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};

fn insertion_sort(data: &mut Vec<i32>) {
    for i in 1..data.len() {
        let key = data[i];
        let mut j = i;
        while j > 0 && data[j - 1] > key {
            data[j] = data[j - 1];
            j -= 1;
        }
        data[j] = key;
    }
}

fn bench_sorting(c: &mut Criterion) {
    let mut group = c.benchmark_group("sorting");
    for size in [1_000, 5_000, 10_000] {
        group.bench_with_input(BenchmarkId::new("std_sort", size), &size, |b, &sz| {
            b.iter_batched(
                || (0..sz).rev().collect::<Vec<i32>>(),
                |mut v| { v.sort(); black_box(v); },
                criterion::BatchSize::SmallInput,
            );
        });
        group.bench_with_input(BenchmarkId::new("sort_unstable", size), &size, |b, &sz| {
            b.iter_batched(
                || (0..sz).rev().collect::<Vec<i32>>(),
                |mut v| { v.sort_unstable(); black_box(v); },
                criterion::BatchSize::SmallInput,
            );
        });
        if size <= 5_000 {
            group.bench_with_input(BenchmarkId::new("insertion", size), &size, |b, &sz| {
                b.iter_batched(
                    || (0..sz).rev().collect::<Vec<i32>>(),
                    |mut v| { insertion_sort(&mut v); black_box(v); },
                    criterion::BatchSize::SmallInput,
                );
            });
        }
    }
    group.finish();
}

criterion_group!(benches, bench_sorting);
criterion_main!(benches);
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Criterion reports mean, median, confidence intervals, outliers |
| 2 | **True** | `cargo bench` looks in `benches/` for benchmark files |
| 3 | **True** | `black_box` is an optimization barrier |
| 4 | **True** | Criterion saves baselines and compares across runs |
| 5 | **True** | HTML reports with charts are auto-generated |
| 6 | **False** | `harness = false` means it DISABLES the built-in harness — Criterion provides its own |

---

## 🏆 Lesson 88 Complete!

**Next up:** [Lesson 89 — Profiling](../lesson_89_profiling/lesson_89_profiling.md) 🦀
