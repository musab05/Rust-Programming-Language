# ✅ Lesson 99 — Answers: #![no_std] (EM1)

---

## Section A

### A1
| Layer | What It Adds | Requires |
|-------|-------------|----------|
| **`core`** | Primitives (`u8`, `bool`), `Option`, `Result`, iterators, `core::fmt`, `core::ops`, slices, arrays | Nothing — runs on bare metal |
| **`alloc`** | Heap types: `Vec`, `String`, `Box`, `Rc`, `Arc`, `BTreeMap` | A **global allocator** (`#[global_allocator]`) |
| **`std`** | File I/O, networking, threads, `HashMap`, `println!`, `std::time`, `std::env` | A full **operating system** |

Each layer is a superset: `core ⊂ alloc ⊂ std`.

### A2
- **Libraries** are compiled into other crates — the final binary (or test harness) provides the panic handler. A library just needs to compile, not link.
- **Binaries** are the final executable — they must define what happens on panic because there's no test harness or `std` runtime to fall back to.

When you run `cargo test` on a no_std library, the test binary links `std`, which provides the default panic handler.

### A3
`#![no_main]` tells the compiler: "I will provide my own entry point — don't generate `main()`."

You need it with `#![no_std]` when:
- Writing **bare-metal binaries** (microcontrollers, OS kernels) — the entry point is `_start`, `Reset`, or a chip-specific handler, not `main()`
- There is **no Rust runtime** to set up the stack, call constructors, or invoke `main()`

For `#![no_std]` **libraries**, you don't need `#![no_main]` because libraries don't have entry points.

---

## Section B

### A4
```rust
// src/lib.rs
#![no_std]

/// A fixed-size circular buffer — no heap allocation.
pub struct RingBuffer<const N: usize> {
    data: [u8; N],
    head: usize,
    len: usize,
}

impl<const N: usize> RingBuffer<N> {
    /// Create an empty ring buffer.
    pub const fn new() -> Self {
        RingBuffer {
            data: [0; N],
            head: 0,
            len: 0,
        }
    }

    /// Push a byte. Returns false if the buffer is full.
    pub fn push(&mut self, byte: u8) -> bool {
        if self.len >= N {
            return false;
        }
        let idx = (self.head + self.len) % N;
        self.data[idx] = byte;
        self.len += 1;
        true
    }

    /// Pop a byte from the front. Returns None if empty.
    pub fn pop(&mut self) -> Option<u8> {
        if self.len == 0 {
            return None;
        }
        let byte = self.data[self.head];
        self.head = (self.head + 1) % N;
        self.len -= 1;
        Some(byte)
    }

    pub fn len(&self) -> usize { self.len }
    pub fn is_empty(&self) -> bool { self.len == 0 }
    pub fn is_full(&self) -> bool { self.len >= N }
}

// Tests CAN use std even though the library is no_std!
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn push_and_pop() {
        let mut buf = RingBuffer::<4>::new();
        assert!(buf.is_empty());
        assert!(buf.push(10));
        assert!(buf.push(20));
        assert_eq!(buf.len(), 2);
        assert_eq!(buf.pop(), Some(10));
        assert_eq!(buf.pop(), Some(20));
        assert!(buf.is_empty());
    }

    #[test]
    fn full_buffer() {
        let mut buf = RingBuffer::<3>::new();
        assert!(buf.push(1));
        assert!(buf.push(2));
        assert!(buf.push(3));
        assert!(buf.is_full());
        assert!(!buf.push(4)); // rejected
    }

    #[test]
    fn wrap_around() {
        let mut buf = RingBuffer::<3>::new();
        buf.push(1); buf.push(2); buf.push(3);
        buf.pop(); // removes 1, head moves
        assert!(buf.push(4)); // wraps to index 0
        assert_eq!(buf.pop(), Some(2));
        assert_eq!(buf.pop(), Some(3));
        assert_eq!(buf.pop(), Some(4));
        assert!(buf.is_empty());
    }
}
```

### A5
```rust
#![no_std]
extern crate alloc;

use alloc::vec::Vec;

/// A dynamically-sized ring buffer using heap allocation.
pub struct DynRingBuffer {
    data: Vec<u8>,
    head: usize,
    len: usize,
    capacity: usize,
}

impl DynRingBuffer {
    pub fn new(capacity: usize) -> Self {
        DynRingBuffer {
            data: alloc::vec![0u8; capacity],
            head: 0,
            len: 0,
            capacity,
        }
    }

    pub fn push(&mut self, byte: u8) -> bool {
        if self.len >= self.capacity { return false; }
        let idx = (self.head + self.len) % self.capacity;
        self.data[idx] = byte;
        self.len += 1;
        true
    }

    pub fn pop(&mut self) -> Option<u8> {
        if self.len == 0 { return None; }
        let byte = self.data[self.head];
        self.head = (self.head + 1) % self.capacity;
        self.len -= 1;
        Some(byte)
    }

    pub fn len(&self) -> usize { self.len }
    pub fn is_empty(&self) -> bool { self.len == 0 }
}
```
Key differences: `extern crate alloc;` must be declared, and `Vec` is imported from `alloc::vec::Vec` instead of `std::vec::Vec`. This requires a global allocator to be defined somewhere.

### A6
```rust
#![no_std]
#![no_main]

use core::panic::PanicInfo;

/// Custom panic handler — required for no_std binaries.
/// On embedded: you might blink an LED, log to UART, or trigger a reset.
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    // The ! return type means this function never returns.
    // On bare metal, the safest thing is to halt.
    loop {
        // Optionally: core::hint::spin_loop() to reduce power consumption
        core::hint::spin_loop();
    }
}

/// Entry point for bare-metal execution.
/// The name `_start` is the conventional ELF entry point.
/// `extern "C"` ensures C ABI calling convention.
/// `#[no_mangle]` prevents Rust name mangling so the linker can find it.
#[no_mangle]
pub extern "C" fn _start() -> ! {
    // Your bare-metal code here
    let _x = 2 + 2;

    // Must never return — there's no OS to return to!
    loop {}
}
```
`_start` returns `!` (the never type) because on bare metal there is **no operating system** to return to. If the entry point returned, the CPU would execute whatever random bytes follow in memory — undefined behavior. The `!` type enforces at compile time that the function contains an infinite loop or diverging call.

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `Option` and `Result` are in `core` — available in no_std |
| 2 | **False** | Only binaries need a panic handler; libraries don't |
| 3 | **True** | `Vec` requires heap allocation, which is in `alloc` |
| 4 | **False** | `HashMap` is only in `std` (it uses a hasher that needs randomness from the OS) |
| 5 | **True** | `#[cfg(test)]` modules can use `std` because the test harness provides it |
| 6 | **True** | You must declare `extern crate alloc;` to access `Vec`, `String`, `Box` in no_std |

### A8
Three real-world no_std use cases:

1. **Microcontrollers (ARM Cortex-M):** There's no OS — no filesystem, no heap (unless you set one up), no threads. You interact directly with hardware registers. `std` requires OS syscalls that don't exist.

2. **Operating system kernels:** You're *building* the OS, so you can't depend on one. `std`'s I/O, threading, and allocation all rely on the kernel you haven't written yet. Only `core` works.

3. **Bootloaders/firmware:** Code runs before any OS loads. The environment is bare metal — only CPU and RAM are available. Even `alloc` may not work if you haven't set up the heap yet.

Common thread: no_std is required when there is **no OS** or when you ARE the OS.

---

## 🏆 Lesson 99 Complete!

**Next up:** [Lesson 100 — Embedded HAL Pattern](../lesson_100_embedded_hal/lesson_100_embedded_hal.md) 🦀
