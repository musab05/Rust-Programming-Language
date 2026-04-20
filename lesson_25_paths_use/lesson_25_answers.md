# ✅ Lesson 25 — Answers: Paths & use (M2)

---

## Section A

### A1
```
2
Some(1)
```
`HashMap` is brought into scope. 2 entries inserted. `get("a")` returns `Some(&1)`.

### A2
```
🦁 🐯
```
`use zoo::animals` brings the module into scope. `animals::lion()` and `animals::tiger()` work.

### A3
```
Success
```
`FmtResult` is an alias for `std::fmt::Result`. `greet()` returns `Ok(())`. Match arm prints "Success".

---

## Section B

### A4 — ❌ No
`use std::collections::HashMap;` inside `main()` is scoped to `main`. The `other()` function can't see it. Fix: move the `use` to the top of the file.

### A5 — ✅ Yes
Nested path `{HashMap, HashSet}` is valid. Output: `1 1`.

### A6 — ✅ Yes
`self` imports the `io` module itself, and `Read` imports the trait. `io::Result<()>` works correctly. Output: `Hello`.

### A7 — ⚠️ Compiles with a warning
Duplicate `use` produces a warning (`unused import`) but is not an error. The code compiles fine.

---

## Section C

### A8
```rust
use std::collections::HashMap;  // fix: add use statement

fn main() {
    let mut map = HashMap::new();
    map.insert("x", 10);
    println!("{:?}", map);
}
```

### A9
```rust
use std::fmt::Result as FmtResult;   // fix: rename one or both
use std::io::Result as IoResult;

fn format_thing() -> FmtResult {
    Ok(())
}
```

### A10
```rust
mod kitchen {
    pub mod pantry {  // fix: make pantry pub
        pub fn get_ingredient() -> &'static str {
            "flour"
        }
    }

    pub fn cook() {
        let item = pantry::get_ingredient();
        println!("Cooking with {item}");
    }
}

use kitchen::pantry::get_ingredient;  // ✅ now works

fn main() {
    kitchen::cook();
    println!("{}", get_ingredient());
}
```

---

## Section D

### A11 — Clean imports
```rust
use std::{
    collections::{HashMap, HashSet},
    fs::File,
    io::{self, Read},
};

fn main() {
    let mut map = HashMap::new();
    map.insert("language", "Rust");

    let mut set = HashSet::new();
    set.insert(42);

    println!("HashMap: {:?}", map);
    println!("HashSet: {:?}", set);
    println!("File and io::Read are in scope!");
}
```

### A12 — Alias practice
```rust
use std::collections::BTreeMap as OrderedMap;
use std::collections::HashMap as Map;

fn main() {
    let mut m = Map::new();
    m.insert("b", 2);
    m.insert("a", 1);
    println!("Map: {:?}", m);  // order not guaranteed

    let mut om = OrderedMap::new();
    om.insert("b", 2);
    om.insert("a", 1);
    println!("OrderedMap: {:?}", om);  // sorted by key: {"a": 1, "b": 2}
}
```

### A13 — Module with use
```rust
mod math_utils {
    pub fn add(a: i32, b: i32) -> i32 { a + b }
    pub fn subtract(a: i32, b: i32) -> i32 { a - b }
    pub fn multiply(a: i32, b: i32) -> i32 { a * b }
}

use math_utils::add;  // bring add directly into scope

fn main() {
    // Style 1: direct use (via `use`)
    println!("3 + 4 = {}", add(3, 4));

    // Style 2: module path
    println!("10 - 3 = {}", math_utils::subtract(10, 3));
    println!("5 × 6 = {}", math_utils::multiply(5, 6));
}
```

---

## Section E

### A14 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `use` only affects the scope where it appears (file, fn, or block) |
| 2 | **True** | `self` imports `io` as a name, `Read` imports the trait |
| 3 | **False** | Glob (`*`) obscures where names come from; only recommended for tests/preludes |
| 4 | **True** | `as` creates an alias for the imported item |
| 5 | **True** | Absolute paths to your own items start with `crate::` |
| 6 | **True** | External crate names serve as the root (e.g., `rand::Rng`) |
| 7 | **False** | `use` can appear inside functions, blocks, or at the top of a module |
| 8 | **True** | `use hosting; hosting::add()` is clearer than `use hosting::add; add()` |
| 9 | **True** | `use super::*;` is the standard pattern for importing parent items in tests |
| 10 | **False** | Duplicate `use` produces a warning, not an error |

### A15 — Refactor challenge
```rust
use std::{
    collections::{BTreeMap, HashMap, HashSet},
    fs::{self, File},
    io::{self, BufReader, Read, Write},
};
```
9 lines → 4 lines. All items still available.

---

## 🏆 Lesson 25 Complete!

✅ `use` to bring items into scope  
✅ Absolute (`crate::`) vs relative paths  
✅ `as` for renaming imports  
✅ Nested paths with `{}`  
✅ `self` in nested imports  
✅ Glob `*` (sparingly)  
✅ Idiomatic conventions for functions vs types  
✅ Scoped `use` inside functions  

**Up next: Lesson 26 — Crates & Cargo 🦀**
