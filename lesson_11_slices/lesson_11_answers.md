# ✅ Lesson 11 — Answers: Slices (O4)

---

## Section A — Predict the Output

### A1
```
hello
world
11
```
`&s[..5]` = "hello", `&s[6..]` = "world", `&s[..]` = full string (len 11).

### A2
```
[20, 30, 40]
5
3
```
`arr.len()` = 5 (original). `mid.len()` = 3 (indices 1,2,3).

### A3
```
15
60
8
```
`sum(&v)` = 1+2+3+4+5 = 15. `sum(&a)` = 10+20+30 = 60. `sum(&v[2..])` = 3+4+5 = 8.

### A4
```
[1, 2, 3] [2, 3, 4] [3, 4, 5] [4, 5, 6]
[1, 2] [3, 4] [5, 6]
```
`windows(3)`: overlapping groups of 3. `chunks(2)`: non-overlapping pairs.

### A5
```
rust
rust programming
```
`first_word` returns a slice up to the first space. The original `s` is still valid — only a read borrow was taken.

---

## Section B — Will It Compile?

### A6 — ✅ Yes
`r = &s[..]` borrows all of `s` as `&str`. After printing `r` (last use of the borrow), `s` is accessible. Output: `hello\nhello`.

### A7 — ❌ No
`r` immutably borrows `s`. Then `s.push_str()` tries a mutable borrow — conflict. **Error:** cannot borrow `s` as mutable because it is also borrowed as immutable. Fix: print `r` before calling `push_str`.

### A8 — ❌ Runtime panic (compiles, panics)
Compiles fine, but panics at runtime: `byte index 2 is not a char boundary`. `😀` is 4 bytes in UTF-8; slicing at byte 2 splits it. Fix: `&emoji[0..4]`.

### A9 — ✅ Yes
`&owned` coerces from `&String` to `&str` via deref coercion. `owned` is only borrowed, so it's still valid after the call. Output: `5\nhello`.

### A10 — ❌ No
Two `&mut` borrows of `arr` (one for `[..2]`, one for `[3..]`). The compiler sees both as `&mut arr` — violation of exclusive borrow rule. Fix: use `arr.split_at_mut(3)` to get non-overlapping mutable halves safely.

---

## Section C — Fix the Bugs

### A11 — Inflexible parameter
```rust
fn word_count(s: &str) -> usize {     // &str instead of &String
    s.split_whitespace().count()
}
fn main() {
    let owned = String::from("one two three");
    let literal = "four five";
    println!("{}", word_count(&owned));   // 3 — &String coerces to &str
    println!("{}", word_count(literal));  // 2 — &str directly
}
```

### A12 — Unicode panic
```rust
fn main() {
    let s = "Héllo";
    // Safe: collect first 3 chars
    let first3: String = s.chars().take(3).collect();
    println!("{first3}"); // "Hél"

    // Alternative: find the byte position of the 4th char boundary
    let byte_end = s.char_indices().nth(3).map(|(i, _)| i).unwrap_or(s.len());
    println!("{}", &s[..byte_end]); // "Hél"
}
```

### A13 — Wrong return from first_word
```rust
fn first_word(s: &str) -> &str {
    // Return a slice of the input directly — no Vec needed
    match s.find(' ') {
        Some(i) => &s[..i],
        None    => s,
    }
}
fn main() {
    let s = String::from("hello world");
    println!("{}", first_word(&s)); // hello
}
```
The original bug: `words` is a local `Vec<&str>`. `words[0]` is a reference into that Vec. When the function returns, the Vec is dropped — dangling reference. The fix avoids the Vec entirely.

### A14 — Out-of-bounds access
```rust
fn third_element(data: &[i32]) -> Option<i32> {
    data.get(2).copied()    // returns None if len < 3
}
fn main() {
    let a = vec![10, 20, 30];
    let b: Vec<i32> = vec![];
    println!("{:?}", third_element(&a)); // Some(30)
    println!("{:?}", third_element(&b)); // None — no panic
}
```

---

## Section D — Write It Yourself

### A15 — `first_word` (roadmap practice task)
```rust
fn first_word(s: &str) -> &str {
    match s.find(' ') {
        Some(i) => &s[..i],
        None    => s,
    }
}

fn main() {
    println!("{}", first_word("rust programming language")); // rust
    println!("{}", first_word("single"));                    // single
    println!("{}", first_word(""));                          // (empty)
}
```

### A16 — `last_word`
```rust
fn last_word(s: &str) -> &str {
    match s.rfind(' ') {
        Some(i) => &s[i + 1..],
        None    => s,
    }
}

fn main() {
    println!("{}", last_word("rust programming language")); // language
    println!("{}", last_word("single"));                    // single
    println!("{}", last_word(""));                          // (empty)
}
```

### A17 — Array statistics
```rust
fn stats(data: &[f64]) -> (f64, f64, f64) {
    if data.is_empty() { return (0.0, 0.0, 0.0); }
    let min  = data.iter().cloned().fold(f64::INFINITY, f64::min);
    let max  = data.iter().cloned().fold(f64::NEG_INFINITY, f64::max);
    let mean = data.iter().sum::<f64>() / data.len() as f64;
    (min, max, mean)
}

fn main() {
    let v = vec![3.0, 1.0, 4.0, 1.0, 5.0, 9.0, 2.0, 6.0];
    let mid = v.len() / 2;

    println!("Full:       {:?}", stats(&v));
    println!("First half: {:?}", stats(&v[..mid]));
    println!("Last half:  {:?}", stats(&v[mid..]));
    println!("Original:   {:?}", v); // unchanged ✅
}
```

### A18 — Count vowels
```rust
fn count_vowels(s: &str) -> usize {
    s.chars().filter(|c| "aeiouAEIOU".contains(*c)).count()
}

fn main() {
    println!("{}", count_vowels("Hello World")); // 3
    println!("{}", count_vowels("rhythm"));      // 0
    println!("{}", count_vowels("AEIOUaeiou"));  // 10
}
```

### A19 — Sliding window max
```rust
fn window_max(data: &[i32], window: usize) -> Vec<i32> {
    if window > data.len() || window == 0 { return vec![]; }
    data.windows(window)
        .map(|w| *w.iter().max().unwrap())
        .collect()
}

fn main() {
    let data = [1, 3, 2, 5, 4, 6];
    println!("{:?}", window_max(&data, 3)); // [3, 5, 5, 6]
    println!("{:?}", window_max(&data, 10)); // []
}
```

### A20 — `split_at_mut` rotate
```rust
fn rotate_left(data: &mut [i32], n: usize) {
    if data.is_empty() { return; }
    let n = n % data.len();
    if n == 0 { return; }
    let (left, right) = data.split_at_mut(n);
    left.reverse();
    right.reverse();
    data.reverse();
}

fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    rotate_left(&mut v, 2);
    println!("{:?}", v); // [3, 4, 5, 1, 2]

    let mut v2 = vec![1, 2, 3, 4, 5];
    rotate_left(&mut v2, 7); // 7 % 5 = 2
    println!("{:?}", v2);    // [3, 4, 5, 1, 2]
}
```

### A21 — Word analysis
```rust
fn main() {
    let text = "rust is a systems programming language";

    // Total characters
    println!("Characters: {}", text.chars().count());

    // Total words
    let words: Vec<&str> = text.split_whitespace().collect();
    println!("Words: {}", words.len());

    // Longest word — slice of input
    let longest = words.iter().max_by_key(|w| w.len()).unwrap_or(&"");
    println!("Longest: {longest}");

    // Palindrome check (ignoring spaces)
    let stripped: String = text.chars().filter(|c| *c != ' ').collect();
    let rev: String = stripped.chars().rev().collect();
    println!("Palindrome: {}", stripped == rev);

    // Sorted words (Vec<&str> — slices of original text, no clone)
    let mut sorted = words.clone();
    sorted.sort();
    println!("Sorted: {:?}", sorted);
}
```

---

## Section E — Deep Understanding

### A22 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `&str` can point to binary (literals), heap (String slice), or stack data |
| 2 | **False** | `.get(5)` returns `None` — it never panics |
| 3 | **True** | Deref coercion: `&Vec<T>` → `&[T]` automatically in function calls |
| 4 | **False** | `.windows(n)` produces **overlapping** sub-slices; `.chunks(n)` is non-overlapping |
| 5 | **True** | Fat pointer = address + length (16 bytes on 64-bit systems) |
| 6 | **False** | `'é'` in "café" is 2 bytes — `[0..3]` would panic at that byte |
| 7 | **True** | `.sort()` requires `T: Ord`; for floats use `.sort_by(|a,b| a.partial_cmp(b).unwrap())` |
| 8 | **True** | `split_at_mut` guarantees non-overlapping parts — both mutable borrows are safe |
| 9 | **True** | `&s[..]` = full string slice = `s.as_str()` |
| 10 | **False** | Lifetime annotations describe relationships — they can't save a reference to dropped data |

### A23 — Debug challenge (5 bugs)
```rust
// Bug 1: parameter should be &str (more flexible) and return &str
fn longest_word<'a>(sentence: &'a str) -> &'a str {
    sentence
        .split_whitespace()
        .max_by_key(|w| w.len())
        .unwrap_or("")   // Bug 2 fixed: "" is &str ✅ (was trying to return &String)
}

fn double_slice(data: &[i32]) -> Vec<i32> {
    let mut out = Vec::new();
    for i in 0..data.len() {    // Bug 3 fixed: removed `+ 1` (off-by-one)
        out.push(data[i] * 2);
    }
    out
}

fn main() {
    let s = String::from("the quick brown fox");
    println!("{}", longest_word(&s)); // "quick" ... actually "brown" (5 chars)

    let v = vec![1, 2, 3];
    let doubled = double_slice(&v);
    println!("{:?}", doubled); // [2, 4, 6]

    let emoji_s = "😀bye";
    let part = &emoji_s[4..];       // Bug 4 fixed: 😀 is 4 bytes, start at 4
    println!("{part}");             // "bye"

    let mut arr = [5, 3, 1, 4, 2];
    let (left, right) = arr.split_at_mut(2);
    left.sort();
    right.sort();
    // Bug 5: split-sort gives [3,5] and [1,2,4] — not a fully sorted array
    // Fix: sort the whole array at once:
    arr.sort();
    println!("{:?}", arr); // [1, 2, 3, 4, 5]
}
```

---

## 🏆 Lesson 11 Complete!

✅ String slices `&str` — pointer + length into UTF-8 data  
✅ Array/Vec slices `&[T]` — pointer + length into T data  
✅ Range syntax: `[..n]`, `[n..]`, `[a..b]`, `[..]`  
✅ Fat pointers — why slices are 16 bytes  
✅ Deref coercion: `&String→&str`, `&Vec<T>→&[T]`  
✅ Prefer `&str` / `&[T]` in function parameters  
✅ Unicode safety — byte vs character indexing  
✅ Key methods: `.get()`, `.windows()`, `.chunks()`, `.binary_search()`  
✅ `split_at_mut` for two simultaneous mutable sub-slices  
✅ Slice pattern matching  

**Up next: [Lesson 12 — Structs](../lesson_12_structs/lesson_12_structs.md) 🦀**
