# 📘 Lesson 64 — FFI: Foreign Function Interface (FF1)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** FF1 · Category: 🌍 FFI  
> **Previous:** [Lesson 63 — Procedural Macros](../lesson_63_proc_macros/lesson_63_proc_macros.md)  
> **Next:** [Lesson 65 — Advanced Lifetimes](../lesson_65_advanced_lifetimes/lesson_65_advanced_lifetimes.md)  
> **Practice:** [Questions](./lesson_64_questions.md) · [Answers](./lesson_64_answers.md)  
> **Practice Task:** Call a C function from Rust and expose a Rust function to C

---

## Table of Contents

1. [What Is FFI?](#1-what-is-ffi)
2. [Calling C from Rust](#2-calling-c-from-rust)
3. [C Types in Rust](#3-c-types-in-rust)
4. [Strings Across the Boundary](#4-strings-across-the-boundary)
5. [Exposing Rust to C](#5-exposing-rust-to-c)
6. [Linking and Build Scripts](#6-linking-and-build-scripts)
7. [The libc Crate](#7-the-libc-crate)
8. [Safety and Wrappers](#8-safety-and-wrappers)
9. [Real-World Example: System Info](#9-real-world-example-system-info)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is FFI?

FFI lets Rust call functions written in other languages (usually C) and vice versa:

```
┌──────────┐     extern "C"     ┌──────────┐
│   Rust   │ ◄──────────────► │    C     │
│  code    │   ABI boundary    │  library │
└──────────┘                    └──────────┘
```

**Use cases:**
- Using existing C libraries (OpenSSL, SQLite, zlib)
- Embedding Rust in C/C++/Python applications
- Interfacing with OS system calls

---

## 2. Calling C from Rust

Use `extern "C"` to declare C functions:

```rust
// Declare the C function signature
extern "C" {
    fn abs(input: i32) -> i32;
    fn sqrt(x: f64) -> f64;
    fn rand() -> i32;
}

fn main() {
    unsafe {
        println!("abs(-5) = {}", abs(-5));        // 5
        println!("sqrt(16) = {}", sqrt(16.0));    // 4.0
        println!("rand() = {}", rand());          // random number
    }
}
```

**Why unsafe?** Rust can't verify:
- The function exists and matches the signature
- It doesn't cause undefined behavior
- Memory is handled correctly

---

## 3. C Types in Rust

C and Rust have different type representations:

| C Type | Rust Type | `std::ffi` Type |
|---|---|---|
| `int` | `i32` | `c_int` |
| `unsigned int` | `u32` | `c_uint` |
| `long` | `i64` (varies!) | `c_long` |
| `char` | `i8` or `u8` | `c_char` |
| `float` | `f32` | `c_float` |
| `double` | `f64` | `c_double` |
| `void*` | `*mut c_void` | `c_void` |
| `size_t` | `usize` | — |
| `const char*` | `*const c_char` | `CStr` |

```rust
use std::ffi::{c_int, c_double};

extern "C" {
    fn pow(base: c_double, exp: c_double) -> c_double;
}

fn main() {
    let result = unsafe { pow(2.0, 10.0) };
    println!("2^10 = {result}");  // 1024
}
```

### repr(C) for struct layout:

```rust
// Ensure struct has C-compatible memory layout
#[repr(C)]
struct Point {
    x: f64,
    y: f64,
}

// Now safe to pass to/from C functions expecting this layout
```

---

## 4. Strings Across the Boundary

C strings (`\0`-terminated) differ from Rust strings (length-based, UTF-8):

```rust
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

extern "C" {
    fn strlen(s: *const c_char) -> usize;
    fn puts(s: *const c_char) -> i32;
}

fn main() {
    // Rust → C: use CString
    let rust_string = "Hello from Rust!";
    let c_string = CString::new(rust_string).unwrap();

    unsafe {
        puts(c_string.as_ptr());                      // prints the string
        println!("strlen: {}", strlen(c_string.as_ptr()));  // 16
    }

    // C → Rust: use CStr
    let c_ptr: *const c_char = c_string.as_ptr();
    unsafe {
        let c_str = CStr::from_ptr(c_ptr);
        let rust_str = c_str.to_str().unwrap();
        println!("Back to Rust: {rust_str}");
    }
}
```

### String conversion table:

| Direction | Type | Use |
|---|---|---|
| Rust → C | `CString` | Owned, null-terminated |
| Rust → C (ptr) | `.as_ptr() → *const c_char` | Pass to C function |
| C → Rust | `CStr::from_ptr(ptr)` | Borrow C string |
| C → Rust (owned) | `CStr → .to_string_lossy()` | Copy to Rust String |

---

## 5. Exposing Rust to C

Make Rust functions callable from C:

```rust
/// A Rust function callable from C.
#[no_mangle]                    // preserve the function name
pub extern "C" fn rust_add(a: i32, b: i32) -> i32 {
    a + b
}

/// Expose a string processing function.
#[no_mangle]
pub extern "C" fn rust_greeting(name: *const std::os::raw::c_char) -> *mut std::os::raw::c_char {
    let c_str = unsafe { std::ffi::CStr::from_ptr(name) };
    let name = c_str.to_str().unwrap_or("unknown");
    let greeting = format!("Hello, {name}!");
    let c_greeting = std::ffi::CString::new(greeting).unwrap();
    c_greeting.into_raw()  // caller must free this!
}

/// Free a string allocated by Rust.
#[no_mangle]
pub extern "C" fn rust_free_string(s: *mut std::os::raw::c_char) {
    if !s.is_null() {
        unsafe { drop(std::ffi::CString::from_raw(s)); }
    }
}
```

### Compile as a C library:

```toml
# Cargo.toml
[lib]
crate-type = ["cdylib"]   # dynamic library (.so / .dll)
# or
crate-type = ["staticlib"] # static library (.a / .lib)
```

### C header (generated or manual):

```c
// my_lib.h
int rust_add(int a, int b);
char* rust_greeting(const char* name);
void rust_free_string(char* s);
```

---

## 6. Linking and Build Scripts

### Linking to system libraries:

```rust
// Link to libm (math library)
#[link(name = "m")]
extern "C" {
    fn sin(x: f64) -> f64;
    fn cos(x: f64) -> f64;
}
```

### Build scripts (`build.rs`):

```rust
// build.rs — compile and link C code
fn main() {
    // Compile a C file
    cc::Build::new()
        .file("src/helper.c")
        .compile("helper");

    // Tell Cargo to link a system library
    println!("cargo:rustc-link-lib=z");  // link libz
}
```

```toml
# Cargo.toml
[build-dependencies]
cc = "1"
```

---

## 7. The libc Crate

Portable C type definitions and function bindings:

```toml
[dependencies]
libc = "0.2"
```

```rust
use libc::{c_int, c_char, size_t};

extern "C" {
    fn getpid() -> c_int;
}

fn main() {
    let pid = unsafe { getpid() };
    println!("Process ID: {pid}");

    // libc also provides many bindings directly
    let pid2 = unsafe { libc::getpid() };
    println!("Process ID (via libc): {pid2}");
}
```

---

## 8. Safety and Wrappers

Always wrap unsafe FFI in safe Rust APIs:

```rust
use std::ffi::{CStr, CString};
use std::os::raw::c_char;

extern "C" {
    fn strlen(s: *const c_char) -> usize;
}

/// Safe wrapper around C's strlen.
fn safe_strlen(s: &str) -> usize {
    let c_str = CString::new(s).expect("String contains null byte");
    unsafe { strlen(c_str.as_ptr()) }
}

fn main() {
    println!("Length: {}", safe_strlen("Hello, world!"));  // 13
}
```

### Error handling across FFI:

```rust
extern "C" {
    fn fopen(filename: *const c_char, mode: *const c_char) -> *mut libc::FILE;
    fn fclose(file: *mut libc::FILE) -> c_int;
}

use std::ffi::CString;
use std::os::raw::c_char;

fn open_file(path: &str) -> Result<*mut libc::FILE, String> {
    let c_path = CString::new(path).map_err(|e| e.to_string())?;
    let c_mode = CString::new("r").unwrap();

    let file = unsafe { fopen(c_path.as_ptr(), c_mode.as_ptr()) };

    if file.is_null() {
        Err(format!("Failed to open: {path}"))
    } else {
        Ok(file)
    }
}
```

---

## 9. Real-World Example: System Info

```rust
use std::ffi::CStr;

#[cfg(target_os = "linux")]
extern "C" {
    fn gethostname(name: *mut i8, len: usize) -> i32;
}

fn get_hostname() -> String {
    let mut buf = vec![0i8; 256];

    #[cfg(target_os = "linux")]
    {
        unsafe { gethostname(buf.as_mut_ptr(), buf.len()); }
    }

    #[cfg(target_os = "windows")]
    {
        // Windows equivalent would use GetComputerNameA
        return "windows-host".to_string();
    }

    let c_str = unsafe { CStr::from_ptr(buf.as_ptr()) };
    c_str.to_string_lossy().to_string()
}

fn main() {
    println!("Hostname: {}", get_hostname());
    println!("PID: {}", std::process::id());
}
```

---

## 10. Summary Cheat Sheet

```
CALLING C FROM RUST
────────────────────────────────────────────────────────────
extern "C" {
    fn c_func(arg: c_int) -> c_int;
}
unsafe { c_func(42) }

EXPOSING RUST TO C
────────────────────────────────────────────────────────────
#[no_mangle]
pub extern "C" fn my_func(x: i32) -> i32 { x }

STRING CONVERSION
────────────────────────────────────────────────────────────
Rust → C:  CString::new("hello").as_ptr()
C → Rust:  CStr::from_ptr(ptr).to_str()

TYPE MAPPING
────────────────────────────────────────────────────────────
C int         → c_int / i32
C double      → c_double / f64
C char*       → *const c_char → CStr
C void*       → *mut c_void
C struct      → #[repr(C)] struct

STRUCT LAYOUT
────────────────────────────────────────────────────────────
#[repr(C)]   C-compatible layout (required for FFI)

LIBRARY TYPES
────────────────────────────────────────────────────────────
crate-type = ["cdylib"]    → .so / .dll (dynamic)
crate-type = ["staticlib"] → .a / .lib (static)
```

---

## What's Next?

**Lesson 65 — Advanced Lifetimes** — Higher-ranked trait bounds (HRTB), lifetime elision rules, and lifetime subtyping.

## Further Reading
- [The Rust Book — Ch 19.1: FFI](https://doc.rust-lang.org/book/ch19-01-unsafe-rust.html#using-extern-functions-to-call-external-code)
- [Rustonomicon — FFI](https://doc.rust-lang.org/nomicon/ffi.html)
- [The `libc` crate](https://docs.rs/libc/)

---

*FFI: Rust talking to the world, one extern "C" at a time! 🦀*
