# 🧪 Lesson 35 — Questions: Collecting & FromIterator (C7)

> **Lesson:** [lesson_35_collecting.md](./lesson_35_collecting.md)  
> **Answers:** [lesson_35_answers.md](./lesson_35_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let s: String = vec!['R', 'u', 's', 't'].into_iter().collect();
    println!("{s}");
}
```

### Q2
```rust
use std::collections::HashSet;
fn main() {
    let v = vec![1, 2, 2, 3, 3, 3];
    let s: HashSet<_> = v.into_iter().collect();
    println!("{}", s.len());
}
```

### Q3
```rust
fn main() {
    let r: Result<Vec<i32>, _> = vec!["1", "2", "3"]
        .iter().map(|s| s.parse::<i32>()).collect();
    println!("{:?}", r);
}
```

### Q4
```rust
fn main() {
    let r: Result<Vec<i32>, _> = vec!["1", "bad", "3"]
        .iter().map(|s| s.parse::<i32>()).collect();
    println!("{}", r.is_err());
}
```

---

## Section B — Write It Yourself

### Q5 — Collect into HashMap
Create a `HashMap<char, usize>` counting character frequency in the string `"hello world"` (skip spaces).

### Q6 — Collect Results (Roadmap Practice Task)
Write a function `parse_all(items: &[&str]) -> Result<Vec<f64>, std::num::ParseFloatError>` that parses all strings to `f64`. Test with a valid list and one with an invalid entry.

### Q7 — Partition
Given `vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10]`, partition into numbers divisible by 3 and the rest.

### Q8 — Custom FromIterator
Create a `struct CsvRow` that holds a `Vec<String>`. Implement `FromIterator<String>` for it. Demonstrate collecting strings into a `CsvRow`.

---

## Section C — Deep Understanding

### Q9 — True or False?
1. `collect()` can only produce `Vec`.
2. Collecting `Iterator<Item=Result<T,E>>` into `Result<Vec<T>, E>` stops at the first `Err`.
3. The turbofish syntax `::<>` is always required with `collect()`.
4. `partition` consumes the iterator and produces two collections.
5. `String` implements `FromIterator<char>`.

### Q10
Why might you prefer `collect::<Result<Vec<_>, _>>()` over manually looping and checking each Result?

---

*collect() transforms any iterator into any collection — pure Rust magic! 🦀*
