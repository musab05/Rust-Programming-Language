# ✅ Lesson 18 — Answers: String vs `&str` (C1)

---

## Section A

### A1: `hello, world!` / `13`
`push_str` appends `", world"` (7 chars), `push` appends `'!'`. Total: 5+7+1 = 13 bytes.

### A2: `Hello, Rust!` / `Rust!`
`+` moves `s1` and borrows `s2`. `s3` = combined. `s2` still valid. `s1` — printing it would be a compile error ("use of moved value").

### A3: `5` / `4` / `CAFÉ`
`"café"` = 4 chars but 5 bytes (`é` = 2 bytes). `.chars().count()` = 4. `to_uppercase` on each char → "CAFÉ".

### A4: `4` / `2` / `one two three four`
`count_words` takes `&str` — deref coercion works for `&owned`. `owned` is only borrowed, still valid afterward.

### A5:
```
Hello, World!
hello, world!
true
Hello, Rust!
```

---

## Section B

### A6 — ❌ No
`s[0]` — integer indexing on a String is a compile error. **Error:** `cannot index into a value of type String`. Use `.chars().nth(0)` or `&s[0..1]`.

### A7 — ❌ No (for `greet(literal)`)
`greet` takes `&String`. `"Bob"` is `&str`, not `&String`. Deref coercion goes `&String → &str` but NOT `&str → &String`. Fix: change param to `&str`.

### A8 — ✅ Yes
`push_str` takes `&str` — borrows `extra`, doesn't consume it. `extra` remains valid. Output: ` world\nhello world`.

### A9 — ❌ No
`s1` is moved into `s3` by the `+` operator. Printing `s1` afterward is "use of moved value". Fix: use `format!` or clone.

### A10 — ✅ Yes
`😀` is exactly 4 bytes in UTF-8. `&emoji[0..4]` is a valid char boundary. Output: `😀`.

---

## Section C

### A11 — Wrong parameter type
```rust
fn shout(s: &str) -> String { s.to_uppercase() }  // &str is more general

fn main() {
    let owned = String::from("hello");
    println!("{}", shout(&owned));   // &String → &str
    println!("{}", shout("world"));  // &str directly
}
```

### A12 — Use after move
```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("World");
    // Fix: use format! instead of + to avoid moving s1
    let s3 = format!("{s1}{s2}");
    println!("{s1}");  // ✅ s1 not moved
    println!("{s3}");
}
```

### A13 — Byte index panic
```rust
fn first_char(s: &str) -> Option<char> {
    s.chars().next()   // safe for any UTF-8 string
}

fn main() {
    println!("{:?}", first_char("hello"));  // Some('h')
    println!("{:?}", first_char("😀hi"));  // Some('😀') — correct!
    println!("{:?}", first_char(""));       // None
}
```

### A14 — Inefficient string building
```rust
fn build_list(items: &[&str]) -> String {
    let mut result = String::new();
    for (i, item) in items.iter().enumerate() {
        if i > 0 { result.push_str(", "); }
        result.push_str(item);
    }
    result
}

fn main() {
    println!("{}", build_list(&["one", "two", "three"])); // one, two, three
}
```
The original used `result = result + item + ", "` — this moves `result` each time, allocating a new String on every iteration. `push_str` modifies in place and only reallocates when capacity is exceeded.

---

## Section D

### A15 — Sentence joiner (roadmap practice task)
```rust
fn join_push_str(words: &[&str]) -> String {
    let mut result = String::new();
    for (i, word) in words.iter().enumerate() {
        if i > 0 { result.push(' '); }
        result.push_str(word);
    }
    result
}

fn join_format(words: &[&str]) -> String {
    words.join(" ")  // std library join — uses format internally
}

fn join_with(words: &[&str], sep: &str) -> String {
    words.join(sep)
}

// Why push_str is more efficient than +:
// - `+` takes ownership of the left String and returns a new one.
//   Each iteration: new_string = old_string + " " + word
//   This potentially allocates a new buffer each time.
// - `push_str` modifies the String in place.
//   Rust only reallocates when capacity is exceeded (amortised O(1) per push).
//   No intermediate String values are created.

fn main() {
    let words = ["the", "quick", "brown", "fox"];
    println!("{}", join_push_str(&words));   // the quick brown fox
    println!("{}", join_format(&words));
    println!("{}", join_with(&words, "-")); // the-quick-brown-fox
}
```

### A16 — String inspector
```rust
fn inspect(s: &str) {
    let trimmed = s.trim();
    let chars: Vec<char> = trimmed.chars().collect();
    let lower_chars: Vec<char> = trimmed.to_lowercase().chars().filter(|c| !c.is_whitespace()).collect();
    let rev_chars: Vec<char> = lower_chars.iter().rev().cloned().collect();
    let is_palindrome = lower_chars == rev_chars;

    println!("=== inspect({:?}) ===", s);
    println!("  Bytes:     {}", s.len());
    println!("  Chars:     {}", s.chars().count());
    println!("  Words:     {}", s.split_whitespace().count());
    println!("  Empty:     {}", s.trim().is_empty());
    println!("  First:     {:?}", s.chars().next().unwrap_or(' '));
    println!("  Last:      {:?}", s.chars().last().unwrap_or(' '));
    println!("  Palindrome:{}", is_palindrome);
}

fn main() {
    inspect("Hello World");
    inspect("racecar");
    inspect("A man a plan a canal Panama");
    inspect("");
}
```

### A17 — String transformers
```rust
fn title_case(s: &str) -> String {
    s.split_whitespace()
        .map(|word| {
            let mut chars = word.chars();
            match chars.next() {
                None => String::new(),
                Some(first) => first.to_uppercase().to_string() + chars.as_str(),
            }
        })
        .collect::<Vec<_>>()
        .join(" ")
}

fn snake_to_camel(s: &str) -> String {
    s.split('_')
        .map(|part| {
            let mut chars = part.chars();
            match chars.next() {
                None => String::new(),
                Some(f) => f.to_uppercase().to_string() + chars.as_str(),
            }
        })
        .collect()
}

fn count_vowels(s: &str) -> usize {
    s.chars().filter(|c| "aeiouAEIOU".contains(*c)).count()
}

fn reverse_words(s: &str) -> String {
    s.split_whitespace().rev().collect::<Vec<_>>().join(" ")
}

fn main() {
    println!("{}", title_case("hello world from rust")); // Hello World From Rust
    println!("{}", snake_to_camel("hello_world_rust"));  // HelloWorldRust
    println!("{}", count_vowels("Hello World"));         // 3
    println!("{}", reverse_words("hello world foo"));    // foo world hello
}
```

### A18 — CSV parser
```rust
fn parse_csv(input: &str) -> Vec<Vec<&str>> {
    input.lines()
        .map(|line| line.split(',').collect())
        .collect()
}

fn main() {
    let csv = "name,age,city\nAlice,30,NYC\nBob,25,LA";
    let rows = parse_csv(csv);
    for row in &rows {
        println!("{:?}", row);
    }
    // ["name", "age", "city"]
    // ["Alice", "30", "NYC"]
    // ["Bob", "25", "LA"]
}
```

### A19 — Full program
```rust
use std::collections::HashMap;

fn title_case(s: &str) -> String {
    s.split_whitespace()
        .map(|w| {
            let mut c = w.chars();
            match c.next() {
                None => String::new(),
                Some(f) => f.to_uppercase().to_string() + c.as_str(),
            }
        })
        .collect::<Vec<_>>()
        .join(" ")
}

fn main() {
    let text = "the quick brown fox jumps over the lazy dog the fox";

    // 1. Total words
    let words: Vec<&str> = text.split_whitespace().collect();
    println!("Words: {}", words.len());

    // 2. Unique words sorted
    let mut unique: Vec<&str> = {
        let mut set: Vec<&str> = words.iter().cloned().collect();
        set.sort(); set.dedup(); set
    };
    println!("Unique: {:?}", unique);

    // 3. Most frequent
    let mut freq: HashMap<&str, usize> = HashMap::new();
    for w in &words { *freq.entry(w).or_insert(0) += 1; }
    let (most, count) = freq.iter().max_by_key(|(_, c)| *c).unwrap();
    println!("Most frequent: '{}' ({}×)", most, count);

    // 4. Title case
    println!("Title: {}", title_case(text));

    // 5. Reversed
    let rev: String = text.chars().rev().collect();
    println!("Reversed: {rev}");
}
```

---

## Section E

### A20 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Both are valid UTF-8 — Rust enforces this at all times |
| 2 | **False** | `.len()` returns **byte** count, not character count |
| 3 | **False** | `s[0]` is a **compile error** — integers cannot index a String |
| 4 | **True** | Deref coercion: `&String` → `&str` in function calls |
| 5 | **False** | `push_str` takes `&str` — borrows, does not take ownership |
| 6 | **True** | `+` signature: `fn add(self, s: &str) -> String` |
| 7 | **False** | `format!` borrows all arguments — nothing is moved |
| 8 | **True** | String literals are embedded in the binary with `'static` lifetime |
| 9 | **True** | Both produce an owned `String` from a `&str` — identical result |
| 10 | **False** | `"café".len()` = 5 (bytes), `.chars().count()` = 4 (characters) |

### A21 — Capacity explanation
```
len=0 cap=10       // created with_capacity(10) — 10 bytes pre-allocated, 0 used
len=5 cap=10       // "hello" is 5 bytes — fits in existing 10-byte buffer, no realloc
len=12 cap=20      // "hello world!" is 12 bytes — exceeds 10, so Rust reallocates
                   // typically doubles: new cap = 10 * 2 = 20 (exact value is implementation-defined)
```
When `push_str(" world!")` would make `len > cap`, Rust allocates a new larger buffer, copies data over, and frees the old buffer. The exact growth factor is implementation-defined but is typically doubling.

---

## 🏆 Lesson 18 Complete!

✅ `&str` — borrowed slice, no allocation, fat pointer  
✅ `String` — owned, heap-allocated, growable  
✅ Creating Strings: `from`, `to_string`, `new`, `with_capacity`, `format!`  
✅ `push_str` / `push` — in-place growth without taking ownership  
✅ `+` operator — moves left, borrows right  
✅ `format!` — build strings without moves, prefer over `+` chains  
✅ No integer indexing — use `.chars()`, `.bytes()`, `.find()`  
✅ UTF-8 safety — byte len ≠ char count  
✅ Deref coercion: `&String → &str`  
✅ Prefer `&str` in function parameters  
✅ Sentence joiner with `push_str` — roadmap practice ✓  

**Up next: [Lesson 19 — Vec\<T\>](../lesson_19_vec/lesson_19_vec.md) 🦀**
