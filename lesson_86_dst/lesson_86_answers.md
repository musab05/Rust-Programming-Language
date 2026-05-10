# ✅ Lesson 86 — Answers: DSTs (AT4)

---

## Section A

### A1 — ❌ Won't compile
`str` is not `Sized` — can't be a function parameter directly. Error: "the size for values of type `str` cannot be known at compilation time".

### A2 — ✅ Compiles
`?Sized` allows `T = str`, and `&T` is a fat pointer. The string literal `"hello"` is `&str`. Output: `hello`.

---

## Section B

### A3
```rust
fn f1(s: &str) { println!("{s}"); }           // borrow
fn f2(s: String) { println!("{s}"); }         // owned
fn f3(s: Box<str>) { println!("{s}"); }       // boxed
// fn f4<T: AsRef<str>>(s: T) { println!("{}", s.as_ref()); }  // generic

fn main() {
    f1("hello");
    f2("hello".to_string());
    f3("hello".into());
}
```

### A4
```rust
use std::fmt::Display;
fn main() {
    println!("&i32:         {} bytes (thin pointer)", std::mem::size_of::<&i32>());
    println!("&str:         {} bytes (ptr + len)", std::mem::size_of::<&str>());
    println!("&[i32]:       {} bytes (ptr + len)", std::mem::size_of::<&[i32]>());
    println!("&dyn Display: {} bytes (ptr + vtable)", std::mem::size_of::<&dyn Display>());
    println!("Box<dyn D>:   {} bytes (ptr + vtable)", std::mem::size_of::<Box<dyn Display>>());
    println!("String:       {} bytes (ptr+len+cap)", std::mem::size_of::<String>());
}
// 8, 16, 16, 16, 16, 24
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `str` has unknown length at compile time |
| 2 | **True** | `fn f<T>(x: T)` is `fn f<T: Sized>(x: T)` |
| 3 | **False** | `?Sized` means "may or may not be Sized" — it relaxes the bound |
| 4 | **True** | Fat pointer: 8 bytes ptr + 8 bytes length = 16 bytes |
| 5 | **False** | `dyn Trait` is unsized — must be behind a pointer |
| 6 | **True** | Only one DST field allowed, must be last |

---

## 🏆 Lesson 86 Complete!

**Next up:** [Lesson 87 — PhantomData](../lesson_87_phantomdata/lesson_87_phantomdata.md) 🦀
