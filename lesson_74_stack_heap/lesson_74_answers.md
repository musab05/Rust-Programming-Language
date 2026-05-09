# ✅ Lesson 74 — Answers: Stack vs Heap (P1)

---

## Section A

### A1
| Variable | Stack | Heap | Explanation |
|---|---|---|---|
| `a: i32` | ✅ all | — | Primitive, fixed size |
| `b: String` | metadata (24B) | chars | ptr+len+cap on stack, "hello" on heap |
| `c: [u8; 100]` | ✅ all (100B) | — | Fixed-size array, all on stack |
| `d: Vec<i32>` | metadata (24B) | elements | ptr+len+cap on stack, [1,2,3] on heap |
| `e: &str` | pointer (16B) | — | Fat pointer on stack → static memory |
| `f: Box<f64>` | pointer (8B) | f64 (8B) | Pointer on stack, value on heap |

---

## Section B

### A2
```rust
use std::time::Instant;

fn sum_stack(n: i64) -> i64 {
    let mut sum: i64 = 0;
    for i in 0..n { let x: i64 = i; sum += x; }
    sum
}

fn sum_heap(n: i64) -> i64 {
    let mut sum: i64 = 0;
    for i in 0..n { let x = Box::new(i); sum += *x; }
    sum
}

fn main() {
    let n = 1_000_000;

    let t = Instant::now();
    let s1 = sum_stack(n);
    let stack_time = t.elapsed();

    let t = Instant::now();
    let s2 = sum_heap(n);
    let heap_time = t.elapsed();

    println!("Stack: {s1} in {:?}", stack_time);
    println!("Heap:  {s2} in {:?}", heap_time);
}
```

### A3
```rust
fn main() {
    println!("=== Without pre-allocation ===");
    let mut v1 = Vec::new();
    let mut last = 0;
    for i in 0..20 {
        v1.push(i);
        if v1.capacity() != last {
            println!("  len={:2}, cap={:2}", v1.len(), v1.capacity());
            last = v1.capacity();
        }
    }

    println!("\n=== With pre-allocation ===");
    let mut v2 = Vec::with_capacity(20);
    println!("  Initial cap={}", v2.capacity());
    for i in 0..20 { v2.push(i); }
    println!("  Final   cap={} (no reallocs!)", v2.capacity());
}
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Stack just moves the stack pointer — no allocator call |
| 2 | **True** | ptr, len, cap are on the stack; character data is on the heap |
| 3 | **False** | Null pointer optimization: both are 8 bytes |
| 4 | **True** | Each OS thread has its own stack |
| 5 | **True** | Pre-allocation avoids growth reallocs within capacity |
| 6 | **False** | It returns the stack portion size (24 bytes), not the heap data |

---

## 🏆 Lesson 74 Complete!

**Next up:** [Lesson 75 — Zero-Cost Abstractions](../lesson_75_zero_cost/lesson_75_zero_cost.md) 🦀
