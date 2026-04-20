# ✅ Lesson 10 — Answers: References & Borrowing

---

## Section A — Will It Compile?

### A1 — Yes ✅

Multiple immutable references simultaneously — allowed. All of `r1`, `r2`, `r3`, and `s` are valid.

### A2 — No ❌

Two simultaneous mutable borrows of `s` — data race risk. Fix:

```rust
fn main() {
    let mut s = String::from("hello");
    { let r1 = &mut s; r1.push_str(" world"); println!("{r1}"); }
    let r2 = &mut s; // r1 gone — now fine
    r2.push_str("!");
    println!("{r2}");
}
```

### A3 — Yes ✅ (NLL)

After the `println!("{r1} {r2}")`, the immutable borrows end (last use). The `&mut` borrow is then allowed.

### A4 — No ❌

Returns a reference to `s` which is dropped at function return — dangling. Fix: return `String`.

### A5 — No ❌

`&data[0]` immutably borrows `data`. `data.push(4)` requires mutable borrow — conflict. Fix: copy the value: `let first = data[0];`

### A6 — Yes ✅

`s.clone()` creates an independent `String`. No borrow conflict. Output: `"hihi"`.

---

## Section B — Predict the Output

### A7

`hello!!` — two pushes of `'!'`.

### A8

```
hello
hello world
```

Slice borrows `s` — `s` remains valid.

### A9

```
15
60
9
```

sum of arr=15, sum of vec=60, sum of arr[1..4]=2+3+4=9.

---

## Section C — Fix the Bugs

### A10

```rust
fn main() {
    let mut s = String::from("hello");
    let r = &mut s; // fix: mutable reference
    r.push_str(" world");
    println!("{s}");
}
```

### A11

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &s;
    println!("{r1}"); // end immutable borrow here (NLL)
    let r2 = &mut s;  // now safe
    r2.push_str(" world");
    println!("{r2}");
}
```

### A12

```rust
fn main() {
    let mut numbers = vec![10, 20, 30];
    let first = numbers[0]; // i32 is Copy — no borrow needed
    numbers.push(40);       // no live borrow — fine
    println!("{first}");    // 10
}
```

### A13

```rust
fn print_length(s: &str) { println!("Length: {}", s.len()); }
fn main() {
    let s = String::from("hello");
    print_length(&s);
    print_length(&s);
    println!("{s}");
}
```

---

## Section D — Write It Yourself

### A14 — count_uppercase

```rust
fn count_uppercase(s: &str) -> usize {
    s.chars().filter(|c| c.is_uppercase()).count()
}
fn main() {
    let text = String::from("Hello World RUST");
    println!("{}", count_uppercase(&text)); // 6
    println!("{text}");                     // still valid
}
```

### A15 — remove_spaces

```rust
fn remove_spaces(s: &mut String) { s.retain(|c| c != ' '); }
fn main() {
    let mut s = String::from("hello world foo");
    println!("Before: {s}");
    remove_spaces(&mut s);
    println!("After:  {s}"); // helloworldfoo
}
```

### A16 — first_word

```rust
fn first_word(s: &str) -> &str {
    match s.find(' ') { Some(i) => &s[..i], None => s }
}
fn main() {
    let owned = String::from("hello world");
    let literal = "rust programming";
    println!("{}", first_word(&owned));   // hello
    println!("{}", first_word(literal));  // rust
    println!("{owned}");                  // still valid
}
```

### A17 — stats

```rust
fn stats(data: &[f64]) -> (f64, f64, f64) {
    let min  = data.iter().cloned().fold(f64::INFINITY, f64::min);
    let max  = data.iter().cloned().fold(f64::NEG_INFINITY, f64::max);
    let mean = data.iter().sum::<f64>() / data.len() as f64;
    (min, max, mean)
}
fn main() {
    let data = vec![3.0, 1.0, 4.0, 1.0, 5.0, 9.0];
    let (mn, mx, avg) = stats(&data);
    println!("min={mn}, max={mx}, mean={avg:.2}");
    let (mn2, mx2, avg2) = stats(&data[..3]); // sub-slice
    println!("first 3: min={mn2}, max={mx2}, mean={avg2:.2}");
}
```

### A18 — replace_first

```rust
fn replace_first(v: &mut Vec<String>, new_val: &str) {
    if !v.is_empty() { v[0] = new_val.to_string(); }
}
fn main() {
    let mut names = vec![String::from("Alice"), String::from("Bob")];
    println!("Before: {names:?}");
    replace_first(&mut names, "Zara");
    println!("After:  {names:?}");
}
```

### A19 — Struct references

```rust
struct Student { name: String, grade: u32 }
fn get_name(s: &Student) -> &str { &s.name }
fn promote(s: &mut Student) { s.grade += 1; }
fn main() {
    let mut student = Student { name: String::from("Alice"), grade: 9 };
    println!("Name:  {}", get_name(&student));
    println!("Grade: {}", student.grade);
    promote(&mut student);
    println!("After: grade={}, name={}", student.grade, get_name(&student));
}
```

### A20 — Full analysis program

```rust
fn word_count(s: &str) -> usize { s.split_whitespace().count() }

fn unique_word_count(s: &str) -> usize {
    let mut words: Vec<String> = s.split_whitespace()
        .map(|w| w.to_lowercase()).collect();
    words.sort(); words.dedup();
    words.len()
}

fn longest_word<'a>(s: &'a str) -> &'a str {
    s.split_whitespace().max_by_key(|w| w.len()).unwrap_or("")
}

fn avg_word_len(s: &str) -> f64 {
    let words: Vec<&str> = s.split_whitespace().collect();
    if words.is_empty() { return 0.0; }
    words.iter().map(|w| w.len()).sum::<usize>() as f64 / words.len() as f64
}

fn main() {
    let text = "the quick brown fox jumps over the lazy dog";
    println!("Words:        {}", word_count(text));
    println!("Unique:       {}", unique_word_count(text));
    println!("Longest:      {}", longest_word(text));
    println!("Avg length:   {:.2}", avg_word_len(text));
}
// Words: 9, Unique: 8, Longest: quick, Avg: 3.89
```

---

## Section E — Deep Understanding

### A21 — True or False?

| #   | Answer    | Explanation                                                  |
| --- | --------- | ------------------------------------------------------------ |
| 1   | **True**  | Any number of `&T` — read-only, no conflict                  |
| 2   | **False** | `&mut T` while `&T` exists in scope — forbidden              |
| 3   | **False** | `&str` borrows data; it does NOT own it                      |
| 4   | **False** | Borrow checker is compile-time only                          |
| 5   | **False** | Local dropped at return — dangling reference — compile error |
| 6   | **True**  | NLL: borrows end at last use, not at `}`                     |
| 7   | **True**  | Deref coercion: `&Vec<i32>` → `&[i32]`                       |
| 8   | **False** | `&mut T` requires the variable to be declared `mut`          |

### A22 — Trace the borrows

```
a = &v          → immutable borrow starts
println!(a)     → last use of a → borrow ENDS (NLL)
b = &mut v      → mutable borrow starts (safe — a ended)
b.push(4)       → mutable use
println!(b)     → last use of b → mutable borrow ENDS
c = &v          → immutable borrow (safe — b ended)
d = &v          → second immutable borrow (fine)
println!(c, d)  → both end
```

**Compiles: Yes ✅** — NLL handles this cleanly.

### A23 — Big Debug Challenge (8 bugs)

```rust
fn first_word(s: &str) -> &str {         // fix 1: borrow, not own
    match s.find(' ') { Some(i) => &s[..i], None => s }
}

fn append_all(parts: &[String]) -> String { // fix 2: borrow slice
    let mut result = String::new();
    for part in parts {
        result.push_str(part);           // fix 3: &String coerces to &str
    }
    result
}

fn main() {
    let sentence = String::from("hello world");
    let word = first_word(&sentence);    // fix 4: pass reference
    println!("{sentence}");              // ✅
    println!("{word}");

    let items = vec![String::from("a"), String::from("b")];
    let combined = append_all(&items);   // fix 5: pass slice
    println!("{items:?}");               // ✅
    println!("{combined}");

    let mut s = String::from("hello");
    let r1 = &s;
    println!("{r1}");                    // fix 6: use r1 before &mut
    let r2 = &mut s;                     // r1 ended — now safe
    r2.push_str(" world");
    println!("{r2}");

    {
        let r3 = &mut s;
        r3.push_str("!");
        println!("{r3}");                // fix 7: sequential scopes
    }
    {
        let r4 = &mut s;
        println!("{r4}");
    }

    let owned = {
        let temp = String::from("temp");
        temp                             // fix 8: move out, not borrow
    };
    println!("{owned}");
}
```

---

## 🏆 Lesson Complete!

✅ References `&T` and mutable references `&mut T`
✅ Borrowing rules: multiple `&T` OR one `&mut T`
✅ Borrow checker — compile-time, zero runtime cost
✅ Non-Lexical Lifetimes — borrows end at last use
✅ Dangling references — Rust prevents at compile time
✅ `&str` and `&[T]` as the idiomatic slice types
✅ Deref coercion chains
✅ Practical patterns: inspect, modify in place, return slices

**Up next:** [Lesson 11 — Slices](../lesson_11_slices/lesson_11_slices.md) 🦀
