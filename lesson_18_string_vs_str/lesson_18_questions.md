# 🧪 Lesson 18 — Questions: String vs `&str` (C1)

> **Lesson:** [lesson_18_string_vs_str.md](./lesson_18_string_vs_str.md)  
> **Answers:** [lesson_18_answers.md](./lesson_18_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let mut s = String::from("hello");
    s.push_str(", world");
    s.push('!');
    println!("{s}");
    println!("{}", s.len());
}
```

### Q2
```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("Rust!");
    let s3 = s1 + &s2;
    println!("{s3}");
    println!("{s2}");
    // println!("{s1}");  // what would happen here?
}
```

### Q3
```rust
fn main() {
    let s = "café";
    println!("{}", s.len());
    println!("{}", s.chars().count());
    let upper: String = s.chars().map(|c| c.to_uppercase().next().unwrap()).collect();
    println!("{upper}");
}
```

### Q4
```rust
fn count_words(s: &str) -> usize { s.split_whitespace().count() }

fn main() {
    let owned = String::from("one two three four");
    let literal = "five six";
    println!("{}", count_words(&owned));
    println!("{}", count_words(literal));
    println!("{owned}");
}
```

### Q5
```rust
fn main() {
    let s = String::from("  Hello, World!  ");
    let trimmed = s.trim();
    println!("{trimmed}");
    println!("{}", trimmed.to_lowercase());
    println!("{}", s.contains("World"));
    println!("{}", s.replace("World", "Rust").trim().to_string());
}
```

---

## Section B — Will It Compile?

### Q6
```rust
fn main() {
    let s = String::from("hello");
    let c = s[0];  // index by integer
    println!("{c}");
}
```

### Q7
```rust
fn greet(name: &String) { println!("Hello, {name}!"); }

fn main() {
    let owned = String::from("Alice");
    let literal = "Bob";
    greet(&owned);
    greet(literal);
}
```

### Q8
```rust
fn main() {
    let mut s = String::from("hello");
    let extra = String::from(" world");
    s.push_str(&extra);
    println!("{extra}");  // still valid?
    println!("{s}");
}
```

### Q9
```rust
fn main() {
    let s1 = String::from("Hello");
    let s2 = String::from(" World");
    let s3 = s1 + &s2;
    println!("{s1}");  // s1 after + operator
}
```

### Q10
```rust
fn main() {
    let emoji = "😀hi";
    let slice = &emoji[0..4];  // emoji is 4 bytes
    println!("{slice}");
}
```

---

## Section C — Fix the Bugs

### Q11 — Wrong parameter type
```rust
fn shout(s: &String) -> String { s.to_uppercase() }

fn main() {
    let owned = String::from("hello");
    let result = shout(&owned);
    println!("{result}");
    println!("{}", shout("world")); // ERROR: "world" is &str
}
```

### Q12 — Use after move
```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("World");
    let s3 = s1 + &s2;
    println!("{s1}");  // BUG: s1 moved
    println!("{s3}");
}
```

### Q13 — Byte index panic
```rust
fn first_char(s: &str) -> char {
    s[0..1].chars().next().unwrap()  // BUG: unsafe for multi-byte chars
}
fn main() {
    println!("{}", first_char("hello"));  // 'h' — works by luck
    println!("{}", first_char("😀hi"));  // PANIC — 😀 is 4 bytes
}
```
Fix to safely get the first character.

### Q14 — Inefficient string building
```rust
fn build_list(items: &[&str]) -> String {
    let mut result = String::new();
    for item in items {
        result = result + item + ", ";  // BUG: repeated moves + allocations
    }
    result
}
fn main() {
    println!("{}", build_list(&["one", "two", "three"]));
}
```
Refactor to use `push_str` efficiently.

---

## Section D — Write It Yourself

### Q15 — Roadmap Practice Task: Sentence joiner + `push_str` vs `+`
Build two versions of a sentence joiner:
- `fn join_push_str(words: &[&str]) -> String` — using `push_str`
- `fn join_format(words: &[&str]) -> String` — using `format!`

Both join words with `" "`. Then write a comment explaining why `push_str` is more efficient than chaining `+`.

Also write `fn join_with(words: &[&str], sep: &str) -> String` that joins with any separator.

### Q16 — String inspector
Write `fn inspect(s: &str)` that prints:
- Byte length
- Character count
- Word count
- Whether it's empty
- First character (or "empty" if none)
- Last character (or "empty" if none)
- Whether it's a palindrome (character-level, ignoring case and spaces)

### Q17 — String transformer
Write these functions (all returning `String`):
- `fn title_case(s: &str) -> String` — first letter of each word uppercase, rest lower
- `fn snake_to_camel(s: &str) -> String` — `"hello_world"` → `"HelloWorld"`
- `fn count_vowels(s: &str) -> usize` — count a,e,i,o,u (case-insensitive)
- `fn reverse_words(s: &str) -> String` — `"hello world"` → `"world hello"`

### Q18 — CSV parser
Write `fn parse_csv(input: &str) -> Vec<Vec<&str>>` — splits on newlines then commas. All returned `&str` values are slices of the input (no allocation for the strings themselves, only the Vec structure).

Test with:
```
"name,age,city\nAlice,30,NYC\nBob,25,LA"
```

### Q19 — Full program: sentence joiner + word frequency
Read this string:
```
"the quick brown fox jumps over the lazy dog the fox"
```
Print:
1. Total words
2. Unique words (sorted)
3. Most frequent word and its count
4. Title-cased version of the string
5. Reversed string (character-level)

---

## Section E — Deep Understanding

### Q20 — True or False?
1. `String` and `&str` both store UTF-8 encoded text.
2. `s.len()` on a String returns the number of characters.
3. You can index a String with `s[0]` to get the first byte.
4. `&String` automatically coerces to `&str` in function calls.
5. `push_str` takes ownership of the argument string.
6. `s1 + &s2` moves `s1` and borrows `s2`.
7. `format!("{s1}{s2}")` moves both `s1` and `s2`.
8. A string literal like `"hello"` has type `&'static str`.
9. `to_string()` and `to_owned()` both produce the same result for `&str` input.
10. `"café".len()` equals `"café".chars().count()`.

### Q21 — Explain the output
```rust
fn main() {
    let mut s = String::with_capacity(10);
    println!("len={} cap={}", s.len(), s.capacity());
    s.push_str("hello");
    println!("len={} cap={}", s.len(), s.capacity());
    s.push_str(" world!");
    println!("len={} cap={}", s.len(), s.capacity());
}
```
Predict all three lines of output and explain what happened to capacity on the third push.
