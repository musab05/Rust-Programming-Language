# 🧪 Lesson 64 — Questions: FFI (FF1)

> **Lesson:** [lesson_64_ffi.md](./lesson_64_ffi.md)  
> **Answers:** [lesson_64_answers.md](./lesson_64_answers.md)

---

## Section A — Conceptual

### Q1
Why is calling an `extern "C"` function unsafe?

### Q2
What is the difference between `CString` and `CStr`?

### Q3
What does `#[no_mangle]` do and why is it needed for FFI?

---

## Section B — Write It Yourself

### Q4 — Call C from Rust (Roadmap Practice Task)
Write Rust code that calls C's `abs()`, `pow()`, and `strlen()` functions. Wrap each in a safe Rust function.

### Q5 — Expose Rust to C
Write a Rust function `rust_multiply(a: i32, b: i32) -> i32` callable from C. Show the correct attributes.

### Q6 — String conversion
Convert a Rust `&str` to a C string, pass it to `puts()`, and explain each step.

---

## Section C — True or False?

### Q7
1. `extern "C"` functions use the C calling convention (ABI).
2. `#[repr(C)]` ensures Rust structs have C-compatible memory layout.
3. `CString::new("hello")` can fail if the string contains a null byte.
4. `extern "C"` functions are safe to call without an unsafe block.
5. `crate-type = ["cdylib"]` produces a dynamic library callable from C.
6. The `libc` crate provides portable C type aliases.

---

*FFI: bridging Rust and the C world! 🦀*
