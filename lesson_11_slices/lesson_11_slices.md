# 📘 Lesson 11 — Slices (O4)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** O4 · Category: 🧠 Ownership  
> **Previous:** [Lesson 10 — References & Borrowing](./lesson_10_references.md)  
> **Next:** [Lesson 12 — Structs](./lesson_12_structs.md)  
> **Practice:** [Questions](./lesson_11_questions.md) · [Answers](./lesson_11_answers.md)

---

## Table of Contents

1. [What Is a Slice?](#1-what-is-a-slice)
2. [String Slices — `&str`](#2-string-slices--str)
3. [Slice Range Syntax](#3-slice-range-syntax)
4. [String Slices vs Owned Strings](#4-string-slices-vs-owned-strings)
5. [Array Slices — `&[T]`](#5-array-slices--t)
6. [Slices as Function Parameters](#6-slices-as-function-parameters)
7. [Mutable Slices](#7-mutable-slices)
8. [How Slices Work Internally (Fat Pointers)](#8-how-slices-work-internally-fat-pointers)
9. [Unicode Safety with String Slices](#9-unicode-safety-with-string-slices)
10. [Useful Slice Methods](#10-useful-slice-methods)
11. [Common Patterns & Idioms](#11-common-patterns--idioms)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. What Is a Slice?

A **slice** is a reference to a **contiguous sequence** of elements inside a collection. It borrows part (or all) of a `String`, array, or `Vec` — without copying anything.

```
Full String: [ h ][ e ][ l ][ l ][ o ][ _ ][ w ][ o ][ r ][ l ][ d ]
                                                   ↑               ↑
Slice &s[6..]:                                  [ w ][ o ][ r ][ l ][ d ]
```

Key facts:
- A slice is a **reference** — it borrows, never owns
- It knows the **start address** and the **length** of what it covers
- Two main flavours: `&str` (string slice) and `&[T]` (array/vec slice)
- Slices are **dynamically sized** — size known at runtime, not compile time

---

## 2. String Slices — `&str`

A `&str` is a reference to a piece of UTF-8 string data, either inside a `String` on the heap or in the program binary.

```rust
fn main() {
    let s = String::from("hello world");

    let hello: &str = &s[0..5];   // bytes 0,1,2,3,4
    let world: &str = &s[6..11];  // bytes 6,7,8,9,10

    println!("{hello}"); // hello
    println!("{world}"); // world
    println!("{s}");     // hello world — original untouched
}
```

String **literals** are also `&str` — they are slices baked into the program binary with `'static` lifetime:

```rust
fn main() {
    let s: &str = "hello world"; // points into the binary, not heap
    println!("{s}");
}
```

### Memory picture

```
String s (on heap):
  ptr ──────► [ h ][ e ][ l ][ l ][ o ][ _ ][ w ][ o ][ r ][ l ][ d ]
  len: 11
  cap: 11

&str "world" slice:
  ptr ──────────────────────────────► [ w ][ o ][ r ][ l ][ d ]
  len: 5
```

A `&str` is just a **pointer + length** into existing UTF-8 data. No heap allocation.

---

## 3. Slice Range Syntax

| Syntax | Meaning |
|--------|---------|
| `&s[0..5]` | bytes 0, 1, 2, 3, 4 (end exclusive) |
| `&s[0..=4]` | bytes 0, 1, 2, 3, 4 (end inclusive) |
| `&s[..5]` | from start to byte 4 |
| `&s[6..]` | from byte 6 to the end |
| `&s[..]` | the whole string |

```rust
fn main() {
    let s = String::from("hello world");

    println!("{}", &s[..5]);    // hello
    println!("{}", &s[6..]);    // world
    println!("{}", &s[..]);     // hello world
    println!("{}", &s[0..=4]);  // hello
}
```

---

## 4. String Slices vs Owned Strings

| | `String` | `&str` |
|---|---|---|
| Ownership | Owns the data | Borrows the data |
| Heap allocation | Yes | No |
| Mutable? | Yes (`mut`) | No |
| Can grow? | Yes (`push_str`) | No |
| Use in struct | Yes | Yes (with lifetime `'a`) |
| Best for function params | When you need to own/modify | When you only need to read |

### The golden rule for function parameters

**Always prefer `&str` over `&String`** when a function only needs to read string data. `&str` is more general — it accepts both `String` and literal `&str`:

```rust
// ❌ Less flexible — only accepts &String
fn bad(s: &String) -> usize {
    s.len()
}

// ✅ Better — accepts &String, &str, and string literals
fn good(s: &str) -> usize {
    s.len()
}

fn main() {
    let owned = String::from("hello");
    let literal = "world";

    good(&owned);   // &String coerces to &str ✅
    good(literal);  // &str directly ✅
    bad(&owned);    // works
    // bad(literal);  ❌ literal is &str, not &String
}
```

This automatic coercion from `&String` → `&str` is called **deref coercion**.

---

## 5. Array Slices — `&[T]`

Just as `&str` is a slice of characters, `&[T]` is a slice of any type `T`:

```rust
fn main() {
    let arr = [10, 20, 30, 40, 50];
    let slice: &[i32] = &arr[1..4]; // [20, 30, 40]
    println!("{:?}", slice);
    println!("len = {}", slice.len()); // 3

    let v = vec![1, 2, 3, 4, 5];
    let mid: &[i32] = &v[1..4]; // [2, 3, 4]
    println!("{:?}", mid);
}
```

Full slice of a whole array or Vec:

```rust
fn main() {
    let arr = [1, 2, 3];
    let full: &[i32] = &arr;      // same as &arr[..]
    let v = vec![4, 5, 6];
    let vfull: &[i32] = &v;       // same as &v[..]
    println!("{:?} {:?}", full, vfull);
}
```

---

## 6. Slices as Function Parameters

Taking slices in functions is the idiomatic way to accept arrays, Vecs, and sub-ranges uniformly:

```rust
fn sum(data: &[i32]) -> i32 {
    let mut total = 0;
    for &x in data {
        total += x;
    }
    total
}

fn main() {
    let arr = [1, 2, 3, 4, 5];
    let v   = vec![10, 20, 30];

    println!("{}", sum(&arr));        // &[i32; 5] → &[i32]
    println!("{}", sum(&v));          // &Vec<i32> → &[i32]
    println!("{}", sum(&arr[1..4]));  // sub-slice
    println!("{}", sum(&[100, 200])); // temporary array literal
}
```

### Prefer `&[T]` over `&Vec<T>` for the same reason as `&str` over `&String`

```rust
// ❌ Inflexible
fn bad_sum(v: &Vec<i32>) -> i32 { v.iter().sum() }

// ✅ Accepts arrays, Vecs, and sub-slices
fn good_sum(v: &[i32]) -> i32 { v.iter().sum() }
```

---

## 7. Mutable Slices

A `&mut [T]` lets you modify elements in place:

```rust
fn double_all(data: &mut [i32]) {
    for x in data.iter_mut() {
        *x *= 2;
    }
}

fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    double_all(&mut v);
    println!("{:?}", v); // [2, 4, 6, 8, 10]

    // Works on sub-slices too
    double_all(&mut v[1..4]);
    println!("{:?}", v); // [2, 8, 12, 16, 10]
}
```

Mutable slice rules follow normal borrow rules — only one `&mut` slice at a time:

```rust
fn main() {
    let mut arr = [1, 2, 3, 4, 5];
    // Use split_at_mut for two non-overlapping mutable slices:
    let (left, right) = arr.split_at_mut(3);
    left[0] = 100;   // arr[0]
    right[0] = 400;  // arr[3]
    println!("{:?}", arr); // [100, 2, 3, 400, 5]
}
```

---

## 8. How Slices Work Internally (Fat Pointers)

A slice reference is a **fat pointer** — two machine words instead of one:

```
&[i32] (fat pointer):
┌──────────────────┐
│  ptr  ──────────►│  actual data in memory
│  len: 3          │
└──────────────────┘
```

Regular references (`&i32`) are thin pointers — just an address. Slices need the length because the compiler doesn't know the slice size at compile time.

```rust
fn main() {
    use std::mem::size_of;
    println!("&i32:  {} bytes", size_of::<&i32>());   // 8 (one pointer)
    println!("&[i32]: {} bytes", size_of::<&[i32]>()); // 16 (pointer + length)
    println!("&str:   {} bytes", size_of::<&str>());   // 16 (pointer + length)
}
```

This is why you always use slices behind a reference — `[T]` and `str` are **unsized types** and can't exist on the stack directly.

---

## 9. Unicode Safety with String Slices

⚠️ String slices use **byte indices**, not character indices. Multi-byte UTF-8 characters span multiple bytes — slicing at the wrong byte boundary causes a **runtime panic**:

```rust
fn main() {
    let s = "hello";          // ASCII — each char is 1 byte
    println!("{}", &s[0..3]); // "hel" — safe

    let emoji = "😀hi";       // 😀 = 4 bytes in UTF-8
    // println!("{}", &emoji[0..2]); // PANIC — splits the emoji!
    println!("{}", &emoji[0..4]); // "😀" — correct
    println!("{}", &emoji[4..]);  // "hi"
}
```

### Safe character-level access

```rust
fn main() {
    let s = "café";  // 'é' = 2 bytes

    // Safe: iterate characters
    for c in s.chars() { print!("{c} "); }
    println!();

    // Safe: get Nth character
    let third = s.chars().nth(2);
    println!("{:?}", third); // Some('f')

    // Safe: find byte position then slice
    let pos = s.find('é').unwrap();
    println!("{}", &s[pos..]); // "é"

    // Check if a byte index is a valid char boundary
    println!("{}", s.is_char_boundary(3)); // might be false for 'é'
    println!("{}", s.is_char_boundary(4)); // true for 'é' (it's 2 bytes)
}
```

---

## 10. Useful Slice Methods

### Inspection

```rust
let s = [10, 20, 30, 40, 50];

println!("{}", s.len());           // 5
println!("{}", s.is_empty());      // false
println!("{:?}", s.first());       // Some(10)
println!("{:?}", s.last());        // Some(50)
println!("{:?}", s.get(2));        // Some(30) — safe, returns Option
println!("{:?}", s.get(99));       // None — no panic
```

### Searching

```rust
let s = [1, 3, 5, 7, 9];
println!("{}", s.contains(&5));    // true
println!("{:?}", s.iter().position(|&x| x == 5)); // Some(2)
```

### Sorting (requires `&mut`)

```rust
let mut v = vec![3, 1, 4, 1, 5, 9, 2, 6];
v.sort();
println!("{:?}", v);               // [1, 1, 2, 3, 4, 5, 6, 9]
v.sort_by(|a, b| b.cmp(a));       // descending
println!("{:?}", v);
v.sort_by_key(|&x| x % 3);        // sort by x mod 3
```

### Windows and Chunks

```rust
let data = [1, 2, 3, 4, 5];

// windows: overlapping sub-slices of size n
for w in data.windows(3) {
    print!("{w:?} ");  // [1,2,3] [2,3,4] [3,4,5]
}
println!();

// chunks: non-overlapping sub-slices of up to size n
for c in data.chunks(2) {
    print!("{c:?} ");  // [1,2] [3,4] [5]
}
```

### Binary search (slice must be sorted)

```rust
let sorted = [1, 3, 5, 7, 9, 11];
println!("{:?}", sorted.binary_search(&7));   // Ok(3)
println!("{:?}", sorted.binary_search(&4));   // Err(2) — where it would go
```

### Splitting

```rust
let data = [1, 2, 3, 4, 5];
let (left, right) = data.split_at(3);
println!("{left:?} {right:?}"); // [1,2,3]  [4,5]

// Split on a predicate
let parts: Vec<&[i32]> = data.split(|&x| x % 2 == 0).collect();
println!("{parts:?}"); // [[1], [3], [5]]
```

---

## 11. Common Patterns & Idioms

### The classic `first_word` function

```rust
fn first_word(s: &str) -> &str {
    match s.find(' ') {
        Some(i) => &s[..i],
        None    => s,          // whole string if no space
    }
}

fn main() {
    let sentence = String::from("hello world");
    let word = first_word(&sentence);
    println!("{word}"); // hello
    // sentence.clear(); // ❌ would break 'word' — compiler stops this
    println!("{sentence}");
}
```

### Returning slices — borrow checker ensures safety

```rust
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str {
    if a.len() >= b.len() { a } else { b }
}

fn main() {
    let s1 = String::from("long string");
    let s2 = "short";
    println!("{}", longest(&s1, s2)); // long string
}
```

### Pattern matching on slices

```rust
fn describe(data: &[i32]) {
    match data {
        []        => println!("empty"),
        [x]       => println!("one: {x}"),
        [x, y]    => println!("two: {x}, {y}"),
        [first, .., last] => println!("many: first={first}, last={last}"),
    }
}

fn main() {
    describe(&[]);
    describe(&[42]);
    describe(&[1, 2]);
    describe(&[10, 20, 30, 40]);
}
```

### Moving average with `windows`

```rust
fn moving_avg(data: &[f64], window: usize) -> Vec<f64> {
    data.windows(window)
        .map(|w| w.iter().sum::<f64>() / w.len() as f64)
        .collect()
}

fn main() {
    let prices = [1.0, 2.0, 3.0, 4.0, 5.0];
    println!("{:?}", moving_avg(&prices, 3)); // [2.0, 3.0, 4.0]
}
```

---

## 12. Summary Cheat Sheet

```
SLICE TYPES
────────────────────────────────────────────────────────────
&str          String slice — pointer + length into UTF-8 data
&[T]          Array/Vec slice — pointer + length into T data
&mut [T]      Mutable slice — can modify elements

CREATING SLICES
────────────────────────────────────────────────────────────
&s[0..5]      bytes 0..4 (exclusive end)
&s[0..=4]     bytes 0..4 (inclusive end)
&s[..5]       from start to byte 4
&s[6..]       from byte 6 to end
&s[..]        entire collection

DEREF COERCIONS (happen automatically in fn calls)
────────────────────────────────────────────────────────────
&String  → &str      use &str in fn params, not &String
&Vec<T>  → &[T]      use &[T] in fn params, not &Vec<T>
&[T; N]  → &[T]

KEY METHODS
────────────────────────────────────────────────────────────
.len()              number of elements
.is_empty()         true if len == 0
.first()            Option<&T> — first element
.last()             Option<&T> — last element
.get(i)             Option<&T> — safe indexed access (no panic)
.contains(&x)       bool — linear search
.iter()             iterator over &T
.iter_mut()         iterator over &mut T
.sort()             sort in place (needs &mut)
.binary_search(&x)  Ok(i) or Err(i) — requires sorted
.windows(n)         overlapping sub-slices of size n
.chunks(n)          non-overlapping sub-slices up to size n
.split_at(i)        (&[..i], &[i..])
.split_at_mut(i)    two mutable non-overlapping sub-slices

UNICODE SAFETY
────────────────────────────────────────────────────────────
Byte indices — slicing at wrong boundary PANICS at runtime
Use .chars() for character-level access
Use .find() to get safe byte positions
Use .is_char_boundary(i) to verify
```

---

## What's Next?

**Lesson 12 — Structs** — Define your own data types, group related fields, and add methods with `impl`.

## Further Reading
- [The Rust Book — Ch 4.3: The Slice Type](https://doc.rust-lang.org/book/ch04-03-slices.html)
- [std::slice documentation](https://doc.rust-lang.org/std/primitive.slice.html)
- [std::str documentation](https://doc.rust-lang.org/std/primitive.str.html)
