# 📘 Lesson 96 — Inline Assembly (U5)

> **Series:** Rust From Zero · Expert Level (Gap Fill)  
> **Roadmap ID:** U5 · Category: ☢️ Unsafe  
> **Previous:** [Lesson 95 — Capstone: CLI SQL DB](../lesson_95_capstone_db/lesson_95_capstone_db.md)  
> **Next:** [Lesson 97 — Attribute & Function-like Macros](../lesson_97_proc_macros/lesson_97_proc_macros.md)  
> **Practice:** [Questions](./lesson_96_questions.md) · [Answers](./lesson_96_answers.md)  
> **Practice Task:** Read the CPU timestamp counter via `rdtsc`

---

## Table of Contents

1. [What Is Inline Assembly?](#1-what-is-inline-assembly)
2. [Basic Syntax](#2-basic-syntax)
3. [Inputs and Outputs](#3-inputs-and-outputs)
4. [Common Instructions](#4-common-instructions)
5. [Clobbers and Options](#5-clobbers-and-options)
6. [CPUID Example](#6-cpuid-example)
7. [Platform-Specific Code](#7-platform-specific-code)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. What Is Inline Assembly?

Write raw CPU instructions inside Rust code. Requires `unsafe` — the compiler can't verify assembly correctness:

```rust
use std::arch::asm;

fn main() {
    let result: u64;
    unsafe {
        asm!(
            "mov {}, 42",
            out(reg) result,
        );
    }
    println!("Result: {result}"); // 42
}
```

**Use cases:** CPUID, RDTSC, hardware access, SIMD not exposed via intrinsics, performance-critical inner loops.

---

## 2. Basic Syntax

```rust
use std::arch::asm;

fn add(a: u64, b: u64) -> u64 {
    let result: u64;
    unsafe {
        asm!(
            "add {0}, {1}",        // instruction template
            inout(reg) a => result, // a goes in, result comes out
            in(reg) b,              // b is input only
        );
    }
    result
}

fn main() {
    println!("3 + 4 = {}", add(3, 4)); // 7
}
```

### Operand types:

| Operand | Meaning |
|---|---|
| `in(reg) val` | Input: value → register |
| `out(reg) val` | Output: register → variable |
| `inout(reg) val` | Input + output (same register) |
| `inout(reg) a => b` | Input `a`, output to `b` |
| `const N` | Compile-time constant |

---

## 3. Inputs and Outputs

```rust
use std::arch::asm;

fn multiply(a: u64, b: u64) -> u64 {
    let lo: u64;
    unsafe {
        asm!(
            "mul {b}",
            b = in(reg) b,
            inout("rax") a => lo,
            out("rdx") _,          // high bits discarded
        );
    }
    lo
}

fn swap(a: &mut u64, b: &mut u64) {
    unsafe {
        asm!(
            "xchg [{0}], {1}",
            in(reg) a as *mut u64,
            inout(reg) *b,
        );
        *a = *b;
    }
}

fn main() {
    println!("6 * 7 = {}", multiply(6, 7));
}
```

---

## 4. Common Instructions

```rust
use std::arch::asm;

// No-op (do nothing)
fn nop() { unsafe { asm!("nop"); } }

// Read timestamp counter
fn rdtsc() -> u64 {
    let lo: u32;
    let hi: u32;
    unsafe {
        asm!(
            "rdtsc",
            out("eax") lo,
            out("edx") hi,
        );
    }
    ((hi as u64) << 32) | (lo as u64)
}

// Pause hint (spin-loop optimization)
fn spin_pause() { unsafe { asm!("pause"); } }

fn main() {
    let t1 = rdtsc();
    nop();
    let t2 = rdtsc();
    println!("RDTSC delta: {} cycles", t2 - t1);
}
```

---

## 5. Clobbers and Options

```rust
use std::arch::asm;

fn demo() {
    unsafe {
        asm!(
            "nop",
            options(nomem),     // doesn't access memory
        );

        asm!(
            "nop",
            options(nostack),   // doesn't use stack
        );

        asm!(
            "nop",
            options(pure, nomem, nostack), // no side effects
        );
    }
}
```

| Option | Meaning |
|---|---|
| `nomem` | Doesn't read/write memory |
| `nostack` | Doesn't use the stack |
| `pure` | No side effects (can be optimized) |
| `preserves_flags` | Doesn't modify CPU flags |
| `att_syntax` | Use AT&T syntax instead of Intel |

---

## 6. CPUID Example

```rust
use std::arch::asm;

fn cpuid(leaf: u32) -> (u32, u32, u32, u32) {
    let (eax, ebx, ecx, edx);
    unsafe {
        asm!(
            "cpuid",
            inout("eax") leaf => eax,
            out("ebx") ebx,
            out("ecx") ecx,
            out("edx") edx,
        );
    }
    (eax, ebx, ecx, edx)
}

fn cpu_vendor() -> String {
    let (_, ebx, ecx, edx) = cpuid(0);
    let bytes: [u8; 12] = unsafe {
        std::mem::transmute([ebx, edx, ecx])
    };
    String::from_utf8_lossy(&bytes).to_string()
}

fn main() {
    println!("CPU Vendor: {}", cpu_vendor());
}
```

---

## 7. Platform-Specific Code

```rust
#[cfg(target_arch = "x86_64")]
fn timestamp() -> u64 {
    let lo: u32; let hi: u32;
    unsafe { std::arch::asm!("rdtsc", out("eax") lo, out("edx") hi); }
    ((hi as u64) << 32) | lo as u64
}

#[cfg(target_arch = "aarch64")]
fn timestamp() -> u64 {
    let val: u64;
    unsafe { std::arch::asm!("mrs {}, cntvct_el0", out(reg) val); }
    val
}

#[cfg(not(any(target_arch = "x86_64", target_arch = "aarch64")))]
fn timestamp() -> u64 { 0 }
```

---

## 8. Summary Cheat Sheet

```
SYNTAX
────────────────────────────────────────────────────────────
use std::arch::asm;
unsafe { asm!("instruction", operands, options); }

OPERANDS
────────────────────────────────────────────────────────────
in(reg) val              input register
out(reg) val             output register
inout(reg) a => b        input a, output b
in("rax") val            specific register
const N                  compile-time constant

OPTIONS
────────────────────────────────────────────────────────────
nomem, nostack, pure, preserves_flags, att_syntax

COMMON INSTRUCTIONS (x86-64)
────────────────────────────────────────────────────────────
nop            no-op
rdtsc          read timestamp counter
cpuid          CPU identification
pause          spin-loop hint
```

---

## What's Next?

**Lesson 97 — Attribute & Function-like Macros** — Procedural macros that transform code at compile time.

---

*Inline assembly: when you need the metal! 🦀*
