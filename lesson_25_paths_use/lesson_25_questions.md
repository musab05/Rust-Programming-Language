# 🧪 Lesson 25 — Questions: Paths & use (M2)

> **Lesson:** [lesson_25_paths_use.md](./lesson_25_paths_use.md)  
> **Answers:** [lesson_25_answers.md](./lesson_25_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert("a", 1);
    map.insert("b", 2);
    println!("{}", map.len());
    println!("{:?}", map.get("a"));
}
```

### Q2
```rust
mod zoo {
    pub mod animals {
        pub fn lion() -> &'static str { "🦁" }
        pub fn tiger() -> &'static str { "🐯" }
    }
}

use zoo::animals;

fn main() {
    println!("{} {}", animals::lion(), animals::tiger());
}
```

### Q3
```rust
use std::fmt::Result as FmtResult;

fn greet() -> FmtResult {
    Ok(())
}

fn main() {
    match greet() {
        Ok(()) => println!("Success"),
        Err(_) => println!("Failed"),
    }
}
```

---

## Section B — Will It Compile?

### Q4
```rust
fn main() {
    use std::collections::HashMap;
    let map: HashMap<&str, i32> = HashMap::new();
    println!("{}", map.len());
}

fn other() {
    let map: HashMap<&str, i32> = HashMap::new(); // HashMap in scope here?
}
```

### Q5
```rust
use std::collections::{HashMap, HashSet};

fn main() {
    let mut map = HashMap::new();
    let mut set = HashSet::new();
    map.insert("key", 1);
    set.insert(1);
    println!("{} {}", map.len(), set.len());
}
```

### Q6
```rust
use std::io::{self, Read};

fn main() -> io::Result<()> {
    println!("Hello");
    Ok(())
}
```

### Q7
```rust
use std::collections::HashMap;
use std::collections::HashMap;  // duplicate

fn main() {
    let map = HashMap::<String, i32>::new();
}
```

---

## Section C — Fix the Bugs

### Q8
```rust
fn main() {
    let mut map = HashMap::new();  // BUG: HashMap not in scope
    map.insert("x", 10);
    println!("{:?}", map);
}
```

### Q9
```rust
use std::fmt::Result;
use std::io::Result;  // BUG: name clash

fn format_thing() -> Result {
    Ok(())
}
```

### Q10
```rust
mod kitchen {
    mod pantry {  // BUG: not pub
        pub fn get_ingredient() -> &'static str {
            "flour"
        }
    }

    pub fn cook() {
        let item = pantry::get_ingredient();
        println!("Cooking with {item}");
    }
}

use kitchen::pantry::get_ingredient;  // BUG: pantry is private

fn main() {
    kitchen::cook();
    println!("{}", get_ingredient());
}
```

---

## Section D — Write It Yourself

### Q11 — Clean imports (Roadmap Practice Task)
Write a program that uses all of the following, brought into scope with clean `use` statements:
- `HashMap` from `std::collections`
- `HashSet` from `std::collections`
- `File` from `std::fs`
- `io` module from `std`
- `Read` trait from `std::io`

Use nested paths where possible. In `main`, create a `HashMap`, a `HashSet`, and print a message. (Don't actually open a file — just show the imports compile.)

### Q12 — Alias practice
Import `std::collections::BTreeMap` as `OrderedMap` and `std::collections::HashMap` as `Map`. Create one of each, insert data, and print both.

### Q13 — Module with use
Create a module `math_utils` with functions `add`, `subtract`, `multiply`. Then in `main`:
1. Use `use math_utils::add;` to bring `add` directly into scope
2. Call `subtract` and `multiply` via the module path
3. Show both styles working

---

## Section E — Deep Understanding

### Q14 — True or False?
1. `use` brings an item into the current scope only.
2. `use std::io::{self, Read}` imports both the `io` module and the `Read` trait.
3. The glob operator `*` is recommended for production code.
4. `use x as y` creates a new name `y` for item `x`.
5. You must use `crate::` for absolute paths to your own modules.
6. External crate names act as the root of their path (like `crate` for your own code).
7. `use` can only appear at the top of a file.
8. Importing a parent module for functions is more idiomatic than importing the function directly.
9. `use super::*;` is commonly used in test modules.
10. Two `use` statements importing the same item will cause a compile error.

### Q15 — Refactor challenge
Simplify these imports using nested paths:
```rust
use std::collections::HashMap;
use std::collections::HashSet;
use std::collections::BTreeMap;
use std::io;
use std::io::Read;
use std::io::Write;
use std::io::BufReader;
use std::fs;
use std::fs::File;
```

---

*Clean imports = clean code! 🦀*
