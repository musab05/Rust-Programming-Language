# 📘 Lesson 31 — Lifetimes in Structs & Advanced (O6)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** O6 · Category: 🧠 Ownership  
> **Previous:** [Lesson 30 — Lifetimes Basics](../lesson_30_lifetimes/lesson_30_lifetimes.md)  
> **Next:** [Lesson 32 — HashSet, BTreeMap, BTreeSet](../lesson_32_hashset_btree/lesson_32_hashset_btree.md)  
> **Practice:** [Questions](./lesson_31_questions.md) · [Answers](./lesson_31_answers.md)  
> **Practice Task:** Build a struct that holds a reference; annotate correctly

---

## Table of Contents

1. [Lifetimes in Structs](#1-lifetimes-in-structs)
2. [Why Structs Need Lifetime Annotations](#2-why-structs-need-lifetime-annotations)
3. [Methods on Structs with Lifetimes](#3-methods-on-structs-with-lifetimes)
4. [Multiple Lifetimes in Structs](#4-multiple-lifetimes-in-structs)
5. [Lifetime Bounds on Generics](#5-lifetime-bounds-on-generics)
6. [`'static` in Depth](#6-static-in-depth)
7. [Lifetime Subtyping](#7-lifetime-subtyping)
8. [Common Patterns and Idioms](#8-common-patterns-and-idioms)
9. [When to Avoid Lifetimes](#9-when-to-avoid-lifetimes)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Lifetimes in Structs

When a struct holds a reference, you **must** annotate the lifetime. This tells the compiler that the struct can't outlive the data it references.

```rust
struct Excerpt<'a> {
    text: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().unwrap();

    let excerpt = Excerpt { text: first_sentence };
    println!("Excerpt: {}", excerpt.text);
    // excerpt is valid here because novel is still alive
}
```

### The annotation means:

```
struct Excerpt<'a> {
//            ^^   — "I have a lifetime parameter called 'a"
    text: &'a str,
//         ^^      — "my text field borrows data that lives at least as long as 'a"
}
```

Translation: **"An `Excerpt` instance can't outlive the string it references."**

---

## 2. Why Structs Need Lifetime Annotations

Without annotations, the compiler can't verify safety:

```rust
// ❌ Won't compile — missing lifetime
struct BadExcerpt {
    text: &str,  // Error: expected named lifetime parameter
}
```

The compiler needs to know: *"How long does the data behind this reference live?"*

### The dangling reference problem:

```rust
struct Excerpt<'a> {
    text: &'a str,
}

fn main() {
    let excerpt;
    {
        let text = String::from("hello world");
        excerpt = Excerpt { text: &text };
    }  // text is dropped here!
    
    // println!("{}", excerpt.text);  // ❌ ERROR: text doesn't live long enough
    // excerpt.text would point to freed memory
}
```

The lifetime annotation lets the compiler catch this at compile time.

---

## 3. Methods on Structs with Lifetimes

When writing `impl` blocks for structs with lifetimes, you declare the lifetime on the `impl`:

```rust
struct Excerpt<'a> {
    text: &'a str,
}

impl<'a> Excerpt<'a> {
    // Constructor
    fn new(text: &'a str) -> Excerpt<'a> {
        Excerpt { text }
    }

    // Method that returns part of the borrowed text
    fn first_word(&self) -> &str {
        self.text.split_whitespace().next().unwrap_or("")
    }

    // Method returning &self's lifetime (elision Rule 3)
    fn announce(&self, announcement: &str) -> &str {
        println!("Attention: {announcement}");
        self.text  // returns text with lifetime 'a
    }

    // Method returning owned data — no lifetime needed on return
    fn to_uppercase(&self) -> String {
        self.text.to_uppercase()
    }
}

fn main() {
    let text = String::from("hello world from Rust");
    let excerpt = Excerpt::new(&text);

    println!("Full:  {}", excerpt.text);
    println!("First: {}", excerpt.first_word());
    println!("Upper: {}", excerpt.to_uppercase());

    let announced = excerpt.announce("Important!");
    println!("Text:  {announced}");
}
```

### Elision in methods:

```rust
impl<'a> Excerpt<'a> {
    // Elision Rule 3: &self's lifetime is assigned to the output
    fn get_text(&self) -> &str { self.text }
    //                    ^^^^
    // Compiler infers: -> &'a str (from &self)
}
```

---

## 4. Multiple Lifetimes in Structs

A struct can hold references with different lifetimes:

```rust
struct Comparison<'a, 'b> {
    left: &'a str,
    right: &'b str,
}

impl<'a, 'b> Comparison<'a, 'b> {
    fn new(left: &'a str, right: &'b str) -> Comparison<'a, 'b> {
        Comparison { left, right }
    }

    // Returns left — tied to 'a
    fn get_left(&self) -> &'a str {
        self.left
    }

    // Returns right — tied to 'b
    fn get_right(&self) -> &'b str {
        self.right
    }

    fn are_equal(&self) -> bool {
        self.left == self.right
    }
}

fn main() {
    let word1 = String::from("hello");
    let result;

    {
        let word2 = String::from("world");
        let cmp = Comparison::new(&word1, &word2);
        println!("Equal? {}", cmp.are_equal());
        result = cmp.get_left();  // ✅ OK: result has lifetime 'a (word1)
    }
    // word2 dropped here, but result doesn't depend on it

    println!("Left was: {result}");  // ✅ word1 is still alive
}
```

### When to use multiple lifetimes:

- **Same lifetime `'a`**: Both references must live equally long
- **Different lifetimes `'a, 'b`**: References can have independent lifetimes — more flexible

---

## 5. Lifetime Bounds on Generics

You can combine lifetimes with generic type parameters:

```rust
use std::fmt::Display;

// T must implement Display AND live at least as long as 'a
fn announce_and_return<'a, T: Display>(text: &'a str, ann: T) -> &'a str {
    println!("Announcement: {ann}");
    text
}

fn main() {
    let s = String::from("important text");
    let result = announce_and_return(&s, 42);
    println!("{result}");
}
```

### Lifetime bounds on generic types:

```rust
// T must outlive 'a — T contains no references shorter than 'a
struct Wrapper<'a, T: 'a> {
    value: &'a T,
}

impl<'a, T: 'a + std::fmt::Debug> Wrapper<'a, T> {
    fn new(value: &'a T) -> Self {
        Wrapper { value }
    }

    fn inspect(&self) {
        println!("Wrapped: {:?}", self.value);
    }
}

fn main() {
    let num = 42;
    let w = Wrapper::new(&num);
    w.inspect();  // Wrapped: 42

    let text = String::from("hello");
    let w2 = Wrapper::new(&text);
    w2.inspect();  // Wrapped: "hello"
}
```

### `T: 'a` means:

> "Any references inside `T` must live at least as long as `'a`"

If `T` contains no references (like `i32`, `String`), it satisfies `T: 'a` for any `'a`.

---

## 6. `'static` in Depth

### String literals are `'static`:

```rust
fn main() {
    let s1: &'static str = "I live forever";
    let s2: &str = "me too";  // actually &'static str

    // String literals are embedded in the program binary
    // They exist for the entire execution
}
```

### `'static` as a trait bound:

```rust
use std::fmt::Display;

// T: Display + 'static means T owns all its data (no short-lived references)
fn print_static<T: Display + 'static>(value: T) {
    println!("{value}");
}

fn main() {
    print_static(42);                        // ✅ i32 is 'static
    print_static(String::from("hello"));     // ✅ String owns its data
    print_static("literal");                 // ✅ &'static str

    let local = String::from("local");
    // print_static(&local);                 // ❌ &local is not 'static
    print_static(local);                     // ✅ move the owned String in
}
```

### `'static` does NOT mean:

| Misconception | Reality |
|---|---|
| "Lives on the heap" | String literals are in the binary's read-only section |
| "Leaked memory" | It means the data is valid for the whole program |
| "Must be allocated with Box::leak" | That's ONE way to get `'static`, not the only way |

### Ways to get `'static` data:

```rust
// 1. String literals
let a: &'static str = "hello";

// 2. Constants
const MAX: i32 = 100;
static GREETING: &str = "hi";

// 3. Owned types satisfy T: 'static (no borrowed data inside)
fn takes_static<T: 'static>(val: T) { /* ... */ }
// String, Vec<i32>, i32, bool all satisfy 'static

// 4. Box::leak (deliberately leak memory — rare)
let leaked: &'static str = Box::leak(String::from("leaked").into_boxed_str());
```

---

## 7. Lifetime Subtyping

A longer lifetime can be used where a shorter one is expected:

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    let s1 = String::from("long string");     // lives for all of main
    {
        let s2 = String::from("hi");           // lives for this block
        // 'a is the OVERLAP — the shorter of the two lifetimes
        let result = longest(&s1, &s2);
        println!("{result}");  // ✅ both alive here
    }
}
```

### The key principle:

> If `'long: 'short` (read: "`'long` outlives `'short`"), then `&'long T` can be used where `&'short T` is expected.

```rust
// Explicitly declaring: 'a outlives 'b
fn first_is_longer<'a: 'b, 'b>(first: &'a str, _second: &'b str) -> &'b str {
    first  // ✅ 'a outlives 'b, so &'a str can be returned as &'b str
}
```

---

## 8. Common Patterns and Idioms

### Pattern 1: Config / Context struct

```rust
struct Config<'a> {
    name: &'a str,
    verbose: bool,
}

fn run(config: &Config) {
    if config.verbose {
        println!("Running: {}", config.name);
    }
}

fn main() {
    let app_name = String::from("MyApp");
    let config = Config {
        name: &app_name,
        verbose: true,
    };
    run(&config);
}
```

### Pattern 2: Parser holding input

```rust
struct Parser<'input> {
    source: &'input str,
    position: usize,
}

impl<'input> Parser<'input> {
    fn new(source: &'input str) -> Self {
        Parser { source, position: 0 }
    }

    fn peek(&self) -> Option<char> {
        self.source[self.position..].chars().next()
    }

    fn remaining(&self) -> &'input str {
        &self.source[self.position..]
    }

    fn advance(&mut self, n: usize) {
        self.position = (self.position + n).min(self.source.len());
    }

    fn take_while(&mut self, pred: fn(char) -> bool) -> &'input str {
        let start = self.position;
        while let Some(c) = self.peek() {
            if pred(c) {
                self.advance(c.len_utf8());
            } else {
                break;
            }
        }
        &self.source[start..self.position]
    }
}

fn main() {
    let input = "hello world 42";
    let mut parser = Parser::new(input);

    let word = parser.take_while(|c| c.is_alphabetic());
    println!("Word: {word}");  // "hello"

    parser.advance(1);  // skip space
    let word2 = parser.take_while(|c| c.is_alphabetic());
    println!("Word: {word2}");  // "world"

    parser.advance(1);
    let num = parser.take_while(|c| c.is_numeric());
    println!("Number: {num}");  // "42"
}
```

### Pattern 3: Struct returning slices of its data

```rust
struct SplitResult<'a> {
    left: &'a str,
    right: &'a str,
}

fn split_at_char<'a>(s: &'a str, c: char) -> SplitResult<'a> {
    match s.find(c) {
        Some(pos) => SplitResult {
            left: &s[..pos],
            right: &s[pos + c.len_utf8()..],
        },
        None => SplitResult {
            left: s,
            right: "",
        },
    }
}

fn main() {
    let data = "key=value";
    let result = split_at_char(data, '=');
    println!("{} -> {}", result.left, result.right);
    // key -> value
}
```

---

## 9. When to Avoid Lifetimes

Sometimes lifetimes add complexity without benefit. Consider these alternatives:

### Use owned data instead:

```rust
// ❌ Complex — struct borrows data
struct MessageRef<'a> {
    sender: &'a str,
    body: &'a str,
}

// ✅ Simpler — struct owns data
struct Message {
    sender: String,
    body: String,
}
```

### Use `.clone()` or `.to_string()`:

```rust
// Instead of fighting lifetimes, sometimes just clone
fn process(data: &str) -> String {
    data.to_uppercase()  // returns owned String, no lifetime needed
}
```

### Use `Cow<'a, str>` for flexibility (Lesson 38+):

```rust
use std::borrow::Cow;

// Can be either borrowed OR owned — best of both worlds
fn maybe_modify(s: &str) -> Cow<str> {
    if s.contains("bad") {
        Cow::Owned(s.replace("bad", "good"))
    } else {
        Cow::Borrowed(s)
    }
}
```

### When to use lifetimes:

| Situation | Use Lifetimes? |
|---|---|
| Struct borrows data temporarily | ✅ Yes |
| Parser holding input text | ✅ Yes |
| Long-lived struct (stored in collections) | ❌ Own the data |
| Simple data transfer objects | ❌ Own the data |
| Performance-critical hot paths | ✅ Avoid cloning |

---

## 10. Summary Cheat Sheet

```
STRUCT LIFETIMES
────────────────────────────────────────────────────────────
struct Foo<'a> { x: &'a str }       struct with one lifetime
struct Foo<'a, 'b> { x: &'a str, y: &'b str }   multiple lifetimes

IMPL BLOCKS
────────────────────────────────────────────────────────────
impl<'a> Foo<'a> { ... }            declare lifetime on impl
fn method(&self) -> &str            elision Rule 3 applies

LIFETIME BOUNDS
────────────────────────────────────────────────────────────
T: 'a                               T has no references shorter than 'a
T: 'static                          T owns all its data (no borrows)
'a: 'b                              'a outlives 'b

'static IN DEPTH
────────────────────────────────────────────────────────────
&'static str                        reference valid for entire program
T: 'static                          T contains no non-static references
String satisfies 'static            owned types have no borrows
"literal" is &'static str           baked into the binary

COMMON PATTERNS
────────────────────────────────────────────────────────────
Config<'a>                          short-lived struct borrowing config
Parser<'input>                      parser holding reference to input
SplitResult<'a>                     returning multiple slices

WHEN TO AVOID
────────────────────────────────────────────────────────────
Long-lived data → own it (String, Vec)
Simple DTOs → own the fields
Fighting the borrow checker → clone, then optimize
```

---

## What's Next?

**Lesson 32 — HashSet, BTreeMap, BTreeSet** — Expand your collection toolkit with sets and sorted maps. Learn set operations, sorted iteration, and when to pick each collection.

## Further Reading
- [The Rust Book — Ch 10.3: Lifetimes (continued)](https://doc.rust-lang.org/book/ch10-03-lifetime-syntax.html)
- [Rust Reference — Lifetime Elision](https://doc.rust-lang.org/reference/lifetime-elision.html)
- [Common Rust Lifetime Misconceptions](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md)

---

*Lifetimes in structs: borrowing data without owning it! 🦀*
