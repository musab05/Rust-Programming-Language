# 📘 Lesson 86 — Dynamically Sized Types (AT4)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** AT4 · Category: 🔷 Advanced Types  
> **Previous:** [Lesson 85 — Never Type (!)](../lesson_85_never_type/lesson_85_never_type.md)  
> **Next:** [Lesson 87 — PhantomData](../lesson_87_phantomdata/lesson_87_phantomdata.md)  
> **Practice:** [Questions](./lesson_86_questions.md) · [Answers](./lesson_86_answers.md)  
> **Practice Task:** Explain why `fn f(s: str)` won't compile; fix it

---

## Table of Contents

1. [What Are DSTs?](#1-what-are-dsts)
2. [str — The String Slice DST](#2-str--the-string-slice-dst)
3. [\[T\] — The Slice DST](#3-t--the-slice-dst)
4. [dyn Trait — The Trait Object DST](#4-dyn-trait--the-trait-object-dst)
5. [The Sized Trait](#5-the-sized-trait)
6. [?Sized Bound](#6-sized-bound)
7. [Fat Pointers](#7-fat-pointers)
8. [DSTs in Structs](#8-dsts-in-structs)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Are DSTs?

**Dynamically Sized Types** have a size known only at runtime. They can't live on the stack — they must always be behind a pointer:

```rust
// ❌ These won't compile — size unknown at compile time
// let s: str = "hello";         // how many bytes?
// let a: [i32] = [1, 2, 3];    // how many elements?
// let d: dyn Display = 42;     // which concrete type?

// ✅ Always use behind a pointer
let s: &str = "hello";                    // known: pointer + length
let a: &[i32] = &[1, 2, 3];              // known: pointer + length
let d: &dyn std::fmt::Display = &42;     // known: pointer + vtable
let b: Box<str> = "hello".into();        // known: pointer + length
```

---

## 2. str — The String Slice DST

`str` (without `&`) is a DST — a sequence of UTF-8 bytes with unknown length:

```rust
fn main() {
    // str is a DST — always used as &str, Box<str>, or Rc<str>
    let greeting: &str = "hello";   // fat pointer: ptr + len
    println!("size of &str:   {} bytes", std::mem::size_of::<&str>());      // 16
    println!("size of &u8:    {} bytes", std::mem::size_of::<&u8>());       // 8
    println!("size of String: {} bytes", std::mem::size_of::<String>());    // 24

    // &str is a fat pointer: 8 bytes pointer + 8 bytes length = 16 bytes
    // String is: 8 bytes pointer + 8 bytes length + 8 bytes capacity = 24 bytes
}
```

### Why not `str` directly?

```rust
// fn takes_str(s: str) { }  // ❌ str is not Sized — can't be a parameter

fn takes_str_ref(s: &str) { println!("{s}"); }       // ✅ &str is Sized (16 bytes)
fn takes_string(s: String) { println!("{s}"); }       // ✅ String is Sized (24 bytes)
fn takes_box_str(s: Box<str>) { println!("{s}"); }    // ✅ Box<str> is Sized (16 bytes)
```

---

## 3. [T] — The Slice DST

`[T]` is a contiguous sequence of `T` with unknown length:

```rust
fn main() {
    let arr: [i32; 5] = [1, 2, 3, 4, 5];  // sized! exactly 20 bytes
    let slice: &[i32] = &arr;              // fat pointer: ptr + len

    println!("size of [i32; 5]: {} bytes", std::mem::size_of::<[i32; 5]>());  // 20
    println!("size of &[i32]:   {} bytes", std::mem::size_of::<&[i32]>());    // 16

    // Can't have [i32] on the stack — size unknown
    // let s: [i32] = ???;  // ❌

    // Always behind a pointer
    let boxed: Box<[i32]> = vec![1, 2, 3].into_boxed_slice();
    println!("boxed: {:?}", boxed);
}
```

### Array vs Slice:

```rust
fn main() {
    // [i32; 3] — Sized! Exactly 12 bytes. Lives on stack.
    let array: [i32; 3] = [1, 2, 3];

    // [i32] — DST! Unknown length. Must be behind pointer.
    let slice: &[i32] = &array;

    // Functions usually accept slices — more flexible
    fn sum(data: &[i32]) -> i32 { data.iter().sum() }
    println!("{}", sum(&array));   // array coerces to slice
    println!("{}", sum(slice));
}
```

---

## 4. dyn Trait — The Trait Object DST

`dyn Trait` is a DST — the concrete type is unknown:

```rust
use std::fmt::Display;

fn main() {
    // dyn Display is a DST
    // let x: dyn Display = 42;  // ❌ size unknown

    // Always behind a pointer
    let x: &dyn Display = &42;           // fat pointer: data ptr + vtable ptr
    let y: Box<dyn Display> = Box::new("hello");

    println!("size of &dyn Display:     {} bytes", std::mem::size_of::<&dyn Display>());      // 16
    println!("size of Box<dyn Display>: {} bytes", std::mem::size_of::<Box<dyn Display>>());   // 16
    println!("size of &i32:             {} bytes", std::mem::size_of::<&i32>());                // 8

    println!("{x} and {y}");
}
```

---

## 5. The Sized Trait

`Sized` is an **auto-trait** — types with a known compile-time size implement it automatically:

```rust
fn sized_only<T: Sized>(x: T) {
    println!("size: {}", std::mem::size_of::<T>());
}

// All generic parameters are implicitly Sized:
// fn foo<T>(x: T) is really fn foo<T: Sized>(x: T)

fn main() {
    sized_only(42_i32);       // ✅ i32 is Sized
    sized_only("hello".to_string()); // ✅ String is Sized
    // sized_only(*"hello");  // ❌ str is not Sized
}
```

### Types that are NOT Sized:

| Type | Why | Use As |
|---|---|---|
| `str` | Unknown byte length | `&str`, `Box<str>` |
| `[T]` | Unknown element count | `&[T]`, `Box<[T]>` |
| `dyn Trait` | Unknown concrete type | `&dyn Trait`, `Box<dyn Trait>` |

---

## 6. ?Sized Bound

`?Sized` **removes** the implicit `Sized` requirement, allowing DSTs:

```rust
use std::fmt::Display;

// T: Sized (implicit) — only accepts sized types
fn print_sized<T: Display>(val: &T) {
    println!("{val}");
}

// T: ?Sized — accepts BOTH sized AND unsized types
fn print_maybe_unsized<T: Display + ?Sized>(val: &T) {
    println!("{val}");
}

fn main() {
    let s = String::from("hello");
    let slice: &str = "world";

    print_sized(&s);         // ✅ String is Sized
    // print_sized(slice);   // ❌ str is not Sized — T would need to be str

    print_maybe_unsized(&s);     // ✅ T = String (Sized)
    print_maybe_unsized(slice);  // ✅ T = str (!Sized, but ?Sized allows it)
}
```

### Real examples in std:

```rust
// Box::new requires Sized
impl<T> Box<T> { pub fn new(x: T) -> Box<T> { ... } }

// But Box<T> itself works with ?Sized
impl<T: ?Sized> Box<T> { /* deref, display, etc. */ }

// std::borrow::Borrow uses ?Sized
pub trait Borrow<Borrowed: ?Sized> {
    fn borrow(&self) -> &Borrowed;
}

// HashMap::get uses ?Sized for the key lookup
impl<K, V> HashMap<K, V> {
    pub fn get<Q: ?Sized>(&self, k: &Q) -> Option<&V>
    where K: Borrow<Q>, Q: Hash + Eq { ... }
}
```

---

## 7. Fat Pointers

DSTs use **fat pointers** — wider than regular pointers:

```rust
fn main() {
    // Regular pointer: 8 bytes (data pointer only)
    println!("&i32:         {} bytes", std::mem::size_of::<&i32>());         // 8
    println!("&String:      {} bytes", std::mem::size_of::<&String>());      // 8
    println!("&[i32; 5]:    {} bytes", std::mem::size_of::<&[i32; 5]>());    // 8

    // Fat pointer: 16 bytes (data pointer + metadata)
    println!("&str:         {} bytes", std::mem::size_of::<&str>());         // 16 (ptr + len)
    println!("&[i32]:       {} bytes", std::mem::size_of::<&[i32]>());       // 16 (ptr + len)
    println!("&dyn Display: {} bytes", std::mem::size_of::<&dyn std::fmt::Display>()); // 16 (ptr + vtable)
}
```

```
Thin pointer (&T where T: Sized):
┌──────────────┐
│ data pointer │  8 bytes
└──────────────┘

Fat pointer (&str, &[T]):
┌──────────────┐
│ data pointer │  8 bytes
├──────────────┤
│ length       │  8 bytes
└──────────────┘

Fat pointer (&dyn Trait):
┌──────────────┐
│ data pointer │  8 bytes
├──────────────┤
│ vtable ptr   │  8 bytes
└──────────────┘
```

---

## 8. DSTs in Structs

A struct can have ONE DST field — but it must be the **last** field:

```rust
struct MySlice {
    label: String,
    data: [u8],    // DST — must be last field
}

// Can't create MySlice directly — must use references
fn use_slice(s: &MySlice) {
    println!("{}: {:?}", s.label, &s.data);
}

// More practical: use generics with ?Sized
struct Wrapper<T: ?Sized> {
    metadata: u32,
    inner: T,        // can be sized OR unsized
}

fn main() {
    // Sized version
    let w: Wrapper<[i32; 3]> = Wrapper { metadata: 1, inner: [10, 20, 30] };
    println!("{}: {:?}", w.metadata, w.inner);

    // Can coerce to unsized
    let w_ref: &Wrapper<[i32]> = &w;
    println!("{}: {:?}", w_ref.metadata, &w_ref.inner);
}
```

---

## 9. Summary Cheat Sheet

```
DYNAMICALLY SIZED TYPES
────────────────────────────────────────────────────────────
str          UTF-8 bytes, unknown length
[T]          elements, unknown count
dyn Trait    concrete type unknown

ALWAYS BEHIND A POINTER
────────────────────────────────────────────────────────────
&str, &[T], &dyn Trait             borrowed fat pointer
Box<str>, Box<[T]>, Box<dyn Trait> owned fat pointer
Rc<str>, Arc<dyn Trait>            shared fat pointer

SIZED TRAIT
────────────────────────────────────────────────────────────
T            implicitly T: Sized
T: ?Sized    allows both sized AND unsized

FAT POINTERS
────────────────────────────────────────────────────────────
&T (Sized)      → 8 bytes  (data ptr)
&str, &[T]      → 16 bytes (data ptr + length)
&dyn Trait       → 16 bytes (data ptr + vtable ptr)

KEY FIX
────────────────────────────────────────────────────────────
fn f(s: str) {}      ❌ str not Sized
fn f(s: &str) {}     ✅ &str is Sized (fat pointer)
fn f(s: String) {}   ✅ String is Sized (owns heap data)
```

---

## What's Next?

**Lesson 87 — PhantomData** — Zero-sized marker fields for variance, unused type parameters, and type-safe units.

## Further Reading
- [The Rust Book — Ch 19.4: DSTs](https://doc.rust-lang.org/book/ch19-04-advanced-types.html#dynamically-sized-types-and-the-sized-trait)
- [Rust Reference — DSTs](https://doc.rust-lang.org/reference/dynamically-sized-types.html)

---

*DSTs: types without known sizes — always behind a pointer! 🦀*
