# 📘 Lesson 61 — Unsafe Rust (U1)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** U1 · Category: 🔓 Unsafe  
> **Previous:** [Lesson 60 — Async I/O](../lesson_60_async_io/lesson_60_async_io.md)  
> **Next:** [Lesson 62 — Declarative Macros](../lesson_62_macros/lesson_62_macros.md)  
> **Practice:** [Questions](./lesson_61_questions.md) · [Answers](./lesson_61_answers.md)  
> **Practice Task:** Implement a safe wrapper around raw pointer operations

---

## Table of Contents

1. [What Is Unsafe Rust?](#1-what-is-unsafe-rust)
2. [The Five Unsafe Superpowers](#2-the-five-unsafe-superpowers)
3. [Raw Pointers](#3-raw-pointers)
4. [Calling Unsafe Functions](#4-calling-unsafe-functions)
5. [Creating Safe Abstractions](#5-creating-safe-abstractions)
6. [Mutable Static Variables](#6-mutable-static-variables)
7. [Unsafe Traits](#7-unsafe-traits)
8. [Union Types](#8-union-types)
9. [Guidelines for Unsafe Code](#9-guidelines-for-unsafe-code)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is Unsafe Rust?

Unsafe Rust lets you bypass certain compiler guarantees. You're telling the compiler: "trust me — I've verified this is correct."

```rust
fn main() {
    let mut num = 42;
    let r1 = &num as *const i32;       // raw pointer (immutable)
    let r2 = &mut num as *mut i32;     // raw pointer (mutable)

    // Creating raw pointers is safe
    // DEREFERENCING them requires unsafe
    unsafe {
        println!("r1 = {}", *r1);
        println!("r2 = {}", *r2);
    }
}
```

**Key principle:** Unsafe doesn't turn off the borrow checker. It only unlocks five specific capabilities.

---

## 2. The Five Unsafe Superpowers

Inside `unsafe { }` you can:

| # | Superpower | Example |
|---|---|---|
| 1 | Dereference raw pointers | `*raw_ptr` |
| 2 | Call unsafe functions | `libc::malloc(...)` |
| 3 | Access mutable static variables | `COUNTER += 1` |
| 4 | Implement unsafe traits | `unsafe impl Send for MyType {}` |
| 5 | Access union fields | `my_union.field` |

Everything else (ownership, borrows, type checking) still applies.

---

## 3. Raw Pointers

`*const T` (immutable) and `*mut T` (mutable) — like C pointers:

```rust
fn main() {
    let mut value = 10;

    // Creating raw pointers — SAFE (no dereference yet)
    let ptr_const: *const i32 = &value;
    let ptr_mut: *mut i32 = &mut value;

    // Dereferencing — UNSAFE
    unsafe {
        println!("const ptr: {}", *ptr_const);  // 10
        *ptr_mut = 20;
        println!("mut ptr: {}", *ptr_mut);      // 20
    }

    println!("value: {value}");  // 20
}
```

### Raw pointers can break borrow rules:

```rust
fn main() {
    let mut data = vec![1, 2, 3, 4, 5];

    // Two mutable raw pointers to the same data — normally forbidden!
    let ptr1 = data.as_mut_ptr();
    let ptr2 = data.as_mut_ptr();

    unsafe {
        // Write to different indices — safe IF you avoid aliasing
        *ptr1.add(0) = 10;
        *ptr2.add(4) = 50;
    }

    println!("{:?}", data);  // [10, 2, 3, 4, 50]
}
```

### Null and dangling pointers:

```rust
fn main() {
    // Null pointer
    let null: *const i32 = std::ptr::null();
    println!("Is null: {}", null.is_null());  // true

    // ⚠️ Dereferencing null = undefined behavior!
    // unsafe { println!("{}", *null); }  // 💥 UB

    // Safe check first
    if !null.is_null() {
        unsafe { println!("{}", *null); }
    }
}
```

---

## 4. Calling Unsafe Functions

Some functions are marked `unsafe` because they have invariants the compiler can't check:

```rust
unsafe fn dangerous() {
    println!("This function has unchecked invariants!");
}

fn main() {
    // Must call within unsafe block
    unsafe {
        dangerous();
    }
}
```

### std library unsafe functions:

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    // Safe version — panics on out-of-bounds
    let val = v[2];
    println!("Safe: {val}");

    // Unsafe version — no bounds checking (faster, but your responsibility)
    unsafe {
        let val = *v.get_unchecked(2);
        println!("Unsafe: {val}");
    }

    // String from raw bytes — must be valid UTF-8
    let bytes = vec![72, 101, 108, 108, 111];
    unsafe {
        let s = String::from_utf8_unchecked(bytes);
        println!("{s}");  // Hello
    }
}
```

---

## 5. Creating Safe Abstractions

The most important pattern — wrap unsafe code in a safe API:

```rust
/// Splits a mutable slice at index `mid` into two non-overlapping slices.
///
/// This is how `split_at_mut` works internally.
fn split_at_mut_manual<T>(slice: &mut [T], mid: usize) -> (&mut [T], &mut [T]) {
    let len = slice.len();
    assert!(mid <= len, "mid out of bounds");

    let ptr = slice.as_mut_ptr();

    unsafe {
        (
            // First half: [0..mid]
            std::slice::from_raw_parts_mut(ptr, mid),
            // Second half: [mid..len]
            std::slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
    // SAFE because:
    // 1. Both slices come from the same allocation
    // 2. They don't overlap (disjoint ranges)
    // 3. The original slice is valid for the full length
}

fn main() {
    let mut v = vec![1, 2, 3, 4, 5, 6];
    let (left, right) = split_at_mut_manual(&mut v, 3);

    left[0] = 10;
    right[0] = 40;

    println!("Left: {:?}", left);    // [10, 2, 3]
    println!("Right: {:?}", right);  // [40, 5, 6]
}
```

### Another example — a safe wrapper:

```rust
struct SafeBuffer {
    data: Vec<u8>,
}

impl SafeBuffer {
    fn new(size: usize) -> Self {
        SafeBuffer { data: vec![0; size] }
    }

    /// Fast read without bounds check — bounds verified by assertion
    fn read_fast(&self, index: usize) -> u8 {
        assert!(index < self.data.len(), "index out of bounds");
        unsafe { *self.data.get_unchecked(index) }
    }

    /// Fast write without bounds check — bounds verified by assertion
    fn write_fast(&mut self, index: usize, value: u8) {
        assert!(index < self.data.len(), "index out of bounds");
        unsafe { *self.data.get_unchecked_mut(index) = value; }
    }
}

fn main() {
    let mut buf = SafeBuffer::new(10);
    buf.write_fast(3, 42);
    println!("Read: {}", buf.read_fast(3));  // 42
}
```

---

## 6. Mutable Static Variables

Global mutable state requires `unsafe`:

```rust
static mut COUNTER: u32 = 0;

fn increment() {
    unsafe {
        COUNTER += 1;
    }
}

fn get_count() -> u32 {
    unsafe { COUNTER }
}

fn main() {
    increment();
    increment();
    increment();
    println!("Count: {}", get_count());  // 3
}
```

**Why unsafe?** Multiple threads could access `COUNTER` simultaneously → data race. **Prefer `AtomicU32` or `Mutex` instead.**

```rust
use std::sync::atomic::{AtomicU32, Ordering};

static SAFE_COUNTER: AtomicU32 = AtomicU32::new(0);

fn main() {
    SAFE_COUNTER.fetch_add(1, Ordering::Relaxed);
    SAFE_COUNTER.fetch_add(1, Ordering::Relaxed);
    println!("Safe count: {}", SAFE_COUNTER.load(Ordering::Relaxed));
}
```

---

## 7. Unsafe Traits

Implementing a trait can be `unsafe` when the compiler can't verify correctness:

```rust
/// Marker: this type can be safely zeroed.
unsafe trait Zeroable {
    fn zeroed() -> Self;
}

// i32 can be safely zeroed (all-zeros is valid)
unsafe impl Zeroable for i32 {
    fn zeroed() -> Self { 0 }
}

unsafe impl Zeroable for f64 {
    fn zeroed() -> Self { 0.0 }
}

// String CANNOT be safely zeroed (would corrupt internal pointer)
// unsafe impl Zeroable for String { ... }  // ❌ DON'T

fn make_zero<T: Zeroable>() -> T {
    T::zeroed()
}

fn main() {
    let x: i32 = make_zero();
    let y: f64 = make_zero();
    println!("{x}, {y}");  // 0, 0
}
```

---

## 8. Union Types

Unions share memory between fields (like C unions):

```rust
#[repr(C)]
union IntOrFloat {
    i: i32,
    f: f32,
}

fn main() {
    let mut v = IntOrFloat { i: 42 };

    // Reading union fields is unsafe — compiler can't know which is valid
    unsafe {
        println!("As int: {}", v.i);    // 42
        println!("As float: {}", v.f);  // some float (reinterpreted bytes)
    }

    v.f = 3.14;
    unsafe {
        println!("As float: {}", v.f);  // 3.14
        println!("As int: {}", v.i);    // some integer (reinterpreted bytes)
    }
}
```

---

## 9. Guidelines for Unsafe Code

### ✅ DO:

```rust
// 1. Minimize unsafe blocks — keep them as small as possible
unsafe { *ptr }  // just the dereference

// 2. Document safety invariants
/// # Safety
/// `ptr` must be non-null and point to a valid `i32`.
unsafe fn read_ptr(ptr: *const i32) -> i32 { *ptr }

// 3. Wrap in safe abstractions
fn safe_read(slice: &[i32], index: usize) -> i32 {
    assert!(index < slice.len());
    unsafe { *slice.get_unchecked(index) }
}

// 4. Use #[deny(unsafe_op_in_unsafe_fn)] for extra safety
#[deny(unsafe_op_in_unsafe_fn)]
unsafe fn careful(ptr: *const i32) -> i32 {
    unsafe { *ptr }  // must still use unsafe block inside unsafe fn
}
```

### ❌ DON'T:

```rust
// 1. Don't use unsafe to bypass the borrow checker
//    → fix the design instead

// 2. Don't use unsafe for performance without benchmarking
//    → safe code is often just as fast

// 3. Don't forget to document Safety sections

// 4. Don't assume raw pointers are valid
```

---

## 10. Summary Cheat Sheet

```
FIVE SUPERPOWERS
────────────────────────────────────────────────────────────
1. Dereference raw pointers     *ptr
2. Call unsafe functions         unsafe fn f() { }
3. Mutable static variables     static mut X: T
4. Implement unsafe traits       unsafe impl Send
5. Access union fields           union.field

RAW POINTERS
────────────────────────────────────────────────────────────
*const T     immutable raw pointer
*mut T       mutable raw pointer
Creating: safe    &val as *const T
Deref: unsafe     unsafe { *ptr }

SAFE ABSTRACTIONS
────────────────────────────────────────────────────────────
fn safe_api() {          // safe public API
    assert!(invariants);
    unsafe { ... }       // minimal unsafe block
}

GUIDELINES
────────────────────────────────────────────────────────────
Minimize unsafe scope
Document /// # Safety invariants
Wrap in safe APIs
Prefer Atomic/Mutex over static mut
```

---

## What's Next?

**Lesson 62 — Declarative Macros** — Write code that writes code. Master `macro_rules!` for reducing boilerplate and building DSLs.

## Further Reading
- [The Rust Book — Ch 19.1: Unsafe](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html)
- [Rustonomicon — Unsafe](https://doc.rust-lang.org/nomicon/)

---

*Unsafe: with great power comes great responsibility! 🦀*
