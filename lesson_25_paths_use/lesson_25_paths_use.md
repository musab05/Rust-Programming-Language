# 📘 Lesson 25 — Paths & use (M2)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** M2 · Category: 📁 Modules  
> **Previous:** [Lesson 24 — Modules & mod](../lesson_24_modules/lesson_24_modules.md)  
> **Next:** [Lesson 26 — Crates & Cargo](../lesson_26_crates_cargo/lesson_26_crates_cargo.md)  
> **Practice:** [Questions](./lesson_25_questions.md) · [Answers](./lesson_25_answers.md)  
> **Practice Task:** Bring `std::collections::HashMap` into scope cleanly

---

## Table of Contents

1. [Why `use`?](#1-why-use)
2. [Basic `use` Syntax](#2-basic-use-syntax)
3. [Absolute vs Relative Paths](#3-absolute-vs-relative-paths)
4. [Renaming with `as`](#4-renaming-with-as)
5. [Nested Paths](#5-nested-paths)
6. [The Glob Operator `*`](#6-the-glob-operator-)
7. [Idiomatic `use` Conventions](#7-idiomatic-use-conventions)
8. [`use` in Different Scopes](#8-use-in-different-scopes)
9. [External Crate Paths](#9-external-crate-paths)
10. [Common `std` Imports](#10-common-std-imports)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. Why `use`?

Without `use`, you'd write the full path every time:

```rust
fn main() {
    let mut map = std::collections::HashMap::new();
    map.insert("key", "value");

    let mut set = std::collections::HashSet::new();
    set.insert(42);

    let path = std::path::PathBuf::from("/home/user");
}
```

That's **verbose**. The `use` keyword brings items into the current scope so you can use short names:

```rust
use std::collections::HashMap;
use std::collections::HashSet;
use std::path::PathBuf;

fn main() {
    let mut map = HashMap::new();
    let mut set = HashSet::new();
    let path = PathBuf::from("/home/user");
}
```

Much cleaner!

---

## 2. Basic `use` Syntax

```rust
// Bring a specific item into scope
use std::collections::HashMap;

// Bring a module into scope (then use module::item)
use std::collections;

fn main() {
    // Direct use
    let map: HashMap<String, i32> = HashMap::new();

    // Module-level use
    let set: collections::HashSet<i32> = collections::HashSet::new();
}
```

### With your own modules:

```rust
mod shapes {
    pub fn circle_area(r: f64) -> f64 {
        std::f64::consts::PI * r * r
    }

    pub fn rectangle_area(w: f64, h: f64) -> f64 {
        w * h
    }
}

use shapes::circle_area;

fn main() {
    println!("{:.2}", circle_area(5.0));              // 78.54 — short name
    println!("{:.2}", shapes::rectangle_area(3.0, 4.0)); // 12.00 — full path
}
```

---

## 3. Absolute vs Relative Paths

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {
            println!("Added to waitlist!");
        }
    }
}

// Absolute path — starts with `crate`
use crate::front_of_house::hosting;

// Relative path — starts with module name
// use front_of_house::hosting;  // also works from the crate root

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
}

fn main() {
    eat_at_restaurant();
}
```

### When to use which:

| Path type | When to use |
|---|---|
| Absolute (`crate::`) | When the `use` statement might move between modules |
| Relative | When the relationship is obvious and won't change |

---

## 4. Renaming with `as`

When two types have the same name, use `as` to rename:

```rust
use std::fmt::Result as FmtResult;
use std::io::Result as IoResult;

fn format_something() -> FmtResult {
    Ok(())
}

fn read_something() -> IoResult<String> {
    Ok(String::from("data"))
}

fn main() {
    let _ = format_something();
    let _ = read_something();
}
```

### Another common use — shortening long names:

```rust
use std::collections::HashMap as Map;
use std::collections::BTreeMap as OrderedMap;

fn main() {
    let mut m = Map::new();
    m.insert("a", 1);

    let mut om = OrderedMap::new();
    om.insert("b", 2);
}
```

---

## 5. Nested Paths

Instead of multiple `use` lines with the same prefix, combine them with `{}`:

```rust
// Before — repetitive
use std::collections::HashMap;
use std::collections::HashSet;
use std::collections::BTreeMap;

// After — nested path
use std::collections::{HashMap, HashSet, BTreeMap};
```

### Nesting with `self`:

```rust
// Import both the module and specific items
use std::io::{self, Read, Write};

fn do_io() -> io::Result<()> {
    // `io` is the module (via self)
    // `Read` and `Write` are traits
    Ok(())
}
```

This is equivalent to:
```rust
use std::io;
use std::io::Read;
use std::io::Write;
```

### Deeply nested:

```rust
use std::{
    collections::{HashMap, HashSet},
    fs::{self, File},
    io::{self, BufReader, Read},
};
```

---

## 6. The Glob Operator `*`

The glob (`*`) imports **everything** from a module:

```rust
use std::collections::*;

fn main() {
    let mut map = HashMap::new();    // no prefix needed
    let mut set = HashSet::new();    // no prefix needed
    let mut btree = BTreeMap::new(); // no prefix needed
    map.insert("a", 1);
    set.insert(1);
    btree.insert("b", 2);
}
```

### ⚠️ When to use glob:

| ✅ Use glob | ❌ Avoid glob |
|---|---|
| Tests: `use super::*;` | Application code |
| Preludes: `use my_lib::prelude::*;` | When imports are unclear |
| Quick scripts | When names might clash |

```rust
// Very common in tests:
#[cfg(test)]
mod tests {
    use super::*;  // bring all items from parent into test scope

    #[test]
    fn test_something() {
        // can use all parent functions directly
    }
}
```

---

## 7. Idiomatic `use` Conventions

### Convention 1: Import the parent module for functions

```rust
// ✅ Idiomatic — makes it clear where `add_to_waitlist` comes from
use front_of_house::hosting;
hosting::add_to_waitlist();

// ❌ Less idiomatic — unclear origin
use front_of_house::hosting::add_to_waitlist;
add_to_waitlist();  // where does this come from?
```

### Convention 2: Import the full path for types

```rust
// ✅ Idiomatic — bring types directly into scope
use std::collections::HashMap;
let map = HashMap::new();

// ❌ Less idiomatic for types — unnecessarily verbose
use std::collections;
let map = collections::HashMap::new();
```

### Convention 3: Use `as` when names clash

```rust
use std::fmt::Result as FmtResult;
use std::io::Result as IoResult;
```

### Convention 4: Group and order imports

```rust
// 1. Standard library imports
use std::collections::HashMap;
use std::fs;
use std::io::{self, Read};

// 2. External crate imports
// use serde::{Serialize, Deserialize};

// 3. Local module imports
// use crate::models::User;
// use crate::utils;
```

---

## 8. `use` in Different Scopes

`use` statements respect scope — you can import items inside functions or blocks:

```rust
fn process_data() {
    use std::collections::HashMap;  // only available in this function

    let mut map = HashMap::new();
    map.insert("key", 42);
    println!("{:?}", map);
}

fn main() {
    process_data();

    // HashMap is NOT available here (not in scope)
    // let map = HashMap::new(); // ❌ ERROR
}
```

### Conditional imports (rare but valid):

```rust
fn main() {
    let need_sorting = true;

    if need_sorting {
        use std::collections::BTreeMap;
        let sorted = BTreeMap::from([("b", 2), ("a", 1), ("c", 3)]);
        for (k, v) in &sorted {
            println!("{k}: {v}");  // prints in sorted order
        }
    }
}
```

---

## 9. External Crate Paths

When you add an external dependency in `Cargo.toml`, you use it like a module:

```toml
# Cargo.toml
[dependencies]
rand = "0.8"
serde = { version = "1.0", features = ["derive"] }
```

```rust
// Use the crate name as the root
use rand::Rng;
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    name: String,
    port: u16,
}

fn main() {
    let mut rng = rand::thread_rng();
    let n: u32 = rng.gen_range(1..=100);
    println!("Random: {n}");
}
```

### `std` is special — always available, no `Cargo.toml` entry needed

```rust
// These are always available:
use std::collections::HashMap;
use std::fs::File;
use std::io::{self, Read};
```

---

## 10. Common `std` Imports

Here are the most frequently used `std` imports you'll encounter:

```rust
// Collections
use std::collections::{HashMap, HashSet, BTreeMap, VecDeque};

// I/O
use std::io::{self, Read, Write, BufRead, BufReader, BufWriter};
use std::fs::{self, File};

// Formatting
use std::fmt::{self, Display, Formatter};

// Error handling
use std::error::Error;

// Paths
use std::path::{Path, PathBuf};

// Environment
use std::env;

// Time
use std::time::{Duration, Instant};

// Concurrency
use std::thread;
use std::sync::{Arc, Mutex};
```

---

## 11. Summary Cheat Sheet

```
BASIC USE
────────────────────────────────────────────────────────────
use std::collections::HashMap;      bring item into scope
use module::function;               use function directly

RENAMING
────────────────────────────────────────────────────────────
use std::io::Result as IoResult;    rename to avoid clashes

NESTED PATHS
────────────────────────────────────────────────────────────
use std::collections::{HashMap, HashSet};     multiple items
use std::io::{self, Read, Write};             module + items
use std::{fs, io, collections::HashMap};      mixed levels

GLOB
────────────────────────────────────────────────────────────
use std::collections::*;            import everything (use sparingly)
use super::*;                       common in tests

CONVENTIONS
────────────────────────────────────────────────────────────
Functions → import parent module:   use module; module::func()
Types     → import directly:        use module::Type; Type::new()
Clashing  → rename with as:         use a::Result as AResult;

EXTERNAL CRATES
────────────────────────────────────────────────────────────
use crate_name::item;              after adding to Cargo.toml
use rand::Rng;                     example

SCOPE
────────────────────────────────────────────────────────────
use at top of file                 module-wide scope
use inside fn / block              local scope only
```

---

## What's Next?

**Lesson 26 — Crates & Cargo** — Binary vs library crates, `Cargo.toml`, adding dependencies, and the Rust package ecosystem.

## Further Reading
- [The Rust Book — Ch 7.4: Bringing Paths into Scope with use](https://doc.rust-lang.org/book/ch07-04-bringing-paths-into-scope-with-the-use-keyword.html)
- [Rust API Guidelines — Re-exports](https://rust-lang.github.io/api-guidelines/documentation.html)

---

*Clean imports make clean code! 🦀*
