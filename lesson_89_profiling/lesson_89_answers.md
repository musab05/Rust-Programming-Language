# ✅ Lesson 89 — Answers: Profiling (P4)

---

## Section A

### A1
| | Benchmarking | Profiling |
|---|---|---|
| Question | "How fast is X?" | "WHERE is the bottleneck?" |
| Tool | Criterion | flamegraph, perf |
| Use when | Comparing implementations | Finding what to optimize |
| Output | Timing numbers | Call stack visualization |

Use benchmarking first to establish a baseline, then profiling to find hotspots.

### A2
- **X-axis**: proportion of total execution time (wider = more time)
- **Y-axis**: call stack depth (bottom = entry point, top = leaf functions)
- Look for the **widest bars** — those are the bottlenecks
- Colors are random and don't carry meaning

---

## Section B

### A3
```rust
use std::time::Instant;

fn main() {
    let raw: Vec<String> = (0..100_000).rev().map(|i| i.to_string()).collect();
    let data = raw.join("\n");

    let t = Instant::now();
    let parsed: Vec<i32> = data.lines()
        .filter_map(|l| l.parse().ok()).collect();
    println!("[parse]     {:?}", t.elapsed());

    let t = Instant::now();
    let mut sorted = parsed;
    sorted.sort();
    println!("[sort]      {:?}", t.elapsed());

    let t = Instant::now();
    let sum: i64 = sorted.iter().map(|&x| x as i64).sum();
    println!("[aggregate] {:?}", t.elapsed());
    println!("Sum: {sum}");
}
```

### A4
```rust
use std::alloc::{GlobalAlloc, Layout, System};
use std::sync::atomic::{AtomicUsize, Ordering};

struct Counter;
static COUNT: AtomicUsize = AtomicUsize::new(0);
unsafe impl GlobalAlloc for Counter {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 { COUNT.fetch_add(1, Ordering::Relaxed); unsafe { System.alloc(layout) } }
    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) { unsafe { System.dealloc(ptr, layout) } }
}
#[global_allocator] static A: Counter = Counter;

fn main() {
    let c1 = COUNT.load(Ordering::Relaxed);
    let mut v1 = Vec::new();
    for i in 0..1000 { v1.push(i); }
    let allocs_push = COUNT.load(Ordering::Relaxed) - c1;

    let c2 = COUNT.load(Ordering::Relaxed);
    let mut v2 = Vec::with_capacity(1000);
    for i in 0..1000 { v2.push(i); }
    let allocs_cap = COUNT.load(Ordering::Relaxed) - c2;

    println!("Vec::new + push: {allocs_push} allocations");
    println!("with_capacity:   {allocs_cap} allocations");
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Debug builds have no optimization — profiling them is misleading |
| 2 | **True** | Debug symbols let the profiler show function names |
| 3 | **False** | It generates an SVG file (vector, interactive in browser) |
| 4 | **True** | Width represents proportion of total time |
| 5 | **True** | `perf stat` shows hardware performance counters |
| 6 | **True** | Benchmark to measure, profile to find hotspots — both together |

---

## 🏆 Lesson 89 Complete!

**Next up:** [Lesson 90 — cargo-expand & cargo-asm](../lesson_90_cargo_tools/lesson_90_cargo_tools.md) 🦀
