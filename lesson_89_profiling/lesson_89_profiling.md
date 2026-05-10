# 📘 Lesson 89 — Profiling (P4)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** P4 · Category: ⚡ Performance  
> **Previous:** [Lesson 88 — Benchmarking with Criterion](../lesson_88_benchmarking/lesson_88_benchmarking.md)  
> **Next:** [Lesson 90 — cargo-expand & cargo-asm](../lesson_90_cargo_tools/lesson_90_cargo_tools.md)  
> **Practice:** [Questions](./lesson_89_questions.md) · [Answers](./lesson_89_answers.md)  
> **Practice Task:** Profile a slow program and find the bottleneck

---

## Table of Contents

1. [Benchmarking vs Profiling](#1-benchmarking-vs-profiling)
2. [cargo flamegraph](#2-cargo-flamegraph)
3. [Perf (Linux)](#3-perf-linux)
4. [Reading a Flamegraph](#4-reading-a-flamegraph)
5. [Time-Based Profiling](#5-time-based-profiling)
6. [Memory Profiling](#6-memory-profiling)
7. [Profiling Tips](#7-profiling-tips)
8. [Platform-Specific Tools](#8-platform-specific-tools)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Benchmarking vs Profiling

| | Benchmarking | Profiling |
|---|---|---|
| Question | "How fast is it?" | "Where is it slow?" |
| Tool | Criterion | flamegraph, perf, VTune |
| Output | Time measurements | Call stacks, hotspots |
| When | Compare algorithms | Find bottlenecks |

---

## 2. cargo flamegraph

The easiest way to profile Rust programs:

### Install:

```bash
# Install flamegraph tool
cargo install flamegraph

# Linux: needs perf
sudo apt install linux-tools-common linux-tools-generic

# macOS: needs dtrace (built-in)

# Windows: needs ETW or WSL
```

### Setup Cargo.toml for debug symbols in release:

```toml
[profile.release]
debug = true    # keep debug symbols for profiling
```

### Generate a flamegraph:

```bash
# Build and profile in release mode
cargo flamegraph --bin my_app

# With specific arguments
cargo flamegraph --bin my_app -- arg1 arg2

# Output: flamegraph.svg (open in browser)
```

---

## 3. Perf (Linux)

Low-level CPU profiling:

```bash
# Build with debug symbols
cargo build --release

# Record profile
perf record -g ./target/release/my_app

# View results
perf report

# Or generate flamegraph from perf data
perf script | stackcollapse-perf.pl | flamegraph.pl > perf.svg
```

### Common perf commands:

```bash
# CPU cycles
perf stat ./target/release/my_app

# Cache misses
perf stat -e cache-misses,cache-references ./target/release/my_app

# Branch mispredictions
perf stat -e branch-misses,branch-instructions ./target/release/my_app
```

---

## 4. Reading a Flamegraph

```
┌───────────────────────────────────────────────────┐
│                    main                            │  ← bottom: entry point
├────────────────────────┬──────────────────────────┤
│     process_data       │      load_file           │
├──────────┬─────────────┤                          │
│ parse    │  transform  │                          │
│          ├─────┬───────┤                          │
│          │sort │filter │                          │
└──────────┴─────┴───────┴──────────────────────────┘
```

**How to read:**
- **X-axis** = proportion of time (wider = more time)
- **Y-axis** = call stack depth (bottom = main, top = leaf functions)
- **Widest bars** = biggest bottlenecks
- **Color** = random (not meaningful)

### What to look for:

| Pattern | Meaning | Action |
|---|---|---|
| Wide flat top | Leaf function taking lots of time | Optimize this function |
| Wide plateau | Function and all children are slow | Dig into children |
| Narrow towers | Deep recursion | Consider iteration |
| `alloc::` bars | Heap allocation overhead | Reduce allocations |

---

## 5. Time-Based Profiling

Simple manual profiling with timing:

```rust
use std::time::Instant;

fn timed<F, R>(label: &str, f: F) -> R
where F: FnOnce() -> R {
    let start = Instant::now();
    let result = f();
    let elapsed = start.elapsed();
    println!("[{label}] {elapsed:.2?}");
    result
}

fn slow_parse(data: &str) -> Vec<i32> {
    data.lines()
        .filter_map(|line| line.trim().parse::<i32>().ok())
        .collect()
}

fn slow_sort(mut data: Vec<i32>) -> Vec<i32> {
    data.sort();
    data
}

fn slow_aggregate(data: &[i32]) -> i64 {
    data.iter().map(|&x| x as i64).sum()
}

fn main() {
    let raw = (0..100_000)
        .map(|i| format!("{}", 100_000 - i))
        .collect::<Vec<_>>()
        .join("\n");

    let parsed = timed("parse", || slow_parse(&raw));
    let sorted = timed("sort", || slow_sort(parsed));
    let total = timed("aggregate", || slow_aggregate(&sorted));
    println!("Sum: {total}");
}
```

### Using the `tracing` crate for instrumentation:

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = "0.3"
```

```rust
use tracing::{info, instrument};

#[instrument]
fn process_item(id: u32) -> u32 {
    info!("Processing item {id}");
    std::thread::sleep(std::time::Duration::from_millis(1));
    id * 2
}

fn main() {
    tracing_subscriber::fmt::init();
    for i in 0..5 {
        let result = process_item(i);
        info!("Result: {result}");
    }
}
```

---

## 6. Memory Profiling

### Tracking allocations with a custom allocator:

```rust
use std::alloc::{GlobalAlloc, Layout, System};
use std::sync::atomic::{AtomicUsize, Ordering};

struct CountingAlloc;

static ALLOC_COUNT: AtomicUsize = AtomicUsize::new(0);
static ALLOC_BYTES: AtomicUsize = AtomicUsize::new(0);

unsafe impl GlobalAlloc for CountingAlloc {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
        ALLOC_COUNT.fetch_add(1, Ordering::Relaxed);
        ALLOC_BYTES.fetch_add(layout.size(), Ordering::Relaxed);
        unsafe { System.alloc(layout) }
    }
    unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
        unsafe { System.dealloc(ptr, layout) }
    }
}

#[global_allocator]
static GLOBAL: CountingAlloc = CountingAlloc;

fn main() {
    let before_count = ALLOC_COUNT.load(Ordering::Relaxed);
    let before_bytes = ALLOC_BYTES.load(Ordering::Relaxed);

    // Code to measure
    let mut v = Vec::new();
    for i in 0..1000 {
        v.push(format!("item-{i}"));
    }

    let allocs = ALLOC_COUNT.load(Ordering::Relaxed) - before_count;
    let bytes = ALLOC_BYTES.load(Ordering::Relaxed) - before_bytes;
    println!("Allocations: {allocs}");
    println!("Bytes allocated: {bytes}");
}
```

### External tools:

```bash
# Linux: heaptrack
heaptrack ./target/release/my_app
heaptrack_gui heaptrack.my_app.*.gz

# Valgrind (Linux/macOS)
valgrind --tool=massif ./target/release/my_app
ms_print massif.out.*

# DHAT (Valgrind)
valgrind --tool=dhat ./target/release/my_app
```

---

## 7. Profiling Tips

### ✅ DO:

```bash
# Always profile in release mode with debug symbols
[profile.release]
debug = true

# Profile realistic workloads (not tiny tests)
# Profile the specific slow path
# Compare before/after with flamegraphs
```

### ❌ AVOID:

```bash
# Profiling debug builds (too slow, misleading results)
cargo run  # ← debug, not representative

# Optimizing without measuring first
# Profiling with unrealistic input sizes
```

### Common bottleneck patterns:

| Pattern | Symptom | Fix |
|---|---|---|
| Excessive allocation | `alloc::` in flamegraph | Pre-allocate, reuse buffers |
| Unnecessary cloning | `clone()` calls | Use references, Cow |
| Lock contention | Time in `Mutex::lock` | Reduce critical section, use RwLock |
| Cache misses | Slow despite low CPU | Improve data locality |
| Redundant computation | Same values recalculated | Memoize, cache results |

---

## 8. Platform-Specific Tools

| Platform | CPU Profiling | Memory Profiling | GUI |
|---|---|---|---|
| **Linux** | perf, flamegraph | heaptrack, valgrind | perf report, hotspot |
| **macOS** | Instruments, dtrace | Instruments | Instruments.app |
| **Windows** | ETW, cargo-flamegraph | VS Diagnostics | Visual Studio, WPA |
| **Cross-platform** | cargo flamegraph | counting allocator | — |

---

## 9. Summary Cheat Sheet

```
TOOLS
────────────────────────────────────────────────────────────
cargo flamegraph         CPU profiling → SVG visualization
perf                     Linux CPU counters and stacks
heaptrack                Linux heap allocation profiling
Instruments              macOS profiling suite

SETUP
────────────────────────────────────────────────────────────
[profile.release]
debug = true             debug symbols for profiling

WORKFLOW
────────────────────────────────────────────────────────────
1. Benchmark (Criterion)  → "how fast?"
2. Flamegraph             → "where is it slow?"
3. Optimize the hotspot
4. Benchmark again        → "did it improve?"

FLAMEGRAPH READING
────────────────────────────────────────────────────────────
Width = time spent        wider = more time
Height = call depth       bottom = main
Look for wide flat bars   = your bottleneck

MANUAL TIMING
────────────────────────────────────────────────────────────
let t = Instant::now();
do_work();
println!("{:?}", t.elapsed());
```

---

## What's Next?

**Lesson 90 — cargo-expand & cargo-asm** — Inspect macro expansion and generated assembly.

## Further Reading
- [cargo-flamegraph](https://github.com/flamegraph-rs/flamegraph)
- [The Rust Performance Book](https://nnethercote.github.io/perf-book/)
- [Profiling Rust Applications](https://nnethercote.github.io/perf-book/profiling.html)

---

*Profiling: find the bottleneck, then fix it! 🦀*
