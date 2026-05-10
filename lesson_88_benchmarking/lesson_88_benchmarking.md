# 📘 Lesson 88 — Benchmarking with Criterion (P3)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** P3 · Category: ⚡ Performance  
> **Previous:** [Lesson 87 — PhantomData](../lesson_87_phantomdata/lesson_87_phantomdata.md)  
> **Next:** [Lesson 89 — Profiling](../lesson_89_profiling/lesson_89_profiling.md)  
> **Practice:** [Questions](./lesson_88_questions.md) · [Answers](./lesson_88_answers.md)  
> **Practice Task:** Benchmark three sorting strategies on Vec\<i32\>

---

## Table of Contents

1. [Why Benchmark?](#1-why-benchmark)
2. [Setup](#2-setup)
3. [Your First Benchmark](#3-your-first-benchmark)
4. [Benchmarking Multiple Functions](#4-benchmarking-multiple-functions)
5. [Parameterized Benchmarks](#5-parameterized-benchmarks)
6. [Benchmark Groups](#6-benchmark-groups)
7. [Avoiding Dead Code Elimination](#7-avoiding-dead-code-elimination)
8. [Reading Results](#8-reading-results)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Why Benchmark?

Intuition about performance is often wrong. Measure, don't guess:

```
"Premature optimization is the root of all evil" — Knuth
"But measuring is always appropriate" — Everyone
```

Criterion provides:
- **Statistical analysis** (mean, median, std deviation)
- **Comparison** across runs (detects regressions)
- **HTML reports** with charts
- **Prevents dead code elimination** (optimizer pitfall)

---

## 2. Setup

```toml
# Cargo.toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "my_benchmarks"
harness = false
```

Create `benches/my_benchmarks.rs`:

```rust
use criterion::{criterion_group, criterion_main};
// ... benchmark functions ...
criterion_group!(benches, bench_function);
criterion_main!(benches);
```

Run: `cargo bench`

---

## 3. Your First Benchmark

```rust
// benches/my_benchmarks.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn fibonacci(n: u64) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn bench_fibonacci(c: &mut Criterion) {
    c.bench_function("fibonacci 20", |b| {
        b.iter(|| fibonacci(black_box(20)))
    });
}

criterion_group!(benches, bench_fibonacci);
criterion_main!(benches);
```

```bash
$ cargo bench

fibonacci 20            time:   [25.112 µs 25.234 µs 25.367 µs]
                        change: [-0.3412% +0.2145% +0.7890%] (p = 0.52 > 0.05)
                        No change in performance detected.
```

---

## 4. Benchmarking Multiple Functions

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion};

fn sum_loop(data: &[i32]) -> i64 {
    let mut sum: i64 = 0;
    for &x in data { sum += x as i64; }
    sum
}

fn sum_iter(data: &[i32]) -> i64 {
    data.iter().map(|&x| x as i64).sum()
}

fn sum_fold(data: &[i32]) -> i64 {
    data.iter().fold(0i64, |acc, &x| acc + x as i64)
}

fn bench_sums(c: &mut Criterion) {
    let data: Vec<i32> = (0..10_000).collect();

    c.bench_function("sum_loop", |b| {
        b.iter(|| sum_loop(black_box(&data)))
    });

    c.bench_function("sum_iter", |b| {
        b.iter(|| sum_iter(black_box(&data)))
    });

    c.bench_function("sum_fold", |b| {
        b.iter(|| sum_fold(black_box(&data)))
    });
}

criterion_group!(benches, bench_sums);
criterion_main!(benches);
```

---

## 5. Parameterized Benchmarks

Test across different input sizes:

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};

fn sort_vec(data: &mut Vec<i32>) {
    data.sort();
}

fn bench_sort_sizes(c: &mut Criterion) {
    let sizes = [100, 1_000, 10_000, 100_000];

    let mut group = c.benchmark_group("sort");
    for &size in &sizes {
        group.bench_with_input(
            BenchmarkId::from_parameter(size),
            &size,
            |b, &size| {
                b.iter_batched(
                    || (0..size).rev().collect::<Vec<i32>>(),  // setup
                    |mut data| sort_vec(black_box(&mut data)),  // routine
                    criterion::BatchSize::SmallInput,
                );
            },
        );
    }
    group.finish();
}

criterion_group!(benches, bench_sort_sizes);
criterion_main!(benches);
```

---

## 6. Benchmark Groups

Compare different algorithms on the same input:

```rust
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};

fn linear_search(data: &[i32], target: i32) -> Option<usize> {
    data.iter().position(|&x| x == target)
}

fn binary_search_wrapper(data: &[i32], target: i32) -> Option<usize> {
    data.binary_search(&target).ok()
}

fn bench_search(c: &mut Criterion) {
    let mut group = c.benchmark_group("search");

    for size in [1_000, 10_000, 100_000] {
        let data: Vec<i32> = (0..size as i32).collect();
        let target = size as i32 - 1;  // worst case for linear

        group.bench_with_input(
            BenchmarkId::new("linear", size),
            &size,
            |b, _| b.iter(|| linear_search(black_box(&data), black_box(target))),
        );

        group.bench_with_input(
            BenchmarkId::new("binary", size),
            &size,
            |b, _| b.iter(|| binary_search_wrapper(black_box(&data), black_box(target))),
        );
    }
    group.finish();
}

criterion_group!(benches, bench_search);
criterion_main!(benches);
```

---

## 7. Avoiding Dead Code Elimination

The optimizer may remove code whose result isn't used. `black_box` prevents this:

```rust
use criterion::black_box;

// ❌ BAD — optimizer may skip the entire computation
fn bad_bench(b: &mut criterion::Bencher) {
    b.iter(|| {
        let result = expensive_function();
        // result is unused — optimizer removes it!
    });
}

// ✅ GOOD — black_box prevents optimization
fn good_bench(b: &mut criterion::Bencher) {
    b.iter(|| {
        black_box(expensive_function())
    });
}

// ✅ ALSO GOOD — black_box on inputs
fn also_good(b: &mut criterion::Bencher) {
    let data = vec![1, 2, 3];
    b.iter(|| {
        process(black_box(&data))
    });
}

fn expensive_function() -> u64 { (0..1000).sum() }
fn process(_data: &[i32]) -> i32 { 42 }
```

---

## 8. Reading Results

```
sorting/std_sort        time:   [245.12 µs 246.78 µs 248.50 µs]
                        ▲▲▲          ▲▲▲          ▲▲▲
                      lower       estimate       upper
                      bound                      bound

Found 2 outliers among 100 measurements (2.00%)
  1 (1.00%) high mild
  1 (1.00%) high severe
```

- **Lower/Upper bound**: 95% confidence interval
- **Estimate**: Best statistical estimate of true mean
- **change**: Compared to previous run (regression detection)

HTML reports in `target/criterion/report/index.html`.

---

## 9. Summary Cheat Sheet

```
SETUP
────────────────────────────────────────────────────────────
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "my_bench"
harness = false

BASIC BENCHMARK
────────────────────────────────────────────────────────────
c.bench_function("name", |b| {
    b.iter(|| black_box(my_function(black_box(input))))
});

PARAMETERIZED
────────────────────────────────────────────────────────────
group.bench_with_input(BenchmarkId::new("name", param), ...);

BATCH (for mutable inputs)
────────────────────────────────────────────────────────────
b.iter_batched(setup_fn, routine_fn, BatchSize::SmallInput);

RUNNING
────────────────────────────────────────────────────────────
cargo bench                 run all benchmarks
cargo bench -- sort         filter by name
target/criterion/report/    HTML reports

KEY RULE
────────────────────────────────────────────────────────────
Always use black_box() to prevent dead code elimination!
```

---

## What's Next?

**Lesson 89 — Profiling** — Find bottlenecks with cargo flamegraph, perf, and heaptrack.

## Further Reading
- [Criterion.rs User Guide](https://bheisler.github.io/criterion.rs/book/)
- [Criterion docs](https://docs.rs/criterion/)

---

*Benchmarking: measure twice, optimize once! 🦀*
