# 📘 Lesson 83 — Cow\<T\>: Clone on Write (SP5)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** SP5 · Category: 📌 Smart Pointers  
> **Previous:** [Lesson 82 — Trait Objects & Object Safety](../lesson_82_trait_objects/lesson_82_trait_objects.md)  
> **Next:** [Lesson 84 — Async Streams & Channels](../lesson_84_async_streams/lesson_84_async_streams.md)  
> **Practice:** [Questions](./lesson_83_questions.md) · [Answers](./lesson_83_answers.md)  
> **Practice Task:** Function that conditionally modifies a &str using Cow

---

## Table of Contents

1. [What Is Cow?](#1-what-is-cow)
2. [Cow Variants](#2-cow-variants)
3. [Basic Usage](#3-basic-usage)
4. [Avoiding Allocations](#4-avoiding-allocations)
5. [Cow with Strings](#5-cow-with-strings)
6. [Cow in Function Signatures](#6-cow-in-function-signatures)
7. [Cow with Slices](#7-cow-with-slices)
8. [When to Use Cow](#8-when-to-use-cow)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. What Is Cow?

**Clone on Write** — either borrows OR owns data, cloning only when mutation is needed:

```rust
use std::borrow::Cow;

enum Cow<'a, B: ToOwned + ?Sized> {
    Borrowed(&'a B),       // no allocation — just a reference
    Owned(<B as ToOwned>::Owned),  // allocated — owns the data
}
```

**Key idea:** Start borrowed. Only allocate if you need to modify.

---

## 2. Cow Variants

```rust
use std::borrow::Cow;

fn main() {
    // Borrowed — no allocation
    let borrowed: Cow<str> = Cow::Borrowed("hello");
    println!("Borrowed: {borrowed}");

    // Owned — allocated
    let owned: Cow<str> = Cow::Owned(String::from("world"));
    println!("Owned: {owned}");

    // Both can be used the same way
    println!("Length: {}, {}", borrowed.len(), owned.len());
}
```

---

## 3. Basic Usage

```rust
use std::borrow::Cow;

fn maybe_uppercase(s: &str, should_upper: bool) -> Cow<str> {
    if should_upper {
        Cow::Owned(s.to_uppercase())  // allocates only when needed
    } else {
        Cow::Borrowed(s)              // zero allocation!
    }
}

fn main() {
    let original = "hello world";

    let result1 = maybe_uppercase(original, false);
    println!("No change: {result1}");  // "hello world" — borrowed, no alloc

    let result2 = maybe_uppercase(original, true);
    println!("Changed:   {result2}");  // "HELLO WORLD" — owned, allocated
}
```

---

## 4. Avoiding Allocations

```rust
use std::borrow::Cow;

/// Trim and normalize whitespace — only allocates if changes are needed
fn normalize(input: &str) -> Cow<str> {
    let trimmed = input.trim();

    // Check if any normalization is needed
    if trimmed == input && !input.contains("  ") {
        Cow::Borrowed(input)  // return as-is — no allocation
    } else {
        // Need to modify — allocate
        let normalized: String = trimmed
            .split_whitespace()
            .collect::<Vec<_>>()
            .join(" ");
        Cow::Owned(normalized)
    }
}

fn main() {
    let clean = "hello world";
    let dirty = "  hello   world  ";

    let r1 = normalize(clean);
    println!("{r1:?} — allocated: {}", matches!(r1, Cow::Owned(_)));
    // "hello world" — allocated: false

    let r2 = normalize(dirty);
    println!("{r2:?} — allocated: {}", matches!(r2, Cow::Owned(_)));
    // "hello world" — allocated: true
}
```

---

## 5. Cow with Strings

```rust
use std::borrow::Cow;

/// Escape HTML special characters — only allocates if escaping is needed
fn escape_html(input: &str) -> Cow<str> {
    if input.contains('&') || input.contains('<') || input.contains('>') || input.contains('"') {
        let escaped = input
            .replace('&', "&amp;")
            .replace('<', "&lt;")
            .replace('>', "&gt;")
            .replace('"', "&quot;");
        Cow::Owned(escaped)
    } else {
        Cow::Borrowed(input)  // no escaping needed — zero cost
    }
}

fn main() {
    let safe = "Hello, world!";
    let dangerous = "Hello <script>alert('xss')</script>";

    println!("{}", escape_html(safe));       // borrowed — no alloc
    println!("{}", escape_html(dangerous));  // owned — allocated
}
```

### to_mut() — lazy clone:

```rust
use std::borrow::Cow;

fn ensure_prefix(s: &str, prefix: &str) -> Cow<str> {
    if s.starts_with(prefix) {
        Cow::Borrowed(s)
    } else {
        let mut owned = String::with_capacity(prefix.len() + s.len());
        owned.push_str(prefix);
        owned.push_str(s);
        Cow::Owned(owned)
    }
}

fn main() {
    println!("{}", ensure_prefix("https://rust-lang.org", "https://"));  // borrowed
    println!("{}", ensure_prefix("rust-lang.org", "https://"));          // owned
}
```

---

## 6. Cow in Function Signatures

```rust
use std::borrow::Cow;

// Accept both &str and String without allocation overhead
fn log_message(msg: Cow<str>) {
    println!("[LOG] {msg}");
}

// Return Cow to let the caller decide
fn get_greeting(name: &str) -> Cow<str> {
    if name.is_empty() {
        Cow::Borrowed("Hello, stranger!")  // static str, no alloc
    } else {
        Cow::Owned(format!("Hello, {name}!"))  // dynamic, allocated
    }
}

fn main() {
    log_message(Cow::Borrowed("static message"));
    log_message(Cow::Owned(format!("dynamic {}", 42)));

    println!("{}", get_greeting(""));
    println!("{}", get_greeting("Alice"));
}
```

---

## 7. Cow with Slices

```rust
use std::borrow::Cow;

/// Remove negative numbers — only allocates if there are negatives
fn filter_positives(nums: &[i32]) -> Cow<[i32]> {
    if nums.iter().all(|&n| n >= 0) {
        Cow::Borrowed(nums)  // all positive — no allocation
    } else {
        Cow::Owned(nums.iter().copied().filter(|&n| n >= 0).collect())
    }
}

fn main() {
    let all_pos = vec![1, 2, 3, 4, 5];
    let mixed = vec![1, -2, 3, -4, 5];

    let r1 = filter_positives(&all_pos);
    println!("{:?} (borrowed: {})", &*r1, matches!(r1, Cow::Borrowed(_)));

    let r2 = filter_positives(&mixed);
    println!("{:?} (borrowed: {})", &*r2, matches!(r2, Cow::Borrowed(_)));
}
```

---

## 8. When to Use Cow

### ✅ USE Cow when:

- Most calls **won't modify** the data (happy path = borrowed)
- You want to avoid **unnecessary cloning**
- Function might or might not need to **allocate**
- Working with **string processing** (trim, escape, normalize)
- **Parsing** where most input passes through unchanged

### ❌ DON'T use Cow when:

- You **always** need to modify the data (just take `String`)
- You **never** modify the data (just take `&str`)
- The overhead of the enum check isn't worth the saved allocations
- Code readability suffers for minimal performance gain

---

## 9. Summary Cheat Sheet

```
COW DEFINITION
────────────────────────────────────────────────────────────
Cow::Borrowed(&data)     no allocation, just a reference
Cow::Owned(owned_data)   allocated, owns the data

COMMON TYPES
────────────────────────────────────────────────────────────
Cow<'a, str>      → borrows &str OR owns String
Cow<'a, [T]>      → borrows &[T] OR owns Vec<T>
Cow<'a, Path>     → borrows &Path OR owns PathBuf

KEY PATTERN
────────────────────────────────────────────────────────────
fn process(input: &str) -> Cow<str> {
    if needs_change { Cow::Owned(modified) }
    else { Cow::Borrowed(input) }
}

METHODS
────────────────────────────────────────────────────────────
cow.to_mut()       get &mut — clones if Borrowed
cow.into_owned()   consume into owned type
matches!(cow, Cow::Borrowed(_))  check variant

WHEN TO USE
────────────────────────────────────────────────────────────
Most calls don't modify     → Cow saves allocations
Always modifies             → just use String/Vec
Never modifies              → just use &str/&[T]
```

---

## What's Next?

**Lesson 84 — Async Streams & Channels** — Stream trait, tokio::sync::mpsc, broadcast, and watch channels.

## Further Reading
- [std::borrow::Cow](https://doc.rust-lang.org/std/borrow/enum.Cow.html)
- [Effective Rust — Cow](https://www.lurklurk.org/effective-rust/cow.html)

---

*Cow: borrow when you can, own when you must! 🦀*
