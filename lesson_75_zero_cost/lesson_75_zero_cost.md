# 📘 Lesson 75 — Zero-Cost Abstractions (P2)

> **Series:** Rust From Zero · Intermediate Level (Gap Fill)  
> **Roadmap ID:** P2 · Category: ⚡ Performance  
> **Previous:** [Lesson 74 — Stack vs Heap](../lesson_74_stack_heap/lesson_74_stack_heap.md)  
> **Next:** [Lesson 76 — Cargo Features & Profiles](../lesson_76_cargo_features/lesson_76_cargo_features.md)  
> **Practice:** [Questions](./lesson_75_questions.md) · [Answers](./lesson_75_answers.md)  
> **Practice Task:** Compare iterator chain vs manual loop in godbolt.org

---

## Table of Contents

1. [What Are Zero-Cost Abstractions?](#1-what-are-zero-cost-abstractions)
2. [Iterators Compile to Loops](#2-iterators-compile-to-loops)
3. [Monomorphization](#3-monomorphization)
4. [Inlining](#4-inlining)
5. [Trait Static vs Dynamic Dispatch](#5-trait-static-vs-dynamic-dispatch)
6. [Closures Are Zero-Cost](#6-closures-are-zero-cost)
7. [What Is NOT Zero-Cost](#7-what-is-not-zero-cost)
8. [Verifying with Godbolt](#8-verifying-with-godbolt)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Are Zero-Cost Abstractions?

Bjarne Stroustrup's principle that Rust embraces:

> *"What you don't use, you don't pay for. And further: What you do use, you couldn't hand code any better."*

```rust
// High-level, readable code:
let sum: i32 = (0..1000)
    .filter(|x| x % 2 == 0)
    .map(|x| x * x)
    .sum();

// Compiles to the SAME assembly as:
let mut sum = 0i32;
let mut i = 0;
while i < 1000 {
    if i % 2 == 0 { sum += i * i; }
    i += 1;
}
```

No runtime overhead for the iterator version — the compiler optimizes it away.

---

## 2. Iterators Compile to Loops

```rust
fn sum_of_squares_iter(data: &[i32]) -> i32 {
    data.iter()
        .filter(|&&x| x > 0)
        .map(|&x| x * x)
        .sum()
}

fn sum_of_squares_manual(data: &[i32]) -> i32 {
    let mut sum = 0;
    for &x in data {
        if x > 0 {
            sum += x * x;
        }
    }
    sum
}

fn main() {
    let data: Vec<i32> = (-1000..1000).collect();

    use std::time::Instant;
    let t = Instant::now();
    let r1 = sum_of_squares_iter(&data);
    let iter_time = t.elapsed();

    let t = Instant::now();
    let r2 = sum_of_squares_manual(&data);
    let manual_time = t.elapsed();

    println!("Iterator: {r1} in {:?}", iter_time);
    println!("Manual:   {r2} in {:?}", manual_time);
    // Both produce identical results at identical speed
}
```

### Chained adaptors — still zero-cost:

```rust
fn process(data: &[f64]) -> f64 {
    data.iter()
        .copied()
        .filter(|&x| x > 0.0)
        .map(|x| x.sqrt())
        .enumerate()
        .filter(|(i, _)| i % 2 == 0)
        .map(|(_, val)| val)
        .fold(0.0, |acc, x| acc + x)
}
// All these adaptor layers are collapsed into a single loop by LLVM
```

---

## 3. Monomorphization

Generics are compiled into specific versions for each type used — no runtime dispatch:

```rust
fn largest<T: PartialOrd>(a: T, b: T) -> T {
    if a >= b { a } else { b }
}

fn main() {
    let x = largest(5, 10);            // compiler generates: largest_i32(i32, i32)
    let y = largest(3.14, 2.72);       // compiler generates: largest_f64(f64, f64)
    let z = largest("hello", "world"); // compiler generates: largest_str(&str, &str)

    println!("{x}, {y}, {z}");
}
```

**What happens at compile time:**

```rust
// The compiler generates THREE separate functions:
fn largest_i32(a: i32, b: i32) -> i32 { if a >= b { a } else { b } }
fn largest_f64(a: f64, b: f64) -> f64 { if a >= b { a } else { b } }
fn largest_str(a: &str, b: &str) -> &str { if a >= b { a } else { b } }
```

### Trade-off:

| | Monomorphization | Dynamic Dispatch |
|---|---|---|
| Speed | ✅ Fast (inlined, no vtable) | ❌ Slower (vtable lookup) |
| Binary size | ❌ Larger (duplicated code) | ✅ Smaller (shared code) |
| Flexibility | ❌ Types known at compile time | ✅ Types chosen at runtime |

---

## 4. Inlining

The compiler replaces function calls with the function body:

```rust
#[inline]
fn square(x: i32) -> i32 { x * x }

#[inline(always)]
fn add(a: i32, b: i32) -> i32 { a + b }

#[inline(never)]  // prevent inlining (for debugging/profiling)
fn complex_calc(x: i32) -> i32 { x * x + x + 1 }

fn main() {
    // square(5) is replaced with: 5 * 5
    // add(3, 4) is replaced with: 3 + 4
    let result = add(square(3), square(4));
    println!("{result}");  // 25
}
```

**When the compiler inlines automatically:**
- Small functions (usually < ~40 instructions)
- Functions called from one place
- Functions in the same crate
- Closures used in iterators (almost always inlined)

---

## 5. Trait Static vs Dynamic Dispatch

```rust
trait Shape {
    fn area(&self) -> f64;
}

struct Circle { radius: f64 }
impl Shape for Circle { fn area(&self) -> f64 { std::f64::consts::PI * self.radius * self.radius } }

struct Square { side: f64 }
impl Shape for Square { fn area(&self) -> f64 { self.side * self.side } }

// STATIC dispatch — monomorphized, inlined, zero-cost
fn print_area_static(shape: &impl Shape) {
    println!("Area: {:.2}", shape.area());
}

// DYNAMIC dispatch — vtable lookup, not zero-cost
fn print_area_dynamic(shape: &dyn Shape) {
    println!("Area: {:.2}", shape.area());
}

fn main() {
    let c = Circle { radius: 5.0 };
    let s = Square { side: 4.0 };

    // Static: compiler knows exact type → inlines area()
    print_area_static(&c);  // monomorphized to print_area_Circle
    print_area_static(&s);  // monomorphized to print_area_Square

    // Dynamic: compiler uses vtable → indirect function call
    let shapes: Vec<Box<dyn Shape>> = vec![Box::new(c), Box::new(s)];
    for shape in &shapes {
        print_area_dynamic(shape.as_ref());
    }
}
```

---

## 6. Closures Are Zero-Cost

Closures are compiled into structs with call methods:

```rust
fn main() {
    let offset = 10;

    // This closure:
    let add_offset = |x: i32| x + offset;

    // Is equivalent to this struct:
    // struct AddOffset { offset: i32 }
    // impl FnOnce<(i32,)> for AddOffset {
    //     fn call_once(self, (x,): (i32,)) -> i32 { x + self.offset }
    // }

    let result = add_offset(5);
    println!("{result}");  // 15

    // Size of closure = size of captured variables
    println!("Closure size: {} bytes", std::mem::size_of_val(&add_offset));  // 4 (one i32)

    // Empty closure = zero-sized!
    let no_capture = |x: i32| x * 2;
    println!("Empty closure: {} bytes", std::mem::size_of_val(&no_capture));  // 0
}
```

---

## 7. What Is NOT Zero-Cost

Some abstractions DO have runtime cost:

```rust
use std::collections::HashMap;

fn main() {
    // ❌ Dynamic dispatch (dyn Trait) — vtable lookup
    let shapes: Vec<Box<dyn std::fmt::Display>> = vec![
        Box::new(42),
        Box::new("hello"),
    ];

    // ❌ Heap allocation (Box, Vec, String, HashMap)
    let map: HashMap<String, Vec<i32>> = HashMap::new();

    // ❌ Reference counting (Rc, Arc)
    let shared = std::rc::Rc::new(42);

    // ❌ Runtime bounds checking (indexing with [i])
    let v = vec![1, 2, 3];
    let _ = v[0];  // bounds check at runtime

    // ✅ But iterators SKIP bounds checks:
    for x in v.iter() { /* no bounds check — sequential access */ }
}
```

---

## 8. Verifying with Godbolt

Visit [godbolt.org](https://godbolt.org/) to see generated assembly:

```rust
// Paste this in godbolt.org with rustc compiler
pub fn sum_iter(data: &[i32]) -> i32 {
    data.iter().filter(|&&x| x > 0).sum()
}

pub fn sum_loop(data: &[i32]) -> i32 {
    let mut s = 0;
    for &x in data { if x > 0 { s += x; } }
    s
}

// Both produce nearly identical assembly!
```

**Steps:**
1. Go to godbolt.org
2. Select "Rust" as the language
3. Paste both functions
4. Add `-O` (optimize) in compiler options
5. Compare the assembly output — they'll be nearly identical

---

## 9. Summary Cheat Sheet

```
ZERO-COST ABSTRACTIONS
────────────────────────────────────────────────────────────
Iterators     → compiled to loops (no overhead)
Generics      → monomorphized (no dispatch cost)
Closures      → compiled to structs (zero-sized if no captures)
impl Trait     → static dispatch (inlined)
Option<T>      → same size as T (null pointer opt for pointers)

NOT ZERO-COST
────────────────────────────────────────────────────────────
dyn Trait      → vtable lookup
Box/Vec/String → heap allocation
Rc/Arc         → reference counting overhead
v[i]           → runtime bounds check

MONOMORPHIZATION
────────────────────────────────────────────────────────────
fn f<T>(x: T)  → compiler generates f_i32, f_f64, f_String...
Pro: fast (inlined), Con: larger binary

INLINING
────────────────────────────────────────────────────────────
#[inline]        hint to compiler
#[inline(always)] force inline
#[inline(never)]  prevent inline (profiling)

VERIFY
────────────────────────────────────────────────────────────
godbolt.org      → compare assembly of two approaches
cargo bench      → measure real performance
```

---

## What's Next?

**Lesson 76 — Cargo Features & Profiles** — Feature flags, optional dependencies, and release optimizations.

## Further Reading
- [Rust Performance Book](https://nnethercote.github.io/perf-book/)
- [Godbolt Compiler Explorer](https://godbolt.org/)
- [The Rust Book — Comparing Performance](https://doc.rust-lang.org/book/ch13-04-performance.html)

---

*Zero-cost abstractions: write beautiful code without paying for it! 🦀*
