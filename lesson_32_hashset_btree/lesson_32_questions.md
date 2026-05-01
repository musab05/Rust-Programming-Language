# 🧪 Lesson 32 — Questions: HashSet, BTreeMap, BTreeSet (C4)

> **Lesson:** [lesson_32_hashset_btree.md](./lesson_32_hashset_btree.md)  
> **Answers:** [lesson_32_answers.md](./lesson_32_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use std::collections::HashSet;

fn main() {
    let mut s = HashSet::new();
    println!("{}", s.insert(1));
    println!("{}", s.insert(2));
    println!("{}", s.insert(1));
    println!("len = {}", s.len());
}
```

### Q2
```rust
use std::collections::BTreeSet;

fn main() {
    let set: BTreeSet<i32> = [5, 3, 1, 4, 2].into_iter().collect();
    for val in &set {
        print!("{val} ");
    }
}
```

### Q3
```rust
use std::collections::HashSet;

fn main() {
    let a: HashSet<i32> = [1, 2, 3].into_iter().collect();
    let b: HashSet<i32> = [2, 3, 4].into_iter().collect();
    let result: HashSet<_> = a.intersection(&b).copied().collect();
    println!("{:?}", result);
}
```

### Q4
```rust
use std::collections::BTreeMap;

fn main() {
    let mut m = BTreeMap::new();
    m.insert("banana", 3);
    m.insert("apple", 5);
    m.insert("cherry", 1);
    for (k, v) in &m {
        println!("{k}: {v}");
    }
}
```

---

## Section B — Choose the Right Collection

For each scenario, choose the best collection and explain why.

### Q5
You need to store unique usernames and quickly check if a username is taken.

### Q6
You need a leaderboard that always shows scores from highest to lowest.

### Q7
You need to count how many times each word appears in a book and then list them alphabetically.

### Q8
You need to quickly look up a student's grade by their ID number.

---

## Section C — Write It Yourself

### Q9 — Duplicate finder (Roadmap Practice Task)
Write a function `find_duplicates(items: &[i32]) -> Vec<i32>` that returns all values that appear more than once. Test with `[1, 5, 3, 2, 5, 1, 8, 3, 9, 1]`.

### Q10 — Set operations
Given two arrays of student names:
- Class A: ["Alice", "Bob", "Charlie", "Diana"]
- Class B: ["Charlie", "Diana", "Eve", "Frank"]

Use HashSet to find:
1. Students in both classes
2. Students only in Class A
3. All unique students across both classes

### Q11 — Word frequency (sorted)
Write a function that takes a `&str`, splits it into words, counts each word's frequency, and prints the results sorted alphabetically. Use `BTreeMap`.

### Q12 — Range query
Create a `BTreeMap<u32, String>` representing products with IDs. Use `.range()` to find all products with IDs between 100 and 200.

---

## Section D — Deep Understanding

### Q13 — True or False?
1. `HashSet` iteration order is guaranteed to be insertion order.
2. `BTreeMap` requires keys to implement `Ord`.
3. `HashSet::insert` returns `true` if the value was already present.
4. `BTreeSet` supports range queries but `HashSet` does not.
5. You can use `f64` as a key in a `HashMap`.
6. Set operations like `union` and `intersection` are available on both `HashSet` and `BTreeSet`.

### Q14 — When would you prefer BTreeMap over HashMap?
Give three specific scenarios where `BTreeMap` is the better choice.

---

*Unique values, sorted keys, and set operations — collections that go beyond Vec! 🦀*
