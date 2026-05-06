# 📘 Lesson 62 — Declarative Macros: macro_rules! (MC1)

> **Series:** Rust From Zero · Advanced Level  
> **Roadmap ID:** MC1 · Category: 🧬 Macros  
> **Previous:** [Lesson 61 — Unsafe Rust](../lesson_61_unsafe/lesson_61_unsafe.md)  
> **Next:** [Lesson 63 — Procedural Macros](../lesson_63_proc_macros/lesson_63_proc_macros.md)  
> **Practice:** [Questions](./lesson_62_questions.md) · [Answers](./lesson_62_answers.md)  
> **Practice Task:** Write a custom `hashmap!` macro and a `debug_print!` macro

---

## Table of Contents

1. [What Are Macros?](#1-what-are-macros)
2. [Your First Macro](#2-your-first-macro)
3. [Matchers and Fragment Specifiers](#3-matchers-and-fragment-specifiers)
4. [Repetition](#4-repetition)
5. [Multiple Arms](#5-multiple-arms)
6. [Common Patterns](#6-common-patterns)
7. [Macro Hygiene](#7-macro-hygiene)
8. [Debugging Macros](#8-debugging-macros)
9. [Real-World Examples](#9-real-world-examples)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Are Macros?

Macros write code that writes code — **metaprogramming**:

```rust
// You've used macros all along!
println!("Hello, {}!", "world");     // macro
vec![1, 2, 3];                       // macro
assert_eq!(1 + 1, 2);               // macro
format!("x = {}", 42);              // macro
```

**Macros vs Functions:**

| | Functions | Macros |
|---|---|---|
| Expanded at | Runtime | Compile time |
| Variable args | ❌ No | ✅ Yes |
| Generate code | ❌ No | ✅ Yes |
| Pattern matching | On values | On syntax |
| Syntax | `fn name()` | `macro_rules! name` |

---

## 2. Your First Macro

```rust
macro_rules! say_hello {
    () => {
        println!("Hello from macro!");
    };
}

fn main() {
    say_hello!();  // Hello from macro!
}
```

### With parameters:

```rust
macro_rules! greet {
    ($name:expr) => {
        println!("Hello, {}!", $name);
    };
}

fn main() {
    greet!("Alice");   // Hello, Alice!
    greet!("Bob");     // Hello, Bob!
    greet!(42);        // Hello, 42!
}
```

---

## 3. Matchers and Fragment Specifiers

Fragment specifiers define what kind of syntax the macro accepts:

| Specifier | Matches | Example |
|---|---|---|
| `$x:expr` | Any expression | `42`, `a + b`, `foo()` |
| `$x:ident` | An identifier | `my_var`, `String` |
| `$x:ty` | A type | `i32`, `Vec<String>` |
| `$x:pat` | A pattern | `Some(x)`, `_` |
| `$x:stmt` | A statement | `let x = 5` |
| `$x:block` | A block | `{ ... }` |
| `$x:literal` | A literal value | `42`, `"hello"` |
| `$x:tt` | A single token tree | Anything |

```rust
macro_rules! create_function {
    ($name:ident, $body:expr) => {
        fn $name() -> i32 {
            $body
        }
    };
}

create_function!(answer, 42);
create_function!(double_21, 21 * 2);

fn main() {
    println!("{}", answer());       // 42
    println!("{}", double_21());    // 42
}
```

### Type parameter:

```rust
macro_rules! make_vec {
    ($t:ty, $($val:expr),*) => {
        {
            let mut v: Vec<$t> = Vec::new();
            $( v.push($val); )*
            v
        }
    };
}

fn main() {
    let v = make_vec!(i32, 1, 2, 3);
    println!("{:?}", v);  // [1, 2, 3]
}
```

---

## 4. Repetition

Handle variable numbers of arguments with `$( ... ),*`:

```rust
// * = zero or more
// + = one or more
// ? = zero or one

macro_rules! print_all {
    ($($item:expr),*) => {
        $(
            println!("{}", $item);
        )*
    };
}

fn main() {
    print_all!(1, "hello", 3.14, true);
}
```

### With separator:

```rust
macro_rules! sum {
    ($($x:expr),+) => {  // + means at least one
        {
            let mut total = 0;
            $( total += $x; )+
            total
        }
    };
}

fn main() {
    println!("{}", sum!(1, 2, 3, 4, 5));  // 15
}
```

### With trailing comma support:

```rust
macro_rules! my_vec {
    () => { Vec::new() };
    ($($elem:expr),+ $(,)?) => {  // $(,)? allows optional trailing comma
        {
            let mut v = Vec::new();
            $( v.push($elem); )+
            v
        }
    };
}

fn main() {
    let a = my_vec![];
    let b = my_vec![1, 2, 3];
    let c = my_vec![1, 2, 3,];  // trailing comma OK
    println!("{:?} {:?} {:?}", a, b, c);
}
```

---

## 5. Multiple Arms

Macros can have multiple pattern arms (like `match`):

```rust
macro_rules! calculate {
    (add $a:expr, $b:expr) => { $a + $b };
    (mul $a:expr, $b:expr) => { $a * $b };
    (neg $a:expr) => { -$a };
}

fn main() {
    println!("{}", calculate!(add 3, 4));   // 7
    println!("{}", calculate!(mul 3, 4));   // 12
    println!("{}", calculate!(neg 5));      // -5
}
```

### Overloading by argument count:

```rust
macro_rules! log {
    ($msg:expr) => {
        println!("[LOG] {}", $msg);
    };
    ($level:expr, $msg:expr) => {
        println!("[{}] {}", $level, $msg);
    };
    ($level:expr, $fmt:expr, $($arg:expr),+) => {
        println!(concat!("[{}] ", $fmt), $level, $($arg),+);
    };
}

fn main() {
    log!("simple message");
    log!("WARN", "something happened");
}
```

---

## 6. Common Patterns

### HashMap literal:

```rust
macro_rules! hashmap {
    ($($key:expr => $val:expr),* $(,)?) => {
        {
            let mut map = std::collections::HashMap::new();
            $( map.insert($key, $val); )*
            map
        }
    };
}

fn main() {
    let scores = hashmap! {
        "Alice" => 95,
        "Bob" => 87,
        "Charlie" => 92,
    };
    println!("{:?}", scores);
}
```

### Struct builder:

```rust
macro_rules! builder {
    ($name:ident { $($field:ident : $ty:ty),* $(,)? }) => {
        struct $name { $( $field: $ty, )* }

        impl $name {
            fn new($( $field: $ty ),*) -> Self {
                $name { $( $field, )* }
            }
        }
    };
}

builder!(Point { x: f64, y: f64 });
builder!(Color { r: u8, g: u8, b: u8 });

fn main() {
    let p = Point::new(1.0, 2.0);
    println!("({}, {})", p.x, p.y);

    let c = Color::new(255, 128, 0);
    println!("rgb({}, {}, {})", c.r, c.g, c.b);
}
```

---

## 7. Macro Hygiene

Rust macros are **hygienic** — they don't accidentally capture variables:

```rust
macro_rules! make_x {
    () => {
        let x = 42;  // this x is in the MACRO's scope
    };
}

fn main() {
    let x = 10;
    make_x!();
    println!("{x}");  // 10 — macro's x doesn't shadow ours
}
```

### Exporting macros:

```rust
// In a library crate:
#[macro_export]
macro_rules! public_macro {
    () => { println!("I'm exported!"); };
}

// In another crate:
// use my_crate::public_macro;
// public_macro!();
```

---

## 8. Debugging Macros

```bash
# See expanded macro output
cargo expand

# Or use the nightly compiler
cargo +nightly rustc -- -Zunpretty=expanded
```

```rust
// Trace macro expansion (nightly only)
#![feature(trace_macros)]

fn main() {
    trace_macros!(true);
    println!("hello");
    trace_macros!(false);
}
```

---

## 9. Real-World Examples

### Debug print with variable names:

```rust
macro_rules! debug_print {
    ($($var:expr),+ $(,)?) => {
        $(
            println!("{} = {:?}", stringify!($var), $var);
        )+
    };
}

fn main() {
    let x = 42;
    let name = "Alice";
    let items = vec![1, 2, 3];

    debug_print!(x, name, items);
    // x = 42
    // name = "Alice"
    // items = [1, 2, 3]
}
```

### Enum with Display:

```rust
macro_rules! display_enum {
    ($name:ident { $($variant:ident),* $(,)? }) => {
        #[derive(Debug)]
        enum $name { $( $variant, )* }

        impl std::fmt::Display for $name {
            fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
                match self {
                    $( $name::$variant => write!(f, stringify!($variant)), )*
                }
            }
        }
    };
}

display_enum!(Color { Red, Green, Blue });

fn main() {
    let c = Color::Green;
    println!("{c}");    // Green
    println!("{c:?}");  // Green
}
```

---

## 10. Summary Cheat Sheet

```
BASIC SYNTAX
────────────────────────────────────────────────────────────
macro_rules! name {
    (pattern) => { expansion };
}

FRAGMENT SPECIFIERS
────────────────────────────────────────────────────────────
$x:expr       expression         42, a + b
$x:ident      identifier         my_var
$x:ty         type               i32, Vec<T>
$x:pat        pattern            Some(x)
$x:literal    literal            42, "hi"
$x:tt         token tree         anything
$x:block      block              { ... }

REPETITION
────────────────────────────────────────────────────────────
$($x:expr),*     zero or more (comma-separated)
$($x:expr),+     one or more
$($x:expr)?      zero or one
$(,)?            optional trailing comma

MULTIPLE ARMS
────────────────────────────────────────────────────────────
macro_rules! m {
    (pattern1) => { ... };
    (pattern2) => { ... };
}

USEFUL BUILT-INS
────────────────────────────────────────────────────────────
stringify!(expr)     → "expr" as string
concat!("a", "b")   → "ab"
file!()              → current file name
line!()              → current line number
```

---

## What's Next?

**Lesson 63 — Procedural Macros** — Derive macros, attribute macros, and function-like proc macros. The power behind `#[derive(Debug)]`.

## Further Reading
- [The Rust Book — Ch 19.6: Macros](https://doc.rust-lang.org/book/ch19-06-macros.html)
- [The Little Book of Rust Macros](https://veykril.github.io/tlborm/)
- [Rust by Example — Macros](https://doc.rust-lang.org/rust-by-example/macros.html)

---

*Macros: code that writes code — Rust's most powerful metaprogramming tool! 🦀*
