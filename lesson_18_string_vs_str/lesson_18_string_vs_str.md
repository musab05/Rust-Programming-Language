# 📘 Lesson 18 — String vs `&str` (C1)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** C1 · Category: 📚 Collections  
> **Previous:** [Lesson 17 — Tuple Structs & Unit Structs](./lesson_17_tuple_unit_structs.md)  
> **Next:** [Lesson 19 — Vec<T>](./lesson_19_vec.md)  
> **Practice:** [Questions](./lesson_18_questions.md) · [Answers](./lesson_18_answers.md)  
> **Practice Task:** Build a sentence joiner; explore `push_str` vs `+`

---

## Table of Contents

1. [Two String Types — Why?](#1-two-string-types--why)
2. [`&str` — The Borrowed String Slice](#2-str--the-borrowed-string-slice)
3. [`String` — The Owned Heap String](#3-string--the-owned-heap-string)
4. [Creating Strings](#4-creating-strings)
5. [String Growth — `push_str` and `push`](#5-string-growth--push_str-and-push)
6. [Concatenation — `+` operator and `format!`](#6-concatenation---operator-and-format)
7. [String Slicing and Indexing](#7-string-slicing-and-indexing)
8. [Iterating Over Strings](#8-iterating-over-strings)
9. [Common String Methods](#9-common-string-methods)
10. [Converting Between String and &str](#10-converting-between-string-and-str)
11. [Strings and UTF-8](#11-strings-and-utf-8)
12. [Function Parameters — Which to Accept?](#12-function-parameters--which-to-accept)
13. [Summary Cheat Sheet](#13-summary-cheat-sheet)

---

## 1. Two String Types — Why?

Rust has two main string types because of its ownership system:

| Type | Ownership | Storage | Mutable? | Use for |
|------|-----------|---------|----------|---------|
| `&str` | Borrowed | Anywhere (stack, heap, binary) | No | Reading/inspecting text |
| `String` | Owned | Heap | Yes | Building/owning text |

```rust
// &str — a view into existing text, no allocation
let greeting: &str = "Hello, world!";  // points into binary

// String — owns its text on the heap
let name: String = String::from("Alice");
```

Think of it like this:
- `&str` is a **window** — you can look through it, but you don't own the glass
- `String` is the **glass itself** — you own it and can modify it

---

## 2. `&str` — The Borrowed String Slice

A `&str` is a **fat pointer**: a memory address + a length. It borrows UTF-8 string data from somewhere:

```
&str:
┌─────────────────┐
│ ptr ────────────►  "hello world"  (anywhere in memory)
│ len: 11          │
└─────────────────┘
```

**String literals** are `&'static str` — they live in the program binary:

```rust
let s: &str = "hello";       // 'static lifetime — lives forever
let s: &'static str = "hi"; // explicit 'static annotation
```

**Slices of a String** are `&str` with the String's lifetime:

```rust
let owned = String::from("hello world");
let hello: &str = &owned[0..5];  // borrows from owned
let world: &str = &owned[6..];
```

---

## 3. `String` — The Owned Heap String

`String` is a growable, heap-allocated UTF-8 string:

```
String:
┌──────────────────────────────────┐
│ ptr ──────►  [h][e][l][l][o]     │  (heap)
│ len: 5    │                       │
│ cap: 8    │                       │
└──────────────────────────────────┘
```

Three components:
- **ptr** — pointer to heap-allocated UTF-8 data
- **len** — how many bytes are currently in use
- **cap** — how many bytes are allocated (capacity ≥ len)

When you grow a `String` beyond its capacity, Rust reallocates on the heap.

---

## 4. Creating Strings

```rust
fn main() {
    // From a literal
    let s1 = String::from("hello");
    let s2 = "hello".to_string();
    let s3 = "hello".to_owned();   // same as to_string()

    // Empty string
    let mut s4 = String::new();

    // With capacity pre-allocated (avoids reallocations)
    let s5 = String::with_capacity(50);
    println!("{} {} {} {}", s1, s2, s3, s4.len());

    // format! macro — builds a String from pieces
    let s6 = format!("Hello, {}! You are {} years old.", "Alice", 30);
    println!("{s6}");
}
```

---

## 5. String Growth — `push_str` and `push`

```rust
fn main() {
    let mut s = String::from("hello");

    // push_str: append a &str
    s.push_str(", world");
    println!("{s}");  // hello, world

    // push: append a single char
    s.push('!');
    println!("{s}");  // hello, world!

    // check capacity management
    let mut s2 = String::with_capacity(10);
    println!("len={} cap={}", s2.len(), s2.capacity()); // 0, 10
    s2.push_str("hello");
    println!("len={} cap={}", s2.len(), s2.capacity()); // 5, 10
    s2.push_str(" world!");
    println!("len={} cap={}", s2.len(), s2.capacity()); // 12, 20 (reallocated)
}
```

`push_str` doesn't take ownership of its argument — it borrows `&str` and copies the bytes in. The original `&str` is still usable:

```rust
fn main() {
    let mut s  = String::from("hello");
    let extra  = String::from(" world");

    s.push_str(&extra);    // &extra is &str — extra still valid
    println!("{extra}");   // ✅ extra still usable
    println!("{s}");       // hello world
}
```

---

## 6. Concatenation — `+` operator and `format!`

### The `+` operator

```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");

    let s3 = s1 + &s2;  // s1 is MOVED; s2 is borrowed

    // println!("{s1}"); // ❌ s1 was moved
    println!("{s3}");    // Hello, world!
    println!("{s2}");    // ✅ s2 still valid
}
```

The signature of `+` is roughly: `fn add(self, s: &str) -> String`. It takes ownership of the left side and borrows the right. That's why `s1` is moved and `s2` stays valid.

This is efficient — `s1`'s buffer is reused if it has capacity.

### Chaining `+` — gets ugly fast

```rust
let s1 = String::from("tic");
let s2 = String::from("tac");
let s3 = String::from("toe");

let result = s1 + "-" + &s2 + "-" + &s3;
println!("{result}"); // tic-tac-toe
// s1 moved, s2 and s3 borrowed
```

### `format!` — cleaner, no ownership issues

```rust
fn main() {
    let s1 = String::from("tic");
    let s2 = String::from("tac");
    let s3 = String::from("toe");

    let result = format!("{s1}-{s2}-{s3}");  // borrows all — nothing moved
    println!("{result}");  // tic-tac-toe
    println!("{s1} {s2} {s3}"); // ✅ all still valid
}
```

**Prefer `format!`** when building strings from many pieces — it's clearer and doesn't consume anything.

---

## 7. String Slicing and Indexing

### Why `s[0]` doesn't work

Rust strings are UTF-8. Characters are 1–4 bytes wide. Indexing by byte position could produce an invalid character:

```rust
let s = String::from("hello");
// let c = s[0]; // ❌ COMPILE ERROR: string cannot be indexed by integer
```

### Byte-range slicing (be careful with multi-byte chars!)

```rust
let s = String::from("hello");
let h = &s[0..1];  // "h" — safe, ASCII
println!("{h}");

let emoji = String::from("😀hi");
// let bad = &emoji[0..2]; // RUNTIME PANIC — inside the 4-byte emoji
let ok = &emoji[0..4];     // "😀" — correct byte boundary
```

### Safe character access

```rust
fn main() {
    let s = String::from("hello");

    // .chars() — iterate Unicode scalar values
    for c in s.chars() { print!("{c} "); }
    println!();

    // nth character (O(n) — has to scan from the start)
    let third = s.chars().nth(2);
    println!("{:?}", third); // Some('l')

    // .bytes() — raw bytes
    for b in s.bytes() { print!("{b} "); }
    println!();
}
```

---

## 8. Iterating Over Strings

```rust
fn main() {
    let s = String::from("hello world");

    // By word
    for word in s.split_whitespace() {
        println!("{word}");
    }

    // By char
    let uppercase_count = s.chars().filter(|c| c.is_uppercase()).count();
    println!("uppercase: {uppercase_count}");

    // Collect chars back to String
    let reversed: String = s.chars().rev().collect();
    println!("{reversed}");

    // By line
    let multiline = "line one\nline two\nline three";
    for line in multiline.lines() {
        println!("→ {line}");
    }
}
```

---

## 9. Common String Methods

```rust
fn main() {
    let s = String::from("  Hello, World!  ");

    // Trimming
    println!("{}", s.trim());            // "Hello, World!"
    println!("{}", s.trim_start());      // "Hello, World!  "
    println!("{}", s.trim_end());        // "  Hello, World!"

    // Case
    println!("{}", s.to_uppercase());
    println!("{}", s.to_lowercase());

    // Searching
    println!("{}", s.contains("World")); // true
    println!("{:?}", s.find("World"));   // Some(9) — byte index
    println!("{}", s.starts_with("  H")); // true

    // Replacing
    let replaced = s.replace("World", "Rust");
    println!("{replaced}");

    // Splitting
    let csv = "a,b,c,d";
    let parts: Vec<&str> = csv.split(',').collect();
    println!("{:?}", parts); // ["a", "b", "c", "d"]

    // Lengths
    let s2 = String::from("café");
    println!("bytes: {}", s2.len());         // 5 (é = 2 bytes)
    println!("chars: {}", s2.chars().count()); // 4
}
```

### Roadmap Practice: sentence joiner with `push_str`

```rust
fn join_sentences(sentences: &[&str]) -> String {
    let mut result = String::new();
    for (i, s) in sentences.iter().enumerate() {
        if i > 0 { result.push(' '); }
        result.push_str(s.trim());
        if !s.trim().ends_with('.') { result.push('.'); }
    }
    result
}

fn main() {
    let sentences = ["Hello", "World", "This is Rust."];
    println!("{}", join_sentences(&sentences));
    // Hello. World. This is Rust.
}
```

---

## 10. Converting Between String and `&str`

```rust
fn main() {
    // &str → String
    let s: &str = "hello";
    let owned1: String = s.to_string();
    let owned2: String = s.to_owned();
    let owned3: String = String::from(s);

    // String → &str
    let owned = String::from("world");
    let borrowed: &str = &owned;        // deref coercion
    let borrowed2: &str = owned.as_str(); // explicit
    let borrowed3: &str = &owned[..];   // slice syntax

    println!("{} {}", owned1, borrowed);
}
```

### Deref coercion in action

```rust
fn takes_str(s: &str) { println!("{s}"); }

fn main() {
    let owned = String::from("hello");
    takes_str(&owned);    // &String automatically coerces to &str
    takes_str("literal"); // &str directly
}
```

---

## 11. Strings and UTF-8

All Rust strings are valid UTF-8. This has three important consequences:

1. **No byte indexing** — `s[0]` is a compile error
2. **Byte length ≠ char count** — `"café".len()` = 5, `.chars().count()` = 4
3. **Runtime panics on bad byte slices** — slicing at a non-char boundary panics

```rust
fn main() {
    // Each of these is a different "character" concept:
    let s = "नमस्ते"; // Hindi: namaste

    println!("bytes: {}", s.len());          // 18 (6 chars × 3 bytes each)
    println!("chars: {}", s.chars().count()); // 6 Unicode scalar values
    // grapheme clusters would give a different count for composed chars

    // Only safe access:
    for c in s.chars() { print!("{c} "); }
    println!();
}
```

---

## 12. Function Parameters — Which to Accept?

**Always prefer `&str` over `&String` for read-only parameters:**

```rust
// ❌ Less flexible — only accepts &String or String::as_str()
fn count_words_bad(s: &String) -> usize {
    s.split_whitespace().count()
}

// ✅ Accepts &String, &str, and string literals via deref coercion
fn count_words(s: &str) -> usize {
    s.split_whitespace().count()
}

fn main() {
    let owned = String::from("hello world foo");
    let literal = "one two three";

    count_words(&owned);    // &String → &str ✅
    count_words(literal);   // &str directly ✅
    count_words_bad(&owned); // works
    // count_words_bad(literal); // ❌ literal is &str, not &String
}
```

**Decision table:**

| Situation | Parameter type |
|-----------|---------------|
| Read text, don't modify or store | `&str` |
| Need to append / modify text | `String` or `&mut String` |
| Storing text in a struct | `String` (owned) |
| Return text built from pieces | `String` |

---

## 13. Summary Cheat Sheet

```
TWO STRING TYPES
────────────────────────────────────────────────────────────
&str        Borrowed string slice — pointer + length, no heap
String      Owned heap string — pointer + length + capacity

CREATING STRINGS
────────────────────────────────────────────────────────────
String::from("text")    from literal
"text".to_string()      from literal
"text".to_owned()       from literal
String::new()           empty
String::with_capacity(n) empty with pre-allocated space
format!("..{}..", val)  build from pieces — preferred

GROWING A STRING
────────────────────────────────────────────────────────────
s.push_str("more")   append &str — doesn't take ownership
s.push('!')          append single char

CONCATENATION
────────────────────────────────────────────────────────────
s1 + &s2     s1 is moved, s2 borrowed — reuses s1's buffer
format!(...)  nothing moved — cleaner for many pieces

CONVERTING
────────────────────────────────────────────────────────────
&owned        &String → &str (deref coercion)
owned.as_str() explicit &str
s.to_string() &str → String

SAFE ITERATION (no indexing by integer!)
────────────────────────────────────────────────────────────
s.chars()            iterate Unicode chars
s.bytes()            iterate raw bytes
s.split_whitespace() iterate words
s.lines()            iterate lines
s.chars().nth(n)     nth char (O(n))

COMMON METHODS
────────────────────────────────────────────────────────────
.trim()/.trim_start()/.trim_end()
.to_uppercase()/.to_lowercase()
.contains(&str)    .find(&str) → Option<usize>
.starts_with() / .ends_with()
.replace(from, to) → String
.split(pattern)    → iterator of &str
.len()             byte count (not char count!)
.chars().count()   character count
.is_empty()

FUNCTION PARAM RULE
────────────────────────────────────────────────────────────
If only reading text → take &str (more general)
If owning/storing   → take String
```

---

## What's Next?

**Lesson 19 — Vec\<T\>** — Rust's growable array: creation, indexing, iteration, sorting, and the most common collection in everyday Rust code.

## Further Reading
- [The Rust Book — Ch 8.2: Storing UTF-8 Encoded Text with Strings](https://doc.rust-lang.org/book/ch08-02-strings.html)
- [std::String documentation](https://doc.rust-lang.org/std/string/struct.String.html)
- [std::str documentation](https://doc.rust-lang.org/std/primitive.str.html)
