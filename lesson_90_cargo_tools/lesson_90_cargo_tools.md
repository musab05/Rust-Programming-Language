# 📘 Lesson 90 — cargo-expand & cargo-asm (B6)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** B6 · Category: 🔧 Tooling  
> **Previous:** [Lesson 89 — Profiling](../lesson_89_profiling/lesson_89_profiling.md)  
> **Next:** [Lesson 91 — Cross-Compilation](../lesson_91_cross_compile/lesson_91_cross_compile.md)  
> **Practice:** [Questions](./lesson_90_questions.md) · [Answers](./lesson_90_answers.md)  
> **Practice Task:** Expand a derive macro and inspect the generated code

---

## Table of Contents

1. [Why Inspect Generated Code?](#1-why-inspect-generated-code)
2. [Installing the Tools](#2-installing-the-tools)
3. [cargo expand — Macro Expansion](#3-cargo-expand--macro-expansion)
4. [Understanding Expanded Code](#4-understanding-expanded-code)
5. [cargo-show-asm — Assembly Output](#5-cargo-show-asm--assembly-output)
6. [Reading Assembly](#6-reading-assembly)
7. [Godbolt as Alternative](#7-godbolt-as-alternative)
8. [Practical Use Cases](#8-practical-use-cases)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Why Inspect Generated Code?

```
Your Rust code
      │
      ▼ (macro expansion)
Expanded Rust code          ← cargo expand shows this
      │
      ▼ (compilation)
LLVM IR
      │
      ▼ (optimization + codegen)
Assembly / Machine code     ← cargo-show-asm shows this
```

**Use cases:**
- Debug derive macros (what does `#[derive(Debug)]` actually generate?)
- Verify zero-cost abstractions (do iterators really compile to loops?)
- Understand compiler optimizations
- Learn how macros and traits work internally

---

## 2. Installing the Tools

```bash
# cargo-expand: shows macro-expanded Rust code
cargo install cargo-expand

# Requires nightly Rust (for expansion)
rustup install nightly

# cargo-show-asm: shows generated assembly
cargo install cargo-show-asm
```

---

## 3. cargo expand — Macro Expansion

### See what derive macros generate:

```rust
// src/main.rs
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

fn main() {
    let p = Point { x: 1.0, y: 2.0 };
    println!("{:?}", p);
}
```

```bash
# Expand all macros
cargo +nightly expand

# Expand a specific item
cargo +nightly expand --lib    # library only
cargo +nightly expand --bin main  # binary only
```

### Output (simplified):

```rust
// What #[derive(Debug)] generates:
impl ::core::fmt::Debug for Point {
    fn fmt(&self, f: &mut ::core::fmt::Formatter) -> ::core::fmt::Result {
        ::core::fmt::Formatter::debug_struct_field2_finish(
            f, "Point", "x", &self.x, "y", &&self.y,
        )
    }
}

// What #[derive(Clone)] generates:
impl ::core::clone::Clone for Point {
    fn clone(&self) -> Point {
        Point {
            x: ::core::clone::Clone::clone(&self.x),
            y: ::core::clone::Clone::clone(&self.y),
        }
    }
}

// What #[derive(PartialEq)] generates:
impl ::core::cmp::PartialEq for Point {
    fn eq(&self, other: &Point) -> bool {
        self.x == other.x && self.y == other.y
    }
}
```

---

## 4. Understanding Expanded Code

### Expand println!:

```rust
fn main() {
    let name = "World";
    println!("Hello, {name}!");
}
```

```bash
cargo +nightly expand
```

```rust
// println! expands to:
fn main() {
    let name = "World";
    {
        ::std::io::_print(
            format_args!("Hello, {0}!\n", name)
        );
    }
}
```

### Expand vec!:

```rust
fn main() {
    let v = vec![1, 2, 3];
}
```

```rust
// vec! expands to:
fn main() {
    let v = <[_]>::into_vec(
        #[rustc_box]
        ::alloc::boxed::Box::new([1, 2, 3])
    );
}
```

---

## 5. cargo-show-asm — Assembly Output

See the actual machine instructions:

```rust
// src/lib.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

pub fn sum_iter(data: &[i32]) -> i32 {
    data.iter().sum()
}

pub fn sum_loop(data: &[i32]) -> i32 {
    let mut s = 0;
    for &x in data { s += x; }
    s
}
```

```bash
# List all available functions
cargo asm --lib

# Show assembly for a specific function
cargo asm --lib my_crate::add

# Show with Intel syntax (default) or AT&T
cargo asm --lib my_crate::sum_iter
cargo asm --lib my_crate::sum_loop
```

### Example output:

```asm
; my_crate::add
; fn add(a: i32, b: i32) -> i32
    lea     eax, [rdi + rsi]    ; a + b in one instruction
    ret

; Both sum_iter and sum_loop produce nearly identical assembly!
```

---

## 6. Reading Assembly

### Key x86-64 registers:

| Register | Purpose |
|---|---|
| `rax` / `eax` | Return value |
| `rdi`, `rsi`, `rdx`, `rcx` | 1st–4th arguments |
| `rsp` | Stack pointer |
| `rbp` | Frame pointer |

### Common instructions:

```asm
mov  rax, rdi    ; copy value
add  eax, esi    ; add
lea  rax, [rdi + rsi]  ; add (and store address)
cmp  rax, rsi    ; compare
je   .label      ; jump if equal
jmp  .label      ; unconditional jump
call function    ; function call
ret              ; return
```

### What to look for:

```
✅ Short, no-call assembly = inlined, zero-cost
✅ SIMD instructions (paddd, etc.) = auto-vectorization
❌ call alloc::alloc = heap allocation
❌ call core::panicking = bounds check not elided
❌ Many branches = complex control flow
```

---

## 7. Godbolt as Alternative

[godbolt.org](https://godbolt.org/) provides an interactive web-based assembly viewer:

1. Select **Rust** as the language
2. Paste your code
3. Add `-O` in compiler options (for optimization)
4. Assembly appears side-by-side with source

```rust
// Paste this in godbolt.org:
pub fn dot_product(a: &[f64], b: &[f64]) -> f64 {
    a.iter().zip(b.iter()).map(|(x, y)| x * y).sum()
}

// Compare with:
pub fn dot_product_manual(a: &[f64], b: &[f64]) -> f64 {
    let mut sum = 0.0;
    let len = a.len().min(b.len());
    for i in 0..len { sum += a[i] * b[i]; }
    sum
}
// They produce very similar optimized assembly!
```

---

## 8. Practical Use Cases

### 1. Verify zero-cost abstraction:

```bash
# Do iterators really compile to loops?
cargo asm --lib my_crate::sum_iter
cargo asm --lib my_crate::sum_loop
# Compare output — should be nearly identical
```

### 2. Debug a derive macro:

```bash
# What does my custom #[derive(MyTrait)] generate?
cargo +nightly expand
# Find MyTrait implementation in the output
```

### 3. Check bounds elision:

```rust
pub fn safe_sum(data: &[i32]) -> i32 {
    let mut s = 0;
    for i in 0..data.len() {
        s += data[i];  // does the compiler remove the bounds check?
    }
    s
}
```

```bash
cargo asm --lib my_crate::safe_sum
# Look for: call core::panicking::panic_bounds_check
# If absent — the compiler proved the access is safe!
```

### 4. Understand monomorphization:

```bash
# See separate versions generated for generic function
cargo asm --lib
# Lists: my_crate::process::<i32>, my_crate::process::<String>, etc.
```

---

## 9. Summary Cheat Sheet

```
CARGO EXPAND
────────────────────────────────────────────────────────────
cargo +nightly expand       expand all macros
cargo +nightly expand --lib library only
Shows: derive output, macro! output, desugared syntax

CARGO-SHOW-ASM
────────────────────────────────────────────────────────────
cargo asm --lib                 list functions
cargo asm --lib crate::func    show assembly for func
Shows: actual x86/ARM instructions

GODBOLT
────────────────────────────────────────────────────────────
godbolt.org                    web-based, interactive
Use -O flag for optimized output
Side-by-side source ↔ assembly

WHAT TO LOOK FOR
────────────────────────────────────────────────────────────
Inlining            small functions → no call instruction
Zero-cost           iterator chain ≈ manual loop assembly
Auto-vectorization  SIMD instructions (paddd, mulps)
Bounds checks       call panic_bounds_check (bad)
Allocations         call alloc::alloc (potentially slow)
```

---

## What's Next?

**Lesson 91 — Cross-Compilation** — Build Rust programs for different targets (Linux, macOS, Windows, ARM).

## Further Reading
- [cargo-expand](https://github.com/dtolnay/cargo-expand)
- [cargo-show-asm](https://github.com/pacak/cargo-show-asm)
- [Godbolt Compiler Explorer](https://godbolt.org/)

---

*Peek behind the curtain: see what the compiler actually does! 🦀*
