# ✅ Lesson 64 — Answers: FFI (FF1)

---

## Section A

### A1
Calling C functions is unsafe because Rust **cannot verify**:
1. The function signature matches the actual C function
2. The function doesn't cause undefined behavior
3. Memory is properly managed (no double-frees, dangling pointers)
4. The function is actually linked and available

The programmer takes responsibility for these guarantees.

### A2
- **`CString`**: **Owned**, heap-allocated, null-terminated byte string. Use when creating a string in Rust to pass to C. Equivalent to `String` but for C.
- **`CStr`**: **Borrowed** reference to a null-terminated byte string. Use when receiving a `*const c_char` from C. Equivalent to `&str` but for C.

### A3
`#[no_mangle]` tells the compiler NOT to modify the function name. Without it, Rust applies "name mangling" (encoding type info into the symbol name) which would make the function impossible to find from C code.

---

## Section B

### A4
```rust
use std::ffi::CString;
use std::os::raw::c_char;

extern "C" {
    fn abs(n: i32) -> i32;
    fn pow(base: f64, exp: f64) -> f64;
    fn strlen(s: *const c_char) -> usize;
}

fn safe_abs(n: i32) -> i32 {
    unsafe { abs(n) }
}

fn safe_pow(base: f64, exp: f64) -> f64 {
    unsafe { pow(base, exp) }
}

fn safe_strlen(s: &str) -> usize {
    let c_str = CString::new(s).expect("null byte in string");
    unsafe { strlen(c_str.as_ptr()) }
}

fn main() {
    println!("abs(-42) = {}", safe_abs(-42));         // 42
    println!("pow(2, 8) = {}", safe_pow(2.0, 8.0));   // 256
    println!("strlen(\"hello\") = {}", safe_strlen("hello"));  // 5
}
```

### A5
```rust
#[no_mangle]
pub extern "C" fn rust_multiply(a: i32, b: i32) -> i32 {
    a * b
}
```

In `Cargo.toml`:
```toml
[lib]
crate-type = ["cdylib"]
```

### A6
```rust
use std::ffi::CString;
use std::os::raw::c_char;

extern "C" { fn puts(s: *const c_char) -> i32; }

fn main() {
    let rust_str = "Hello from Rust!";        // 1. Start with a Rust &str
    let c_string = CString::new(rust_str)      // 2. Convert to CString (adds \0)
        .expect("String contains null byte");
    unsafe {
        puts(c_string.as_ptr());               // 3. Get raw pointer and pass to C
    }
    // c_string is dropped here — memory freed
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `extern "C"` specifies the C ABI calling convention |
| 2 | **True** | `repr(C)` forces C-compatible struct field ordering and padding |
| 3 | **True** | C strings can't contain null bytes (it's the terminator) |
| 4 | **False** | All `extern "C"` function calls require `unsafe` blocks |
| 5 | **True** | `cdylib` produces `.so` (Linux) or `.dll` (Windows) |
| 6 | **True** | `libc` provides `c_int`, `c_char`, `c_void`, etc. |

---

## 🏆 Lesson 64 Complete!

**Next up:** [Lesson 65 — Advanced Lifetimes](../lesson_65_advanced_lifetimes/lesson_65_advanced_lifetimes.md) 🦀
