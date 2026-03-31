# 📘 Lesson 10 — References & Borrowing in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Intermediate — Core Rust safety mechanism
> **Prerequisite:** [Lesson 09 — Move vs Copy](../lesson_09_move_copy.md)
>
> **Files in this lesson:**
> - [`lesson_10_references.md`](./lesson_10_references.md) ← You are here
> - [`lesson_10_questions.md`](./lesson_10_questions.md) — Practice problems
> - [`lesson_10_answers.md`](./lesson_10_answers.md) — Solutions

---

## Table of Contents

1. [The Problem with Moving Everything](#1-the-problem-with-moving-everything)
2. [References — Borrowing Without Taking Ownership](#2-references--borrowing-without-taking-ownership)
   - 2.1 [Creating a Reference — `&`](#21-creating-a-reference--)
   - 2.2 [Dereferencing — `*`](#22-dereferencing--)
   - 2.3 [References in Function Parameters](#23-references-in-function-parameters)
3. [The Borrowing Rules](#3-the-borrowing-rules)
4. [Immutable References — `&T`](#4-immutable-references--t)
   - 4.1 [Multiple Simultaneous Immutable Borrows](#41-multiple-simultaneous-immutable-borrows)
5. [Mutable References — `&mut T`](#5-mutable-references--mut-t)
   - 5.1 [Only One Mutable Borrow at a Time](#51-only-one-mutable-borrow-at-a-time)
   - 5.2 [Cannot Mix Immutable and Mutable](#52-cannot-mix-immutable-and-mutable)
6. [The Borrow Checker](#6-the-borrow-checker)
   - 6.1 [Non-Lexical Lifetimes (NLL)](#61-non-lexical-lifetimes-nll)
7. [Dangling References — Rust Prevents Them](#7-dangling-references--rust-prevents-them)
8. [Slices as References](#8-slices-as-references)
   - 8.1 [String Slices — `&str`](#81-string-slices--str)
   - 8.2 [Array Slices — `&[T]`](#82-array-slices--t)
9. [Reference Coercion](#9-reference-coercion)
10. [Common Patterns with References](#10-common-patterns-with-references)
11. [When to Use References vs Ownership](#11-when-to-use-references-vs-ownership)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. The Problem with Moving Everything

From the previous lesson, we know that passing a `String` to a function moves it — the caller loses access:

```rust
fn calculate_length(s: String) -> usize {
    s.len()
} // s is dropped here

fn main() {
    let s = String::from("hello");
    let len = calculate_length(s);
    // println!("{s}"); // ❌ s was moved into the function
    println!("Length: {len}");
}
```

To keep using `s`, we'd have to return it back from the function — verbose and unergonomic. **References solve this cleanly.**

---

## 2. References — Borrowing Without Taking Ownership

A **reference** lets you refer to a value without taking ownership of it. Think of it as "borrowing" — you get to use the value temporarily, but the original owner retains ownership.

```
Without reference:       With reference:
s1 ──owns──→ heap       s1 ──owns──→ heap
let s2 = s1;                 ↑
s1 is INVALID           r ───borrows─┘
                         r is valid, s1 still owns it
```

### 2.1 Creating a Reference — `&`

Use `&` to create a reference:

```rust
fn main() {
    let s = String::from("hello");
    let r = &s; // r is a reference to s; s still owns the String

    println!("{s}"); // ✅ s still valid
    println!("{r}"); // ✅ r borrows s
}
```

```rust
fn calculate_length(s: &String) -> usize { // takes a reference
    s.len()
} // s goes out of scope BUT the reference is dropped, NOT the String

fn main() {
    let s = String::from("hello");
    let len = calculate_length(&s); // pass a reference — s is BORROWED
    println!("{s} has length {len}"); // ✅ s is still valid!
}
```

### 2.2 Dereferencing — `*`

To access the value a reference points to, use `*`:

```rust
fn main() {
    let x = 5;
    let r = &x;       // r is a reference to x

    println!("{r}");   // Rust auto-derefs for display — prints 5
    println!("{}", *r); // explicit dereference — also 5

    let mut y = 10;
    let m = &mut y;
    *m += 5;           // dereference to modify
    println!("{y}");   // 15
}
```

Rust performs **automatic dereferencing** in many contexts (method calls, comparisons, formatting), so explicit `*` is less common than you might expect.

### 2.3 References in Function Parameters

The most common use of references:

```rust
// Takes a reference — doesn't consume the String
fn count_vowels(s: &str) -> usize {
    s.chars().filter(|c| "aeiouAEIOU".contains(*c)).count()
}

// Takes a mutable reference — can modify without taking ownership
fn make_uppercase(s: &mut String) {
    *s = s.to_uppercase();
}

fn main() {
    let sentence = String::from("hello world");
    let vowels = count_vowels(&sentence);
    println!("{sentence} has {vowels} vowels"); // sentence still valid

    let mut greeting = String::from("hello");
    make_uppercase(&mut greeting);
    println!("{greeting}"); // HELLO
}
```

---

## 3. The Borrowing Rules

Rust enforces these rules at **compile time** through the borrow checker:

```
╔═══════════════════════════════════════════════════════╗
║            THE BORROWING RULES                       ║
║                                                       ║
║  At any given time, you can have EITHER:             ║
║  • Any number of immutable references (&T)           ║
║    OR                                                 ║
║  • Exactly ONE mutable reference (&mut T)            ║
║                                                       ║
║  References must ALWAYS be valid (no dangling)       ║
╚═══════════════════════════════════════════════════════╝
```

These rules prevent two fundamental bugs:
- **Data races** — two threads writing simultaneously
- **Use-after-free** — using a reference after the data is freed

---

## 4. Immutable References — `&T`

An immutable (shared) reference lets you read but not modify the value:

```rust
fn main() {
    let s = String::from("hello");

    let r1 = &s;
    let r2 = &s;
    let r3 = &s;

    // Multiple immutable borrows — all fine!
    println!("{r1}, {r2}, {r3}"); // hello, hello, hello
    println!("{s}");              // s still valid too
}
```

### 4.1 Multiple Simultaneous Immutable Borrows

You can have as many `&T` references as you want simultaneously — because nobody can change the data, there's no conflict:

```rust
fn print_info(name: &String, scores: &Vec<i32>) {
    let avg: f64 = scores.iter().sum::<i32>() as f64 / scores.len() as f64;
    println!("{name}'s average: {avg:.1}");
}

fn main() {
    let name = String::from("Alice");
    let scores = vec![85, 92, 78, 95];

    // Both references exist simultaneously — fine
    let r_name = &name;
    let r_scores = &scores;

    print_info(r_name, r_scores);
    print_info(&name, &scores); // can borrow again

    println!("{name}: {scores:?}"); // originals still valid
}
```

---

## 5. Mutable References — `&mut T`

A mutable reference allows reading AND modifying the borrowed value:

```rust
fn main() {
    let mut s = String::from("hello");

    let r = &mut s;  // mutable reference
    r.push_str(", world"); // modify through the reference

    println!("{r}"); // hello, world
    println!("{s}"); // hello, world — s reflects the change
}
```

**The variable must be declared `mut` to create a mutable reference from it.**

### 5.1 Only One Mutable Borrow at a Time

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &mut s;
    let r2 = &mut s; // ❌ COMPILE ERROR — second mutable borrow!

    println!("{r1}, {r2}");
}
```

**Error:**
```
error[E0499]: cannot borrow `s` as mutable more than once at a time
  --> src/main.rs:5:14
   |
4  |     let r1 = &mut s;
   |              ------ first mutable borrow occurs here
5  |     let r2 = &mut s;
   |              ^^^^^^ second mutable borrow occurs here
6  |     println!("{r1}, {r2}");
   |                -- first borrow later used here
```

**Why?** If two mutable references to the same data existed, two pieces of code could write to it simultaneously — a data race. Rust catches this at compile time.

### 5.2 Cannot Mix Immutable and Mutable

You cannot have a mutable reference while immutable references exist:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;     // immutable borrow
    let r2 = &s;     // another immutable borrow — fine
    let r3 = &mut s; // ❌ ERROR — can't mutably borrow while immutable borrows exist

    println!("{r1}, {r2}, {r3}");
}
```

**Why?** Immutable reference holders expect the data not to change under them. A simultaneous mutable reference would violate that expectation.

---

## 6. The Borrow Checker

The **borrow checker** is the part of the Rust compiler that enforces the borrowing rules. It tracks the "lifetime" of every reference and ensures no rules are violated.

```rust
fn main() {
    let mut data = vec![1, 2, 3];

    let first = &data[0]; // immutable borrow of data
    data.push(4);          // ❌ mutable borrow (push borrows data mutably)
                           //    while first still borrows immutably
    println!("{first}");   // first used here
}
```

**Error:** `cannot borrow 'data' as mutable because it is also borrowed as immutable`

### 6.1 Non-Lexical Lifetimes (NLL)

Since Rust 2018, the borrow checker uses **Non-Lexical Lifetimes** — it tracks the *actual last use* of a reference rather than its syntactic scope. This makes many false-positive errors disappear:

```rust
fn main() {
    let mut s = String::from("hello");

    let r1 = &s;          // immutable borrow starts
    let r2 = &s;          // another immutable borrow
    println!("{r1}, {r2}"); // last use of r1 and r2

    // At this point, r1 and r2 are no longer used — NLL knows this
    // So a mutable borrow is now fine:

    let r3 = &mut s;      // ✅ mutable borrow — no conflict with r1/r2
    r3.push_str(", world");
    println!("{r3}");      // hello, world
}
```

This is the same code that would have failed in older Rust (pre-NLL). Modern Rust is smarter about when borrows actually end.

---

## 7. Dangling References — Rust Prevents Them

A **dangling reference** points to memory that has been freed — using it would be undefined behavior. Rust **prevents this at compile time**:

```rust
fn dangle() -> &String { // ❌ trying to return a reference to a local
    let s = String::from("hello");
    &s
} // s is dropped here — the reference would point to freed memory!
  // Rust refuses to compile this

fn main() {
    let r = dangle();
}
```

**Compiler error:**
```
error[E0106]: missing lifetime specifier
  |
  | fn dangle() -> &String {
  |                ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but there is
    no value for it to be borrowed from
```

The fix: return the owned `String`, not a reference to a local:

```rust
fn no_dangle() -> String {
    let s = String::from("hello");
    s // move ownership to caller — no dangling
}
```

---

## 8. Slices as References

**Slices** are references to a contiguous sequence of elements — they're the most common kind of reference you'll work with.

### 8.1 String Slices — `&str`

A `&str` is a reference to a sequence of UTF-8 bytes:

```rust
fn main() {
    let s = String::from("hello world");

    let hello = &s[0..5];   // &str — slice of first 5 bytes
    let world = &s[6..11];  // &str — slice of last 5 bytes
    let all   = &s[..];     // &str — entire string

    println!("{hello}"); // hello
    println!("{world}"); // world

    // String literals are &str — they're slices into the binary
    let literal: &str = "I am a string literal";
}
```

**Why prefer `&str` over `&String` in function parameters?**

```rust
// ❌ Less flexible — only accepts String references
fn first_word_a(s: &String) -> &str { &s[..s.find(' ').unwrap_or(s.len())] }

// ✅ More flexible — accepts &String AND &str (via deref coercion)
fn first_word(s: &str) -> &str { &s[..s.find(' ').unwrap_or(s.len())] }

fn main() {
    let owned = String::from("hello world");
    let literal = "hello world";

    // Both work with &str parameter:
    println!("{}", first_word(&owned));   // ✅ String coerces to &str
    println!("{}", first_word(literal)); // ✅ literal is already &str
}
```

### 8.2 Array Slices — `&[T]`

```rust
fn sum(data: &[i32]) -> i32 {
    data.iter().sum()
}

fn main() {
    let arr = [1, 2, 3, 4, 5];
    let vec = vec![10, 20, 30];

    println!("{}", sum(&arr));      // ✅ array coerces to slice
    println!("{}", sum(&vec));      // ✅ Vec coerces to slice
    println!("{}", sum(&arr[1..4]));// ✅ sub-slice
}
```

Using `&[T]` (slice) instead of `&Vec<T>` follows the same principle as `&str` vs `&String` — it's more flexible.

---

## 9. Reference Coercion

Rust automatically coerces certain reference types into others — this is called **deref coercion**:

```rust
fn main() {
    let s = String::from("hello");

    // String auto-coerces to &str where needed:
    let r: &str = &s;          // &String → &str
    println!("{}", r.len());   // 5

    // Box<T> auto-coerces to &T:
    let boxed = Box::new(String::from("boxed"));
    let r2: &String = &boxed;  // &Box<String> → &String → &str
    let r3: &str = &boxed;

    // Vec<T> auto-coerces to &[T]:
    let v = vec![1, 2, 3];
    let s: &[i32] = &v;        // &Vec<i32> → &[i32]
}
```

**Coercion chain:**
```
&String → &str
&Vec<T> → &[T]
&Box<T> → &T
```

These coercions happen automatically in function calls, assignments, and other contexts.

---

## 10. Common Patterns with References

### Pattern 1 — Inspect without consuming
```rust
fn is_palindrome(s: &str) -> bool {
    let chars: Vec<char> = s.chars().collect();
    chars == chars.iter().rev().cloned().collect::<Vec<_>>()
}

fn main() {
    let word = String::from("racecar");
    println!("{}: {}", word, is_palindrome(&word));
    println!("{word}"); // still usable
}
```

### Pattern 2 — Modify in place
```rust
fn double_all(data: &mut Vec<i32>) {
    for x in data.iter_mut() {
        *x *= 2;
    }
}

fn main() {
    let mut nums = vec![1, 2, 3, 4, 5];
    double_all(&mut nums);
    println!("{nums:?}"); // [2, 4, 6, 8, 10]
}
```

### Pattern 3 — Return a slice (reference to part of input)
```rust
fn longest<'a>(s1: &'a str, s2: &'a str) -> &'a str {
    if s1.len() >= s2.len() { s1 } else { s2 }
}

fn main() {
    let s1 = String::from("long string");
    let result;
    {
        let s2 = String::from("xy");
        result = longest(&s1, &s2);
        println!("Longest: {result}");
    }
}
```

### Pattern 4 — Accumulate references
```rust
fn find_long_words<'a>(text: &'a str, min_len: usize) -> Vec<&'a str> {
    text.split_whitespace()
        .filter(|w| w.len() >= min_len)
        .collect()
}

fn main() {
    let sentence = "the quick brown fox jumps over the lazy dog";
    let long = find_long_words(sentence, 4);
    println!("{long:?}"); // ["quick", "brown", "jumps", "over", "lazy"]
}
```

---

## 11. When to Use References vs Ownership

```
Function needs to:
├── READ the value, return nothing related → &T
├── MODIFY the value in place → &mut T
├── CONSUME the value (transform into something new) → T (move)
├── STORE the value (in a struct/Vec) → T (own it)
└── Return a value DERIVED from the input → &T with lifetime (or T if new allocation)

Prefer &str over &String
Prefer &[T] over &Vec<T>
Prefer references over cloning where possible
```

---

## 12. Summary Cheat Sheet

```rust
// ── Creating References ───────────────────────────
let r = &value;           // immutable reference
let rm = &mut value;      // mutable reference (value must be mut)

// ── Dereferencing ─────────────────────────────────
let x = *r;               // read through reference
*rm = new_value;          // write through mutable reference
// Note: Rust auto-derefs in many contexts

// ── Borrowing Rules ───────────────────────────────
// EITHER: any number of &T (immutable borrows)
// OR:     exactly one &mut T (mutable borrow)
// Never: &T and &mut T at the same time
// Never: reference outlives what it points to (no dangling)

// ── Function Parameters ───────────────────────────
fn read(s: &String) { }          // immutable borrow
fn modify(s: &mut String) { }    // mutable borrow
fn own(s: String) { }            // move

fn read_slice(s: &str) { }       // prefer over &String
fn read_arr(a: &[i32]) { }       // prefer over &Vec<i32>

// ── Calling ───────────────────────────────────────
read(&s);            // pass immutable reference
modify(&mut s);      // pass mutable reference (s must be mut)
own(s);              // pass ownership (s invalid after)

// ── Slices ────────────────────────────────────────
let s: &str = &string[..];  // entire string as &str
let s: &str = &string[2..5];// substring
let sl: &[i32] = &vec[..];  // entire vec as slice
let sl: &[i32] = &arr[1..3];// sub-array

// ── Coercions ─────────────────────────────────────
&String → &str       (automatic in function calls)
&Vec<T> → &[T]       (automatic in function calls)
```

---

## 📚 Further Reading

- [The Rust Book — Chapter 4.2: References and Borrowing](https://doc.rust-lang.org/book/ch04-02-references-and-borrowing.html)
- [The Rust Book — Chapter 4.3: The Slice Type](https://doc.rust-lang.org/book/ch04-03-slices.html)
- [Rust by Example — Borrowing](https://doc.rust-lang.org/rust-by-example/scope/borrow.html)
- [Rustonomicon — References](https://doc.rust-lang.org/nomicon/references.html)

---

*"Borrowing is a contract: I'll use this carefully and give it back exactly as I found it — or tell you I changed it." 🦀*
