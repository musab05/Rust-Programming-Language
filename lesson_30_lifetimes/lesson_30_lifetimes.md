# 📘 Lesson 30 — Lifetimes Basics (O5)

> **Series:** Rust From Zero · Intermediate Level begins!  
> **Roadmap ID:** O5 · Category: 🧠 Ownership  
> **Previous:** [Lesson 29 — File I/O & std::io](../lesson_29_file_io/lesson_29_file_io.md)  
> **Next:** Lesson 31 — Lifetimes in Structs & Advanced *(coming soon)*  
> **Practice:** [Questions](./lesson_30_questions.md) · [Answers](./lesson_30_answers.md)  
> **Practice Task:** Annotate lifetimes on a fn returning the longer of two `&str`

---

## Table of Contents

1. [What Are Lifetimes?](#1-what-are-lifetimes)
2. [The Problem Lifetimes Solve](#2-the-problem-lifetimes-solve)
3. [Lifetime Annotation Syntax](#3-lifetime-annotation-syntax)
4. [Lifetimes in Function Signatures](#4-lifetimes-in-function-signatures)
5. [Multiple Lifetime Parameters](#5-multiple-lifetime-parameters)
6. [Lifetime Elision Rules](#6-lifetime-elision-rules)
7. [The `'static` Lifetime](#7-the-static-lifetime)
8. [Lifetimes and Ownership Mental Model](#8-lifetimes-and-ownership-mental-model)
9. [Common Lifetime Errors](#9-common-lifetime-errors)
10. [Thinking in Lifetimes](#10-thinking-in-lifetimes)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. What Are Lifetimes?

A **lifetime** is the scope for which a reference is valid. Every reference in Rust has a lifetime, though most of the time the compiler figures it out for you.

```rust
fn main() {
    let r;                // r declared here — no value yet

    {
        let x = 5;
        r = &x;          // r borrows x
    }                     // x is dropped here — r would be a dangling reference!

    // println!("{r}");   // ❌ ERROR: `x` does not live long enough
}
```

Lifetimes ensure that **references never outlive the data they point to**. They're the final piece of Rust's ownership system:

| Concept | Lesson | Purpose |
|---|---|---|
| Ownership | 8 | Each value has one owner |
| Move/Copy | 9 | Transfer or duplicate values |
| Borrowing | 10 | References without ownership |
| **Lifetimes** | **30** | **References stay valid** |

---

## 2. The Problem Lifetimes Solve

Consider a function that returns a reference:

```rust
// ❌ This won't compile — Rust doesn't know which lifetime to use
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() { x } else { y }
}
```

**Error:**
```
error[E0106]: missing lifetime specifier
  --> src/main.rs:1:33
   |
1  | fn longest(x: &str, y: &str) -> &str {
   |               ----     ----      ^ expected named lifetime parameter
   |
   = help: this function's return type contains a borrowed value,
     but the signature does not say whether it is borrowed from `x` or `y`
```

The compiler asks: *"The return value is a reference — but a reference to what? x? y? How long should it live?"*

Lifetime annotations answer this question.

---

## 3. Lifetime Annotation Syntax

Lifetime annotations start with an apostrophe `'` followed by a short name (by convention, `'a`, `'b`, `'c`, etc.):

```rust
&i32        // a reference (lifetime implicit)
&'a i32     // a reference with an explicit lifetime 'a
&'a mut i32 // a mutable reference with an explicit lifetime 'a
```

**Important:** Lifetime annotations don't change how long references live. They **describe** relationships between lifetimes so the compiler can verify safety.

Think of them as labels, not instructions:
- ❌ "Make this reference live for lifetime `'a`"
- ✅ "This reference lives for *at least* lifetime `'a`"

---

## 4. Lifetimes in Function Signatures

Let's fix the `longest` function:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    let string1 = String::from("long string is long");

    {
        let string2 = String::from("xyz");
        let result = longest(string1.as_str(), string2.as_str());
        println!("Longest: {result}");  // ✅ both strings alive here
    }
}
```

### What `'a` means here:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str
//         ^^     ^^          ^^            ^^
//  declare  |   x lives at  y lives at   return lives at
//  lifetime |   least 'a    least 'a     least 'a
```

Translation: "The returned reference will be valid for as long as **both** input references are valid."

### When lifetimes matter — dangling reference caught:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    let string1 = String::from("long string");
    let result;

    {
        let string2 = String::from("xyz");
        result = longest(string1.as_str(), string2.as_str());
    }  // string2 dropped here!

    // println!("{result}"); // ❌ ERROR: string2 doesn't live long enough
    // result might point to string2, which is now dead
}
```

---

## 5. Multiple Lifetime Parameters

Sometimes parameters have different lifetimes:

```rust
// The return type only depends on x's lifetime, not y's
fn first<'a, 'b>(x: &'a str, _y: &'b str) -> &'a str {
    x  // always returns x, so only 'a matters for the return
}

fn main() {
    let s1 = String::from("hello");
    let result;

    {
        let s2 = String::from("world");
        result = first(s1.as_str(), s2.as_str());
    }  // s2 dropped here

    println!("{result}"); // ✅ OK! result only depends on s1's lifetime
}
```

### When the return doesn't borrow from inputs:

```rust
// ❌ Can't return a reference to a local variable
fn broken<'a>() -> &'a str {
    let s = String::from("hello");
    &s  // ❌ ERROR: s is dropped when the function returns
    // returning a reference to local data is always wrong
}

// ✅ Return an owned value instead
fn fixed() -> String {
    String::from("hello")
}
```

---

## 6. Lifetime Elision Rules

In many cases, the compiler can infer lifetimes automatically. These are called **elision rules**:

### Rule 1: Each reference parameter gets its own lifetime

```rust
fn foo(x: &str)              →  fn foo<'a>(x: &'a str)
fn foo(x: &str, y: &str)     →  fn foo<'a, 'b>(x: &'a str, y: &'b str)
```

### Rule 2: If there's exactly one input lifetime, it's assigned to all output references

```rust
fn foo(x: &str) -> &str      →  fn foo<'a>(x: &'a str) -> &'a str
```

### Rule 3: If one of the parameters is `&self` or `&mut self`, its lifetime is assigned to outputs

```rust
impl MyStruct {
    fn method(&self, x: &str) -> &str  →  fn method<'a, 'b>(&'a self, x: &'b str) -> &'a str
}
```

### When elision works (no annotation needed):

```rust
// Rule 2 applies — one input reference, one output
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    for (i, &byte) in bytes.iter().enumerate() {
        if byte == b' ' {
            return &s[..i];
        }
    }
    s
}

// No references in output — no lifetime needed
fn length(s: &str) -> usize {
    s.len()
}
```

### When you MUST annotate:

```rust
// Two input references, one output — compiler can't decide which lifetime to use
fn longest(x: &str, y: &str) -> &str { ... }  // ❌ won't compile
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str { ... }  // ✅
```

---

## 7. The `'static` Lifetime

`'static` means "this reference lives for the entire program duration":

```rust
fn main() {
    // String literals have 'static lifetime — they're embedded in the binary
    let s: &'static str = "I live forever!";
    println!("{s}");

    // All string literals are &'static str
    let greeting: &str = "hello";  // actually &'static str
}
```

### When you see `'static`:

```rust
// Function that returns a string literal
fn version() -> &'static str {
    "1.0.0"
}

// Trait bound requiring 'static (common in threading)
fn spawn_thread<F: FnOnce() + Send + 'static>(f: F) {
    // ...
}
```

### ⚠️ Common misconception:

`'static` does NOT mean "allocated on the heap" or "leaked memory". It means "the reference is valid for the entire program". String literals are `'static` because they're baked into the binary.

---

## 8. Lifetimes and Ownership Mental Model

```
OWNED VALUES
────────────────────────────────────────────────────────────
let s = String::from("hello");
  → s OWNS the data
  → s is valid until dropped (end of scope or move)

REFERENCES (BORROWS)
────────────────────────────────────────────────────────────
let r = &s;
  → r BORROWS from s
  → r must not outlive s
  → lifetime of r ≤ lifetime of s

LIFETIME ANNOTATIONS
────────────────────────────────────────────────────────────
fn foo<'a>(x: &'a str) -> &'a str
  → "The output reference lives as long as the input reference"
  → The compiler enforces this at every call site
```

### Visualising lifetimes:

```rust
fn main() {
    let s1 = String::from("hello");     // ─────── s1 ──────────┐
                                        //                       │
    {                                   //                       │
        let s2 = String::from("hi");    // ─ s2 ─┐              │
        let r = longest(&s1, &s2);      //       │ r valid here │
        println!("{r}");                //       │              │
    }                                   // ──────┘              │
                                        //                       │
    // r is no longer valid             //                       │
}                                       // ─────────────────────┘
```

The lifetime `'a` of `r` is the **overlap** of `s1`'s and `s2`'s lifetimes — the shorter one.

---

## 9. Common Lifetime Errors

### Error 1: Returning reference to local data

```rust
// ❌ Can't return a reference to a local variable
fn make_greeting(name: &str) -> &str {
    let greeting = format!("Hello, {name}!");
    &greeting  // ❌ greeting is dropped when function returns
}

// ✅ Fix: return owned data
fn make_greeting(name: &str) -> String {
    format!("Hello, {name}!")
}
```

### Error 2: Reference outlives the data

```rust
fn main() {
    let r;
    {
        let x = 42;
        r = &x;  // ❌ x will be dropped
    }
    // println!("{r}");  // ❌ dangling reference
}

// ✅ Fix: ensure data lives long enough
fn main() {
    let x = 42;
    let r = &x;
    println!("{r}");  // ✅ x is still alive
}
```

### Error 3: Missing lifetime on function with multiple reference params

```rust
// ❌ Compiler can't infer which input's lifetime applies to the output
fn pick(a: &str, b: &str) -> &str {
    a
}

// ✅ Fix: annotate
fn pick<'a>(a: &'a str, _b: &str) -> &'a str {
    a
}
```

---

## 10. Thinking in Lifetimes

### Questions to ask yourself:

1. **"Does my function return a reference?"** → You might need lifetime annotations
2. **"Where does the returned reference point to?"** → It must point to something the caller owns
3. **"Could the reference outlive its source?"** → If yes, the compiler will stop you
4. **"Can I return owned data instead?"** → Often simpler than dealing with lifetimes

### The golden rule:

> **References must always be valid.** Lifetime annotations help the compiler prove this.

### When you don't need lifetimes:

```rust
// No references in return type → no lifetimes needed
fn count_words(s: &str) -> usize { s.split_whitespace().count() }

// Single reference input → elision handles it
fn first_char(s: &str) -> Option<char> { s.chars().next() }

// Owned return type → no lifetime needed
fn to_uppercase(s: &str) -> String { s.to_uppercase() }
```

### When you do need lifetimes:

```rust
// Multiple reference inputs, reference output → annotate
fn longer<'a>(a: &'a str, b: &'a str) -> &'a str { ... }

// Struct containing references → always annotate (Lesson 31)
struct Excerpt<'a> { text: &'a str }
```

---

## 11. Summary Cheat Sheet

```
SYNTAX
────────────────────────────────────────────────────────────
&'a T           reference with lifetime 'a
&'a mut T       mutable reference with lifetime 'a
fn foo<'a>()    function with lifetime parameter 'a

FUNCTION SIGNATURES
────────────────────────────────────────────────────────────
fn f<'a>(x: &'a str) -> &'a str
  → output lives as long as input

fn f<'a>(x: &'a str, y: &'a str) -> &'a str
  → output lives as long as BOTH inputs (the shorter one)

fn f<'a, 'b>(x: &'a str, y: &'b str) -> &'a str
  → output lives as long as x only

ELISION RULES (when you don't need to annotate)
────────────────────────────────────────────────────────────
1. Each input ref gets its own lifetime
2. One input ref → output gets same lifetime
3. &self input → output gets self's lifetime

'static
────────────────────────────────────────────────────────────
&'static str    lives for entire program (string literals)
'static bound   data must live for entire program (threads)

COMMON PATTERNS
────────────────────────────────────────────────────────────
Return owned data (String)     avoids lifetime complexity
Return &'a from single input   elision handles it
Return &'a from multiple       must annotate

CANNOT DO
────────────────────────────────────────────────────────────
Return &local_variable         local data is dropped
Outlive the source             compiler prevents this
```

---

## What's Next?

**Lesson 31 — Lifetimes in Structs & Advanced** *(Intermediate)* — Put references in structs, understand `'static`, lifetime bounds, and advanced patterns.

## Further Reading
- [The Rust Book — Ch 10.3: Validating References with Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)
- [Rust by Example — Lifetimes](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html)
- [Common Rust Lifetime Misconceptions](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md)

---

*Lifetimes are the compiler's way of keeping your references honest! 🦀*
