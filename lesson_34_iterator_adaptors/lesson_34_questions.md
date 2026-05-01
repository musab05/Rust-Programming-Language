# 🧪 Lesson 34 — Questions: Iterator Adaptors (C6)

> **Lesson:** [lesson_34_iterator_adaptors.md](./lesson_34_iterator_adaptors.md)  
> **Answers:** [lesson_34_answers.md](./lesson_34_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let r: Vec<i32> = v.iter().filter(|&&x| x > 3).map(|x| x * 10).collect();
    println!("{:?}", r);
}
```

### Q2
```rust
fn main() {
    let v = vec!["hello world", "foo bar"];
    let words: Vec<&str> = v.iter().flat_map(|s| s.split_whitespace()).collect();
    println!("{:?}", words);
}
```

### Q3
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5, 6, 7, 8];
    let page: Vec<_> = v.iter().skip(2).take(3).collect();
    println!("{:?}", page);
}
```

### Q4
```rust
fn main() {
    let a = vec![1, 2, 3];
    let b = vec![10, 20, 30];
    let sum: i32 = a.iter().zip(b.iter()).map(|(x, y)| x + y).sum();
    println!("{sum}");
}
```

---

## Section B — Write It Yourself

### Q5 — Parse and sum
Given `vec!["10", "abc", "20", "xyz", "30"]`, use `filter_map` to parse valid numbers and compute their sum.

### Q6 — Loop-free pipeline (Roadmap Practice Task)
Given student records, use only iterator methods to:
1. Filter students with grades >= 70
2. Convert names to uppercase
3. Sort alphabetically
4. Number each entry
Print the results.

```rust
let students = vec![
    ("Alice", 92), ("Bob", 65), ("Charlie", 78),
    ("Diana", 55), ("Eve", 88), ("Frank", 71),
];
```

### Q7 — Flatten and deduplicate
Given nested groups, flatten them and collect unique values in sorted order:
```rust
let groups = vec![vec![3, 1, 4], vec![1, 5, 9], vec![2, 6, 5]];
```

### Q8 — Windowed pairs
Using `zip`, create pairs of consecutive elements from `[1, 2, 3, 4, 5]` → `[(1,2), (2,3), (3,4), (4,5)]`.

---

## Section C — Deep Understanding

### Q9 — True or False?
1. `take_while` consumes the entire iterator even after the predicate returns false.
2. `zip` stops at the shorter of the two iterators.
3. `flat_map(f)` is equivalent to `.map(f).flatten()`.
4. `inspect` modifies the elements passing through it.
5. `chain` requires both iterators to have the same `Item` type.

### Q10 — Explain the difference
What is the difference between these two?
```rust
// Version A
v.iter().filter(|x| cond(x)).map(|x| transform(x))
// Version B
v.iter().filter_map(|x| if cond(x) { Some(transform(x)) } else { None })
```

---

*Compose adaptors like LEGO blocks to build powerful pipelines! 🦀*
