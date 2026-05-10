# 📘 Lesson 99 — #![no_std] (EM1)

> **Series:** Rust From Zero · Expert Level (Gap Fill)  
> **Roadmap ID:** EM1 · Category: 🔩 Embedded  
> **Previous:** [Lesson 98 — SIMD & Intrinsics](../lesson_98_simd/lesson_98_simd.md)  
> **Next:** [Lesson 100 — Embedded HAL Pattern](../lesson_100_embedded_hal/lesson_100_embedded_hal.md)  
> **Practice:** [Questions](./lesson_99_questions.md) · [Answers](./lesson_99_answers.md)  
> **Practice Task:** Write a no_std library with custom panic handler

---

## Table of Contents

1. [What Is no_std?](#1-what-is-no_std)
2. [std vs core vs alloc](#2-std-vs-core-vs-alloc)
3. [Creating a no_std Library](#3-creating-a-no_std-library)
4. [Panic Handler](#4-panic-handler)
5. [no_std with alloc](#5-no_std-with-alloc)
6. [Bare-Metal Binary](#6-bare-metal-binary)
7. [When to Use no_std](#7-when-to-use-no_std)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. What Is no_std?

`#![no_std]` tells Rust: **don't link the standard library**. Use only `core` (and optionally `alloc`):

```
┌─────────────────────────────────────────┐
│                  std                     │ ← OS-dependent (fs, net, threads)
│  ┌───────────────────────────────────┐  │
│  │              alloc                │  │ ← Heap allocation (Vec, String, Box)
│  │  ┌─────────────────────────────┐  │  │
│  │  │            core             │  │  │ ← No OS, no heap (Option, Result, iterators)
│  │  └─────────────────────────────┘  │  │
│  └───────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

---

## 2. std vs core vs alloc

| Feature | `core` | `alloc` | `std` |
|---|---|---|---|
| Option, Result | ✅ | ✅ | ✅ |
| Iterators | ✅ | ✅ | ✅ |
| Primitive types | ✅ | ✅ | ✅ |
| Vec, String, Box | ❌ | ✅ | ✅ |
| HashMap | ❌ | ❌ | ✅ |
| File I/O | ❌ | ❌ | ✅ |
| Networking | ❌ | ❌ | ✅ |
| Threads | ❌ | ❌ | ✅ |
| Requires OS | ❌ | ❌ | ✅ |
| Requires heap | ❌ | ✅ | ✅ |

---

## 3. Creating a no_std Library

```rust
// src/lib.rs
#![no_std]

/// A simple fixed-size circular buffer
pub struct RingBuffer<const N: usize> {
    data: [u8; N],
    head: usize,
    len: usize,
}

impl<const N: usize> RingBuffer<N> {
    pub const fn new() -> Self {
        RingBuffer { data: [0; N], head: 0, len: 0 }
    }

    pub fn push(&mut self, byte: u8) -> bool {
        if self.len >= N { return false; }
        let idx = (self.head + self.len) % N;
        self.data[idx] = byte;
        self.len += 1;
        true
    }

    pub fn pop(&mut self) -> Option<u8> {
        if self.len == 0 { return None; }
        let byte = self.data[self.head];
        self.head = (self.head + 1) % N;
        self.len -= 1;
        Some(byte)
    }

    pub fn len(&self) -> usize { self.len }
    pub fn is_empty(&self) -> bool { self.len == 0 }
    pub fn is_full(&self) -> bool { self.len >= N }
}

// Tests can still use std!
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_ring_buffer() {
        let mut buf = RingBuffer::<4>::new();
        assert!(buf.push(1)); assert!(buf.push(2));
        assert!(buf.push(3)); assert!(buf.push(4));
        assert!(!buf.push(5)); // full
        assert_eq!(buf.pop(), Some(1));
        assert!(buf.push(5)); // wraps around
        assert_eq!(buf.pop(), Some(2));
    }
}
```

---

## 4. Panic Handler

Without `std`, you must provide your own panic handler:

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    // On embedded: blink an LED, log to UART, or just loop
    loop {}
}
```

### For libraries — no panic handler needed:

```rust
// lib.rs — libraries don't need a panic handler
#![no_std]

pub fn add(a: u32, b: u32) -> u32 { a + b }
```

---

## 5. no_std with alloc

Use heap allocation without the full standard library:

```rust
#![no_std]
extern crate alloc;

use alloc::vec::Vec;
use alloc::string::String;
use alloc::boxed::Box;

pub fn example() -> Vec<u32> {
    let mut v = Vec::new();
    v.push(1); v.push(2); v.push(3);
    v
}

pub fn greet(name: &str) -> String {
    let mut s = String::from("Hello, ");
    s.push_str(name);
    s
}
```

Requires a **global allocator** on bare metal:

```rust
use core::alloc::{GlobalAlloc, Layout};

struct BumpAllocator { /* ... */ }
unsafe impl GlobalAlloc for BumpAllocator {
    unsafe fn alloc(&self, layout: Layout) -> *mut u8 { /* ... */ core::ptr::null_mut() }
    unsafe fn dealloc(&self, _ptr: *mut u8, _layout: Layout) { /* ... */ }
}

#[global_allocator]
static ALLOC: BumpAllocator = BumpAllocator { /* ... */ };
```

---

## 6. Bare-Metal Binary

A complete bare-metal program (no OS):

```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;

#[panic_handler]
fn panic(_info: &PanicInfo) -> ! { loop {} }

// Entry point (name depends on target)
#[no_mangle]
pub extern "C" fn _start() -> ! {
    // Your bare-metal code here
    let x = 2 + 2;
    loop {}
}
```

```toml
# .cargo/config.toml
[build]
target = "thumbv7em-none-eabihf"  # ARM Cortex-M4

[target.thumbv7em-none-eabihf]
rustflags = ["-C", "link-arg=-Tlink.x"]
```

---

## 7. When to Use no_std

| Use Case | Need `no_std`? |
|---|---|
| Microcontrollers (ARM Cortex-M) | ✅ Yes |
| Operating system kernels | ✅ Yes |
| Bootloaders | ✅ Yes |
| WebAssembly (minimal) | Sometimes |
| Shared libraries (max compat) | Sometimes |
| Desktop/server applications | ❌ No |
| Web servers | ❌ No |

---

## 8. Summary Cheat Sheet

```
DECLARATION
────────────────────────────────────────────────────────────
#![no_std]                    no standard library
#![no_main]                   no main() entry point
#[panic_handler]              custom panic behavior
extern crate alloc;           opt-in heap allocation

AVAILABLE IN core
────────────────────────────────────────────────────────────
Option, Result, iterators, slices, arrays
core::fmt, core::ops, core::cmp
Primitive types (u8, i32, f64, bool, &str)

NEEDS alloc
────────────────────────────────────────────────────────────
Vec, String, Box, Rc, Arc, BTreeMap
(requires a global allocator)

NEEDS std
────────────────────────────────────────────────────────────
File I/O, networking, threads, HashMap
println!, std::time, std::env
```

---

## What's Next?

**Lesson 100 — Embedded HAL Pattern** — Hardware abstraction for portable embedded drivers.

---

*no_std: Rust at the bare metal! 🦀*
