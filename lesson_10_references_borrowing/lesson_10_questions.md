# 🧪 Lesson 10 — Questions: References & Borrowing

> Answers in [`lesson_10_answers.md`](./lesson_10_answers.md)

---

## Section A — Will It Compile?

### Q1
```rust
fn main() {
    let s = String::from("hello");
    let r1 = &s; let r2 = &s; let r3 = &s;
    println!("{r1} {r2} {r3} {s}");
}
```

### Q2
```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &mut s;
    let r2 = &mut s;
    println!("{r1} {r2}");
}
```

### Q3
```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &s; let r2 = &s;
    println!("{r1} {r2}"); // last use of r1, r2
    let r3 = &mut s;
    r3.push_str(", world");
    println!("{r3}");
}
```

### Q4
```rust
fn dangle() -> &String {
    let s = String::from("hello");
    &s
}
fn main() { let _r = dangle(); }
```

### Q5
```rust
fn main() {
    let mut data = vec![1, 2, 3];
    let first = &data[0];
    data.push(4);
    println!("{first}");
}
```

### Q6
```rust
fn double(s: &mut String) {
    let copy = s.clone();
    s.push_str(&copy);
}
fn main() {
    let mut word = String::from("hi");
    double(&mut word);
    println!("{word}");
}
```

---

## Section B — Predict the Output

### Q7
```rust
fn add_exclamation(s: &mut String) { s.push('!'); }
fn main() {
    let mut msg = String::from("hello");
    add_exclamation(&mut msg);
    add_exclamation(&mut msg);
    println!("{msg}");
}
```

### Q8
```rust
fn main() {
    let s = String::from("hello world");
    let word = &s[0..5];
    println!("{word}");
    println!("{s}");
}
```

### Q9
```rust
fn sum(data: &[i32]) -> i32 { data.iter().sum() }
fn main() {
    let arr = [1, 2, 3, 4, 5];
    let vec = vec![10, 20, 30];
    println!("{}", sum(&arr));
    println!("{}", sum(&vec));
    println!("{}", sum(&arr[1..4]));
}
```

---

## Section C — Fix the Bugs

### Q10
```rust
fn main() {
    let mut s = String::from("hello");
    let r = &s;
    r.push_str(" world"); // trying to modify through immutable ref
    println!("{s}");
}
```

### Q11
```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &s;
    let r2 = &mut s;
    println!("{r1} {r2}");
}
```

### Q12
```rust
fn get_first(v: &Vec<i32>) -> &i32 { &v[0] }
fn main() {
    let mut numbers = vec![10, 20, 30];
    let first = get_first(&numbers);
    numbers.push(40);   // push while borrowing
    println!("{first}");
}
```

### Q13
```rust
fn print_length(s: String) { println!("Length: {}", s.len()); }
fn main() {
    let s = String::from("hello");
    print_length(s);
    print_length(s);
    println!("{s}");
}
```

---

## Section D — Write It Yourself

### Q14 — Read without consuming
Write `fn count_uppercase(s: &str) -> usize`. In `main`, call it and show the String is still usable after.

### Q15 — Modify in place
Write `fn remove_spaces(s: &mut String)` that removes all spaces in place. Show before/after in `main`.

### Q16 — Return a slice
Write `fn first_word(s: &str) -> &str` returning the first word as a slice. Works on both `&String` and `&str`.

### Q17 — Multiple borrows
Write `fn stats(data: &[f64]) -> (f64, f64, f64)` returning (min, max, mean). Call it on different slices of the same Vec.

### Q18 — Mixed borrows
Write `fn replace_first(v: &mut Vec<String>, new_val: &str)` that replaces `v[0]` with `new_val`.

### Q19 — Struct references
```rust
struct Student { name: String, grade: u32 }
```
Write:
- `fn get_name(s: &Student) -> &str`
- `fn promote(s: &mut Student)`

### Q20 — Full analysis program
Using only `&str` parameters (no ownership), write:
- `fn word_count(s: &str) -> usize`
- `fn unique_word_count(s: &str) -> usize` (case-insensitive)
- `fn longest_word<'a>(s: &'a str) -> &'a str`
- `fn avg_word_len(s: &str) -> f64`

Test on: `"the quick brown fox jumps over the lazy dog"`

---

## Section E — Deep Understanding

### Q21 — True or False?
1. You can have unlimited simultaneous `&T` references.
2. A `&mut T` can be used while `&T` references to the same value exist.
3. `&str` owns its string data.
4. Rust's borrow checker operates at runtime.
5. A function returning `&String` to a locally created String is valid.
6. Non-Lexical Lifetimes means borrows can end before the closing `}`.
7. `&Vec<i32>` automatically coerces to `&[i32]`.
8. You can create `&mut T` from an immutable variable.

### Q22 — Trace the borrows
State where each borrow starts/ends and whether the program compiles:
```rust
fn main() {
    let mut v = vec![1, 2, 3];
    let a = &v;
    println!("{a:?}");     // last use of a
    let b = &mut v;
    b.push(4);
    println!("{b:?}");     // last use of b
    let c = &v;
    let d = &v;
    println!("{c:?} {d:?}");
}
```

### Q23 — Big Debug Challenge (8 bugs)
```rust
fn first_word(s: String) -> &str {   // bug 1
    match s.find(' ') {
        Some(i) => &s[..i],
        None => &s[..]
    }
}

fn append_all(parts: Vec<String>) -> String { // bug 2
    let mut result = String::new();
    for part in parts {
        result.push_str(part);  // bug 3: wrong type
    }
    result
}

fn main() {
    let sentence = String::from("hello world");
    let word = first_word(sentence);
    println!("{sentence}");    // bug 4
    println!("{word}");

    let items = vec![String::from("a"), String::from("b")];
    let combined = append_all(items);
    println!("{items:?}");     // bug 5

    let mut s = String::from("hello");
    let r1 = &s;
    let r2 = &mut s;           // bug 6
    println!("{r1} {r2}");

    let r3 = &mut s;
    let r4 = &mut s;           // bug 7
    println!("{r3} {r4}");

    let dangling = {
        let temp = String::from("temp");
        &temp                   // bug 8
    };
    println!("{dangling}");
}
```
