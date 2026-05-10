# 🧪 Lesson 99 — Questions: #![no_std] (EM1)

> **Lesson:** [lesson_99_no_std.md](./lesson_99_no_std.md)  
> **Answers:** [lesson_99_answers.md](./lesson_99_answers.md)

---

## Section A — Conceptual

### Q1
Explain the three layers: `core`, `alloc`, and `std`. What does each add, and what does each require from the environment?

### Q2
Why do `#![no_std]` **binaries** need a `#[panic_handler]`, but `#![no_std]` **libraries** don't?

### Q3
What is `#![no_main]` and when do you need it alongside `#![no_std]`?

---

## Section B — Write It Yourself

### Q4 — no_std ring buffer (Roadmap Practice Task)
Write a `#![no_std]` library crate containing a `RingBuffer<const N: usize>` that:
1. Uses a fixed-size array `[u8; N]` (no heap allocation)
2. Has `push(&mut self, byte: u8) -> bool` — returns false if full
3. Has `pop(&mut self) -> Option<u8>` — returns None if empty
4. Has `len()`, `is_empty()`, `is_full()` methods
5. Write `#[cfg(test)]` tests that use `std` (even though the library is no_std)

### Q5 — no_std with alloc
Convert the ring buffer to use `alloc::vec::Vec<u8>` internally (dynamically sized). Show the required `extern crate alloc;` declaration and imports.

### Q6 — Panic handler
Write a bare-metal binary skeleton with:
1. `#![no_std]` and `#![no_main]`
2. A `#[panic_handler]` that loops forever
3. A `#[no_mangle] pub extern "C" fn _start() -> !` entry point
4. Explain why `_start` returns `!` (the never type)

---

## Section C — True or False?

### Q7
1. `#![no_std]` means you can't use `Option` or `Result`.
2. `#![no_std]` libraries need a `#[panic_handler]`.
3. `Vec` requires `alloc`, not just `core`.
4. `HashMap` is available in `core`.
5. Tests `#[cfg(test)]` in a no_std library can still use `std`.
6. `extern crate alloc;` is needed to use `Vec` in a no_std crate.

### Q8
When would you choose `#![no_std]` over standard Rust? List three real-world use cases and explain why each requires no_std.

---

*Rust at the bare metal! 🦀*
