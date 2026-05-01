# ✅ Lesson 31 — Answers: Lifetimes in Structs & Advanced (O6)

---

## Section A

### A1 — ❌ Won't compile
`s` is dropped at the end of the inner block. `h.data` would be a dangling reference. Error: `s does not live long enough`.

### A2 — ✅ Compiles
`s` lives until the end of `main`. `h` borrows `s` and is used while `s` is still alive.

### A3 — ✅ Compiles
`result` is assigned `p.first`, which borrows from `a`. Even though `b` (and `p`) are dropped, `result` only depends on `a`'s lifetime. Since `Pair` has separate lifetimes `'a` and `'b`, the compiler knows `first` and `second` are independent.

### A4 — ✅ Compiles
`x` lives until end of `main`. `w` borrows `x`. The `get()` method returns `&i32` — elision Rule 3 assigns `&self`'s lifetime. Everything is valid.

### A5 — ❌ Won't compile
`s` is a local `String` created inside `make_holder`. It's dropped when the function returns. Can't return a struct holding a reference to it. Fix: return owned data or accept a reference as parameter.

---

## Section B

### A6
```rust
struct Name<'a> {
    value: &'a str,
}

fn main() {
    let n = Name { value: "Alice" };
    println!("{}", n.value);
}
```
Added lifetime parameter `'a` to the struct. References in struct fields always need lifetime annotations.

### A7
```rust
struct TextBlock<'a> {
    content: &'a str,
}

// Option 1: Accept a reference parameter
fn create_block<'a>(text: &'a str) -> TextBlock<'a> {
    TextBlock { content: text }
}

// Option 2: Return owned data
struct OwnedTextBlock {
    content: String,
}

fn create_owned_block() -> OwnedTextBlock {
    let text = String::from("hello");
    OwnedTextBlock { content: text }
}
```
Can't return a reference to a local variable. Either accept the data as a parameter, or use owned data.

### A8
```rust
struct Config<'a> {
    name: &'a str,
}

impl<'a> Config<'a> {
    fn new(name: &'a str) -> Config<'a> {
        Config { name }
    }
}
```
The `impl` block must declare the lifetime parameter: `impl<'a> Config<'a>`.

---

## Section C

### A9 — Excerpt struct
```rust
struct Excerpt<'a> {
    text: &'a str,
}

impl<'a> Excerpt<'a> {
    fn new(text: &'a str) -> Excerpt<'a> {
        Excerpt { text }
    }

    fn word_count(&self) -> usize {
        self.text.split_whitespace().count()
    }

    fn first_word(&self) -> &str {
        self.text.split_whitespace().next().unwrap_or("")
    }
}

fn main() {
    // With a String
    let novel = String::from("The quick brown fox jumps over the lazy dog");
    let excerpt = Excerpt::new(&novel);
    println!("Text:       {}", excerpt.text);
    println!("Word count: {}", excerpt.word_count());
    println!("First word: {}", excerpt.first_word());

    // With a string literal
    let excerpt2 = Excerpt::new("Hello world");
    println!("\nText:       {}", excerpt2.text);
    println!("Word count: {}", excerpt2.word_count());
    println!("First word: {}", excerpt2.first_word());
}
```

### A10 — Key-Value pair
```rust
struct KeyValue<'a> {
    key: &'a str,
    value: &'a str,
}

impl<'a> KeyValue<'a> {
    fn new(key: &'a str, value: &'a str) -> Self {
        KeyValue { key, value }
    }

    fn format(&self) -> String {
        format!("{}={}", self.key, self.value)
    }
}

fn main() {
    let pairs = vec![
        KeyValue::new("name", "Alice"),
        KeyValue::new("age", "30"),
        KeyValue::new("lang", "Rust"),
    ];

    for pair in &pairs {
        println!("{}", pair.format());
    }
}
```

### A11 — Independent lifetimes
```rust
struct Comparison<'a, 'b> {
    left: &'a str,
    right: &'b str,
}

impl<'a, 'b> Comparison<'a, 'b> {
    fn get_left(&self) -> &'a str {
        self.left
    }
}

fn main() {
    let long_lived = String::from("I live long");
    let result;

    {
        let short_lived = String::from("I'm temporary");
        let cmp = Comparison {
            left: &long_lived,
            right: &short_lived,
        };
        result = cmp.get_left();  // only depends on 'a
        println!("Right: {}", cmp.right);
    }
    // short_lived dropped here, but result doesn't care

    println!("Left survived: {result}");  // ✅ long_lived is alive
}
```

---

## Section D

### A12 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | The compiler needs to know how long the referenced data lives |
| 2 | **False** | Methods can return any lifetime; elision often infers `&self`'s lifetime |
| 3 | **False** | `T: 'static` means T contains no non-static references; `String`, `i32`, `Vec<i32>` all satisfy it |
| 4 | **False** | `'a: 'b` means `'a` **outlives** `'b` (i.e., `'a` is longer or equal) |
| 5 | **True** | Lifetimes are only needed when the struct holds references |
| 6 | **True** | Elision Rule 3: if there's a `&self` parameter, its lifetime is assigned to output references |
| 7 | **False** | String literals, constants, and owned types also provide `'static` data |

### A13 — Explanation
`"MyApp"` is a **string literal**, which has type `&'static str`. Since `'static` outlives any lifetime `'a`, it satisfies the `&'a str` requirement. The literal is embedded in the program binary and lives for the entire execution — no separate `String` allocation needed.

### A14 — Design decision
**Use owned data (`String`)** for all three fields:

1. **Stored in `Vec<LogEntry>`**: If entries are stored in a collection, they need to own their data. References would require all source data to outlive the `Vec`, which is impractical.
2. **Outliving source data**: Log messages are often constructed from `format!()` or received from network — the source data is temporary. Owned `String` fields don't have this problem.
3. **Performance**: The slight cost of allocating `String` fields is negligible compared to the I/O cost of logging. Correctness and simplicity far outweigh micro-optimization here.

```rust
struct LogEntry {
    level: String,      // or an enum Level { Info, Error, ... }
    message: String,
    timestamp: String,  // or chrono::DateTime
}
```

Better yet, use an enum for `level` (zero allocation, exhaustive matching):
```rust
enum Level { Info, Warn, Error }

struct LogEntry {
    level: Level,
    message: String,
    timestamp: u64,  // unix timestamp
}
```

---

## 🏆 Lesson 31 Complete!

✅ Lifetimes in struct definitions  
✅ `impl` blocks with lifetime parameters  
✅ Multiple lifetimes in structs  
✅ Lifetime bounds on generics (`T: 'a`, `T: 'static`)  
✅ `'static` in depth  
✅ Lifetime subtyping (`'a: 'b`)  
✅ Common patterns: Config, Parser, SplitResult  
✅ When to avoid lifetimes  

**Next up:** [Lesson 32 — HashSet, BTreeMap, BTreeSet](../lesson_32_hashset_btree/lesson_32_hashset_btree.md) 🦀
