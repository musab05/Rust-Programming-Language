# 📘 Lesson 65 — Advanced Lifetimes (AL1)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** AL1 · Category: 🧬 Advanced Patterns  
> **Previous:** [Lesson 64 — FFI](../lesson_64_ffi/lesson_64_ffi.md)  
> **Next:** [Lesson 66 — Advanced Traits](../lesson_66_advanced_traits/lesson_66_advanced_traits.md)  
> **Practice:** [Questions](./lesson_65_questions.md) · [Answers](./lesson_65_answers.md)  
> **Practice Task:** Work through complex lifetime scenarios including HRTB and subtyping

---

## Table of Contents

1. [Lifetime Elision Rules (Review)](#1-lifetime-elision-rules-review)
2. [Multiple Lifetime Parameters](#2-multiple-lifetime-parameters)
3. [Lifetime Bounds on Generics](#3-lifetime-bounds-on-generics)
4. [Lifetime Subtyping](#4-lifetime-subtyping)
5. [Higher-Ranked Trait Bounds (HRTB)](#5-higher-ranked-trait-bounds-hrtb)
6. [Lifetimes in Structs with Traits](#6-lifetimes-in-structs-with-traits)
7. [The 'static Lifetime](#7-the-static-lifetime)
8. [Lifetime in Closures and Async](#8-lifetime-in-closures-and-async)
9. [Common Lifetime Patterns](#9-common-lifetime-patterns)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Lifetime Elision Rules (Review)

The compiler infers lifetimes automatically using three rules:

```rust
// Rule 1: Each reference parameter gets its own lifetime
fn first(s: &str) -> &str { ... }
// becomes: fn first<'a>(s: &'a str) -> &'a str

// Rule 2: If one input lifetime, output gets that lifetime
fn first(s: &str, t: &str) -> &str { ... }
// becomes: fn first<'a, 'b>(s: &'a str, t: &'b str) -> &??? str
// ❌ Ambiguous — must annotate manually

// Rule 3: If &self or &mut self, output gets self's lifetime
impl MyStruct {
    fn name(&self) -> &str { &self.name }
    // becomes: fn name<'a>(&'a self) -> &'a str
}
```

When elision fails, you must annotate:

```rust
// Compiler can't decide: return tied to s1 or s2?
fn longer<'a>(s1: &'a str, s2: &'a str) -> &'a str {
    if s1.len() > s2.len() { s1 } else { s2 }
}
```

---

## 2. Multiple Lifetime Parameters

Different lifetimes for different relationships:

```rust
// Both inputs share the SAME lifetime — returned value can't outlive either
fn first_match<'a>(text: &'a str, pattern: &'a str) -> Option<&'a str> {
    text.find(pattern).map(|i| &text[i..i + pattern.len()])
}

// DIFFERENT lifetimes — return only depends on 'a
fn first_word<'a, 'b>(text: &'a str, _separator: &'b str) -> &'a str {
    text.split_whitespace().next().unwrap_or("")
}

fn main() {
    let text = String::from("hello world");
    let result;
    {
        let sep = String::from(" ");
        result = first_word(&text, &sep);
        // sep dropped here — OK because result's lifetime is tied to text, not sep
    }
    println!("{result}");  // ✅ works!
}
```

### When to use separate lifetimes:

```rust
// If the return only borrows from ONE parameter:
fn extract<'a, 'b>(data: &'a [u8], _metadata: &'b str) -> &'a [u8] {
    &data[0..4]
}

// If the return could borrow from EITHER:
fn choose<'a>(a: &'a str, b: &'a str, pick_first: bool) -> &'a str {
    if pick_first { a } else { b }
}
```

---

## 3. Lifetime Bounds on Generics

Combine lifetimes with generic type parameters:

```rust
use std::fmt::Display;

// T must live at least as long as 'a
fn announce<'a, T: Display>(data: &'a str, extra: T) -> &'a str {
    println!("Announcement: {extra}");
    data
}

// T contains references — those references must outlive 'a
fn process<'a, T: AsRef<str> + 'a>(items: &'a [T]) -> Vec<&'a str> {
    items.iter().map(|s| s.as_ref()).collect()
}

fn main() {
    let data = "important";
    let result = announce(data, "Extra info!");
    println!("{result}");

    let strings = vec!["hello".to_string(), "world".to_string()];
    let refs = process(&strings);
    println!("{:?}", refs);
}
```

### The `T: 'a` bound:

```rust
// "T must not contain any references shorter than 'a"
struct Wrapper<'a, T: 'a> {
    data: &'a T,
}

// T: 'static means T contains no non-static references
fn spawn_thread<T: Send + 'static>(val: T) {
    std::thread::spawn(move || drop(val));
}
```

---

## 4. Lifetime Subtyping

A longer lifetime can be used where a shorter one is expected:

```rust
fn main() {
    let long_lived = String::from("long");       // 'long

    {
        let short_lived = String::from("short"); // 'short
        let result = longer(&long_lived, &short_lived);
        println!("{result}");
        // result lives for 'short (the shorter of the two)
    }
    // can't use result here — short_lived is gone
}

fn longer<'a>(s1: &'a str, s2: &'a str) -> &'a str {
    if s1.len() > s2.len() { s1 } else { s2 }
}
```

### Explicit subtyping with where clauses:

```rust
struct Parser<'input, 'config> where 'config: 'input {
    input: &'input str,
    config: &'config str,
    // 'config outlives 'input — config must live at least as long as input
}

impl<'input, 'config> Parser<'input, 'config>
where 'config: 'input
{
    fn new(input: &'input str, config: &'config str) -> Self {
        Parser { input, config }
    }

    fn parse(&self) -> &'input str {
        if self.config.contains("upper") {
            self.input  // returns with input's lifetime
        } else {
            self.input
        }
    }
}
```

---

## 5. Higher-Ranked Trait Bounds (HRTB)

"For all lifetimes 'a" — when a closure must work with ANY lifetime:

```rust
// Problem: what lifetime does the closure's argument have?
fn apply_to_ref<F>(f: F, text: &str)
where
    F: Fn(&str) -> usize,  // which lifetime for &str?
{
    println!("Length: {}", f(text));
}

fn main() {
    apply_to_ref(|s| s.len(), "hello");  // works!
}
```

Under the hood, the compiler uses HRTB:

```rust
// This desugars to:
fn apply_to_ref<F>(f: F, text: &str)
where
    F: for<'a> Fn(&'a str) -> usize,  // F works for ANY lifetime 'a
{
    println!("Length: {}", f(text));
}
```

### When you need explicit HRTB:

```rust
// Storing a closure that takes a reference
struct Processor {
    handler: Box<dyn for<'a> Fn(&'a str) -> String>,
}

impl Processor {
    fn new<F>(f: F) -> Self
    where
        F: for<'a> Fn(&'a str) -> String + 'static,
    {
        Processor { handler: Box::new(f) }
    }

    fn process(&self, input: &str) -> String {
        (self.handler)(input)
    }
}

fn main() {
    let p = Processor::new(|s| format!("Processed: {s}"));
    println!("{}", p.process("hello"));
    println!("{}", p.process("world"));
}
```

---

## 6. Lifetimes in Structs with Traits

```rust
// A trait with lifetime-bound methods
trait Parser<'a> {
    fn parse(&self, input: &'a str) -> Vec<&'a str>;
}

struct CsvParser;

impl<'a> Parser<'a> for CsvParser {
    fn parse(&self, input: &'a str) -> Vec<&'a str> {
        input.split(',').map(|s| s.trim()).collect()
    }
}

// A struct holding a trait object with lifetime
struct Document<'a> {
    parser: Box<dyn Parser<'a> + 'a>,
    content: &'a str,
}

impl<'a> Document<'a> {
    fn new(content: &'a str, parser: Box<dyn Parser<'a> + 'a>) -> Self {
        Document { parser, content }
    }

    fn fields(&self) -> Vec<&'a str> {
        self.parser.parse(self.content)
    }
}

fn main() {
    let data = "Alice, 30, Engineer";
    let doc = Document::new(data, Box::new(CsvParser));
    println!("{:?}", doc.fields());  // ["Alice", "30", "Engineer"]
}
```

---

## 7. The 'static Lifetime

`'static` means the reference lives for the **entire program**:

```rust
fn main() {
    // String literals are 'static — embedded in the binary
    let s: &'static str = "I live forever";

    // Owned types satisfy 'static (no borrows)
    let owned = String::from("I'm owned");
    requires_static(owned);  // ✅ owned data is 'static

    // References to local variables are NOT 'static
    let local = 42;
    // requires_static_ref(&local);  // ❌ local doesn't live long enough
}

fn requires_static<T: 'static>(val: T) {
    // T contains no non-'static references
    println!("Got a 'static value");
}

// Common misconception: 'static doesn't mean "lives forever"
// It means "CAN live forever" (no borrowed references that expire)
```

### `T: 'static` vs `&'static T`:

```rust
// &'static T → a reference that lives the entire program
// T: 'static → T has no non-static borrows (T could be String, i32, etc.)

fn must_be_static<T: 'static>(_: T) {}

fn main() {
    must_be_static(42);                        // ✅ i32 is 'static (owned, no refs)
    must_be_static(String::from("hello"));     // ✅ String is 'static (owned)
    must_be_static("hello");                   // ✅ &'static str

    let local = String::from("hi");
    // must_be_static(&local);  // ❌ &local is not 'static
    must_be_static(local);      // ✅ moving the String (not borrowing)
}
```

---

## 8. Lifetime in Closures and Async

### Closures and lifetimes:

```rust
fn make_greeter<'a>(prefix: &'a str) -> impl Fn(&str) -> String + 'a {
    move |name| format!("{prefix} {name}!")
}

fn main() {
    let greeting;
    {
        let prefix = String::from("Hello");
        let greeter = make_greeter(&prefix);
        greeting = greeter("Alice");
    }
    println!("{greeting}");  // Hello Alice!
}
```

### Async and lifetimes:

```rust
// Async functions with references need careful lifetime handling
async fn process_data(data: &str) -> usize {
    data.len()
}

// Spawned tasks need 'static — can't borrow local data
async fn example() {
    let data = String::from("hello");

    // ❌ Can't borrow local data in spawned task
    // tokio::spawn(async { process_data(&data).await });

    // ✅ Move owned data into the task
    let data_clone = data.clone();
    tokio::spawn(async move {
        println!("Length: {}", data_clone.len());
    });
}
```

---

## 9. Common Lifetime Patterns

### Pattern 1 — Return borrows from struct:

```rust
struct Config {
    name: String,
    values: Vec<String>,
}

impl Config {
    fn name(&self) -> &str { &self.name }
    fn values(&self) -> &[String] { &self.values }
    fn first_value(&self) -> Option<&str> { self.values.first().map(|s| s.as_str()) }
}
```

### Pattern 2 — Iterator with lifetime:

```rust
struct Words<'a> {
    text: &'a str,
    pos: usize,
}

impl<'a> Iterator for Words<'a> {
    type Item = &'a str;

    fn next(&mut self) -> Option<&'a str> {
        let remaining = &self.text[self.pos..];
        let trimmed = remaining.trim_start();
        if trimmed.is_empty() { return None; }

        self.pos = self.text.len() - trimmed.len();
        let end = trimmed.find(' ').unwrap_or(trimmed.len());
        self.pos += end;
        Some(&trimmed[..end])
    }
}

fn words(text: &str) -> Words<'_> {
    Words { text, pos: 0 }
}

fn main() {
    for word in words("hello brave new world") {
        println!("{word}");
    }
}
```

### Pattern 3 — Lifetime elision with `'_`:

```rust
// Explicit: fn foo<'a>(s: &'a str) -> Words<'a>
// With '_:  fn foo(s: &str) -> Words<'_>
// The '_ means "infer the lifetime"
```

---

## 10. Summary Cheat Sheet

```
ELISION RULES
────────────────────────────────────────────────────────────
1. Each input ref gets its own lifetime
2. One input → output gets that lifetime
3. &self → output gets self's lifetime

MULTIPLE LIFETIMES
────────────────────────────────────────────────────────────
fn f<'a, 'b>(x: &'a str, y: &'b str) -> &'a str
Use separate lifetimes when return ties to one input only

SUBTYPING
────────────────────────────────────────────────────────────
'long: 'short     'long outlives 'short
where 'b: 'a      'b is at least as long as 'a

HRTB
────────────────────────────────────────────────────────────
for<'a> Fn(&'a str) -> &'a str
"This works for ANY lifetime 'a"
Needed for closures taking references

'static
────────────────────────────────────────────────────────────
&'static str       lives entire program (string literals)
T: 'static         T has no non-static borrows (owns its data)

COMMON PATTERNS
────────────────────────────────────────────────────────────
struct Foo<'a> { data: &'a str }     struct borrows
impl<'a> Iterator for Foo<'a>        iterator over borrows
fn f(s: &str) -> Foo<'_>             elision with '_
```

---

## What's Next?

**Lesson 66 — Advanced Traits & Type-Level Programming** — Fully qualified syntax, blanket implementations, and the newtype pattern.

## Further Reading
- [The Rust Book — Ch 10.3: Lifetimes](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)
- [Rustonomicon — Lifetimes](https://doc.rust-lang.org/nomicon/lifetimes.html)
- [Common Rust Lifetime Misconceptions](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md)

---

*Advanced lifetimes: mastering Rust's most unique feature! 🦀*
