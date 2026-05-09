# 📘 Lesson 74 — Stack vs Heap (P1)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** P1 · Category: ⚡ Performance  
> **Previous:** [Lesson 73 — Newtype Pattern](../lesson_73_newtype/lesson_73_newtype.md)  
> **Next:** [Lesson 75 — Zero-Cost Abstractions](../lesson_75_zero_cost/lesson_75_zero_cost.md)  
> **Practice:** [Questions](./lesson_74_questions.md) · [Answers](./lesson_74_answers.md)  
> **Practice Task:** Profile two versions of a hot loop; measure allocations

---

## Table of Contents

1. [Memory Layout Overview](#1-memory-layout-overview)
2. [The Stack](#2-the-stack)
3. [The Heap](#3-the-heap)
4. [What Lives Where?](#4-what-lives-where)
5. [Cost of Allocation](#5-cost-of-allocation)
6. [Box — Moving to the Heap](#6-box--moving-to-the-heap)
7. [Measuring Allocations](#7-measuring-allocations)
8. [Performance Guidelines](#8-performance-guidelines)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Memory Layout Overview

```
HIGH ADDRESS
┌──────────────────┐
│  Stack            │  ← grows downward
│  (local vars,     │
│   function args)  │
├──────────────────┤
│  ...free space... │
├──────────────────┤
│  Heap             │  ← grows upward
│  (Box, Vec, String│
│   dynamic data)   │
├──────────────────┤
│  Static/Global    │  ← constants, string literals
├──────────────────┤
│  Code (Text)      │  ← compiled instructions
LOW ADDRESS
```

---

## 2. The Stack

**Fast, automatic, fixed-size** memory:

```rust
fn factorial(n: u64) -> u64 {
    // Each call pushes a stack frame: n, return address
    if n <= 1 { 1 } else { n * factorial(n - 1) }
}

fn main() {
    let x: i32 = 42;          // stack: 4 bytes
    let y: f64 = 3.14;        // stack: 8 bytes
    let arr: [i32; 5] = [1, 2, 3, 4, 5];  // stack: 20 bytes
    let tup: (i32, bool) = (10, true);     // stack: 8 bytes

    println!("x = {x}, y = {y}");
    println!("arr = {:?}", arr);
    println!("10! = {}", factorial(10));
}
```

### Stack properties:

| Property | Detail |
|---|---|
| Speed | Very fast (just move stack pointer) |
| Size | Limited (default ~8MB on Linux, ~1MB on Windows) |
| Allocation | Automatic (push/pop) |
| Cleanup | Automatic (when function returns) |
| Layout | Known at compile time |
| Thread-safe | Each thread has its own stack |

### Stack overflow:

```rust
fn infinite_recursion() {
    infinite_recursion();  // 💥 stack overflow!
}

// fn main() { infinite_recursion(); }
// thread 'main' has overflowed its stack
```

---

## 3. The Heap

**Flexible, dynamic, slower** memory:

```rust
fn main() {
    // Heap-allocated data
    let s = String::from("hello");       // heap: 5 bytes + metadata on stack
    let v = vec![1, 2, 3, 4, 5];        // heap: 20 bytes + metadata on stack
    let b = Box::new(42);               // heap: 4 bytes + pointer on stack

    // Stack metadata for String:
    // ptr: *mut u8    (8 bytes) → points to heap data
    // len: usize      (8 bytes) → current length
    // cap: usize      (8 bytes) → allocated capacity
    // Total on stack: 24 bytes
    // Actual string data: on heap

    println!("String size on stack: {}", std::mem::size_of::<String>());  // 24
    println!("Vec size on stack: {}", std::mem::size_of::<Vec<i32>>());   // 24
    println!("Box size on stack: {}", std::mem::size_of::<Box<i32>>());   // 8
    println!("i32 size on stack: {}", std::mem::size_of::<i32>());       // 4
}
```

### Heap properties:

| Property | Detail |
|---|---|
| Speed | Slower (allocator, fragmentation) |
| Size | Large (limited by OS/RAM) |
| Allocation | Manual request (`Box::new`, `Vec::push`) |
| Cleanup | Automatic via `Drop` trait |
| Layout | Dynamic (can grow/shrink) |
| Sharing | Can be shared across scopes (via references/Rc/Arc) |

---

## 4. What Lives Where?

```rust
fn main() {
    // STACK — known size at compile time
    let a: i32 = 42;                    // stack
    let b: f64 = 3.14;                  // stack
    let c: bool = true;                 // stack
    let d: char = 'x';                  // stack
    let e: (i32, i32) = (1, 2);         // stack
    let f: [i32; 3] = [1, 2, 3];       // stack (fixed-size array)

    // HEAP — dynamic size
    let g: String = String::from("hi"); // metadata on stack, chars on heap
    let h: Vec<i32> = vec![1, 2, 3];   // metadata on stack, elements on heap
    let i: Box<i32> = Box::new(99);    // pointer on stack, i32 on heap

    // STACK (references are pointers on the stack)
    let j: &str = "hello";              // pointer on stack → string in static memory
    let k: &[i32] = &f;                // pointer on stack → array on stack

    println!("Sizes:");
    println!("  i32:    {} bytes", std::mem::size_of_val(&a));
    println!("  String: {} bytes (stack) + heap data", std::mem::size_of_val(&g));
    println!("  Vec:    {} bytes (stack) + heap data", std::mem::size_of_val(&h));
    println!("  Box:    {} bytes (stack) + heap data", std::mem::size_of_val(&i));
    println!("  &str:   {} bytes (fat pointer)", std::mem::size_of_val(&j));
}
```

---

## 5. Cost of Allocation

```rust
use std::time::Instant;

fn stack_version() -> i64 {
    let mut sum: i64 = 0;
    for i in 0..1_000_000 {
        let x: i64 = i;  // stack — instant
        sum += x;
    }
    sum
}

fn heap_version() -> i64 {
    let mut sum: i64 = 0;
    for i in 0..1_000_000 {
        let x = Box::new(i as i64);  // heap — allocator call each time!
        sum += *x;
    }
    sum
}

fn main() {
    let start = Instant::now();
    let s1 = stack_version();
    let stack_time = start.elapsed();

    let start = Instant::now();
    let s2 = heap_version();
    let heap_time = start.elapsed();

    println!("Stack: {s1} in {:?}", stack_time);
    println!("Heap:  {s2} in {:?}", heap_time);
    println!("Heap is ~{:.0}x slower", heap_time.as_nanos() as f64 / stack_time.as_nanos() as f64);
}
```

### Vec growth and reallocation:

```rust
fn main() {
    let mut v = Vec::new();
    let mut last_cap = 0;
    let mut reallocs = 0;

    for i in 0..1000 {
        v.push(i);
        if v.capacity() != last_cap {
            if last_cap > 0 { reallocs += 1; }
            println!("  len={:4}, cap={:4} (reallocated!)", v.len(), v.capacity());
            last_cap = v.capacity();
        }
    }
    println!("Total reallocations: {reallocs}");

    // Pre-allocate to avoid reallocs:
    let mut v2 = Vec::with_capacity(1000);
    for i in 0..1000 { v2.push(i); }
    println!("Pre-allocated: 0 reallocations, cap={}", v2.capacity());
}
```

---

## 6. Box — Moving to the Heap

```rust
fn main() {
    // Small values — no reason for heap
    let stack_val = 42;          // 4 bytes on stack
    let heap_val = Box::new(42); // 4 bytes on heap + 8 byte pointer

    // Large values — might want heap to avoid large stack frames
    let large_stack = [0u8; 1024];     // 1KB on stack
    let large_heap = Box::new([0u8; 1024]); // 1KB on heap

    // Recursive types — MUST use heap
    enum List {
        Cons(i32, Box<List>),
        Nil,
    }

    let list = List::Cons(1, Box::new(List::Cons(2, Box::new(List::Nil))));
}
```

---

## 7. Measuring Allocations

```rust
fn main() {
    // Check sizes at compile time
    println!("i32:     {} bytes", std::mem::size_of::<i32>());
    println!("i64:     {} bytes", std::mem::size_of::<i64>());
    println!("String:  {} bytes (stack portion)", std::mem::size_of::<String>());
    println!("Vec<i32>:{} bytes (stack portion)", std::mem::size_of::<Vec<i32>>());
    println!("Box<i32>:{} bytes (pointer)", std::mem::size_of::<Box<i32>>());
    println!("[i32;10]:{} bytes", std::mem::size_of::<[i32; 10]>());
    println!("Option<i32>: {} bytes", std::mem::size_of::<Option<i32>>());
    println!("Option<Box<i32>>: {} bytes", std::mem::size_of::<Option<Box<i32>>>());
    // Option<Box<T>> is same size as Box<T> — null pointer optimization!
}
```

---

## 8. Performance Guidelines

### ✅ DO:

```rust
// Pre-allocate when size is known
let mut v = Vec::with_capacity(1000);

// Use stack types when possible
let point = (1.0_f64, 2.0_f64);  // stack

// Use slices instead of owned collections
fn process(data: &[i32]) -> i32 { data.iter().sum() }

// Reuse allocations
let mut buffer = String::with_capacity(256);
for i in 0..100 {
    buffer.clear();  // reuse allocation
    buffer.push_str(&format!("item {i}"));
}
```

### ❌ AVOID:

```rust
// Unnecessary heap allocation in hot loops
for i in 0..1_000_000 {
    let s = format!("item {i}");  // allocates each iteration
    // use s...
}

// Boxing small Copy types
let x = Box::new(42_i32);  // pointless — 4 bytes on stack is fine

// Creating new Vecs when you can reuse
for _ in 0..1000 {
    let v = vec![0; 100];  // allocates each time
}
```

---

## 9. Summary Cheat Sheet

```
STACK
────────────────────────────────────────────────────────────
Fast, automatic, fixed-size
i32, f64, bool, char, arrays, tuples, references
Each thread has its own stack (~8MB default)

HEAP
────────────────────────────────────────────────────────────
Slower, dynamic, flexible
String, Vec, Box, HashMap — data portion
Shared via Rc/Arc across scopes

SIZES
────────────────────────────────────────────────────────────
std::mem::size_of::<T>()       compile-time size
std::mem::size_of_val(&val)    runtime size

OPTIMIZATION
────────────────────────────────────────────────────────────
Vec::with_capacity(n)          pre-allocate
String::with_capacity(n)       pre-allocate
Reuse buffers (clear + refill)
Avoid Box for small Copy types
Use slices (&[T]) over owned Vec where possible

NULL POINTER OPT
────────────────────────────────────────────────────────────
Option<Box<T>>     same size as Box<T>  (8 bytes, not 16!)
Option<&T>         same size as &T      (8 bytes, not 16!)
```

---

## What's Next?

**Lesson 75 — Zero-Cost Abstractions** — How Rust's iterators compile to the same code as manual loops. Monomorphization and inlining.

## Further Reading
- [The Rust Book — Ch 4.1: Ownership](https://doc.rust-lang.org/book/ch04-01-what-is-ownership.html#the-stack-and-the-heap)
- [Visualizing Memory Layout](https://cheats.rs/#memory-layout)

---

*Stack vs Heap: know where your bytes live! 🦀*
