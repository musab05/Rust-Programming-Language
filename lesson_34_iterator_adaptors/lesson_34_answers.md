# ✅ Lesson 34 — Answers: Iterator Adaptors (C6)

---

## Section A

### A1
```
[40, 50]
```
`filter` keeps 4 and 5, `map` multiplies by 10.

### A2
```
["hello", "world", "foo", "bar"]
```
`flat_map` splits each string and flattens all words.

### A3
```
[3, 4, 5]
```
`skip(2)` skips 1, 2. `take(3)` takes 3, 4, 5.

### A4
```
66
```
zip pairs (1,10), (2,20), (3,30). Sums of pairs: 11+22+33 = 66.

---

## Section B

### A5
```rust
fn main() {
    let data = vec!["10", "abc", "20", "xyz", "30"];
    let sum: i32 = data.iter()
        .filter_map(|s| s.parse::<i32>().ok())
        .sum();
    println!("Sum: {sum}");  // 60
}
```

### A6
```rust
fn main() {
    let students = vec![
        ("Alice", 92), ("Bob", 65), ("Charlie", 78),
        ("Diana", 55), ("Eve", 88), ("Frank", 71),
    ];

    let mut passing: Vec<String> = students.iter()
        .filter(|(_, grade)| *grade >= 70)
        .map(|(name, _)| name.to_uppercase())
        .collect();

    passing.sort();

    let numbered: Vec<String> = passing.iter()
        .enumerate()
        .map(|(i, name)| format!("{}. {name}", i + 1))
        .collect();

    numbered.iter().for_each(|e| println!("{e}"));
    // 1. ALICE
    // 2. CHARLIE
    // 3. EVE
    // 4. FRANK
}
```

### A7
```rust
use std::collections::BTreeSet;

fn main() {
    let groups = vec![vec![3, 1, 4], vec![1, 5, 9], vec![2, 6, 5]];
    let unique: BTreeSet<i32> = groups.into_iter().flatten().collect();
    println!("{:?}", unique);  // {1, 2, 3, 4, 5, 6, 9}
}
```

### A8
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let pairs: Vec<_> = v.iter().zip(v.iter().skip(1)).collect();
    println!("{:?}", pairs);  // [(1,2), (2,3), (3,4), (4,5)]
}
```

---

## Section C

### A9
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `take_while` stops iterating at the first false |
| 2 | **True** | `zip` stops when either iterator returns `None` |
| 3 | **True** | `flat_map(f)` is semantically identical to `.map(f).flatten()` |
| 4 | **False** | `inspect` is read-only; it lets you observe but not modify |
| 5 | **True** | Both iterators must yield the same `Item` type |

### A10
Both are functionally equivalent — they filter and transform in one pass. `filter_map` is slightly more concise and avoids the intermediate type mismatch that can occur when the filter condition and map transformation are tightly coupled. Use `filter_map` when the transform itself determines whether to keep the element (e.g., `parse().ok()`). Use separate `filter` + `map` when the condition and transformation are independent.

---

## 🏆 Lesson 34 Complete!

✅ map, filter, filter_map  
✅ flat_map and flatten  
✅ enumerate and zip  
✅ take, skip, take_while, skip_while  
✅ chain, peekable, inspect  
✅ Loop-free data pipelines  

**Next up:** [Lesson 35 — Collecting & FromIterator](../lesson_35_collecting/lesson_35_collecting.md) 🦀
