# ✅ Lesson 32 — Answers: HashSet, BTreeMap, BTreeSet (C4)

---

## Section A

### A1
```
true
true
false
len = 2
```
`insert` returns `true` when the value is new, `false` when it's a duplicate. The duplicate `1` is not added, so len is 2.

### A2
```
1 2 3 4 5
```
`BTreeSet` always iterates in sorted order, regardless of insertion order.

### A3
```
{2, 3}
```
(or `{3, 2}` — HashSet order is unspecified). Intersection returns elements present in both sets.

### A4
```
apple: 5
banana: 3
cherry: 1
```
`BTreeMap` iterates in sorted key order (alphabetical for strings).

---

## Section B

### A5
**`HashSet<String>`** — You only need to store unique values and check membership. `HashSet::contains()` is O(1) average, perfect for "is username taken?" checks.

### A6
**`BTreeMap<Reverse<u32>, String>`** or **`BTreeSet<Score>`** — `BTreeMap`/`BTreeSet` keeps elements sorted. Use `std::cmp::Reverse` to sort descending. Alternatively, a `Vec` sorted on each update works for small leaderboards.

### A7
**`BTreeMap<String, usize>`** — You need key-value pairs (word → count) and alphabetical output. `BTreeMap` provides both sorted iteration and key-value storage.

### A8
**`HashMap<u32, String>`** — Fast O(1) lookup by ID. No need for sorted order, so `HashMap` is the best choice.

---

## Section C

### A9 — Duplicate finder
```rust
use std::collections::HashSet;

fn find_duplicates(items: &[i32]) -> Vec<i32> {
    let mut seen = HashSet::new();
    let mut duplicates = HashSet::new();

    for &item in items {
        if !seen.insert(item) {
            duplicates.insert(item);
        }
    }

    duplicates.into_iter().collect()
}

fn main() {
    let data = [1, 5, 3, 2, 5, 1, 8, 3, 9, 1];
    let dupes = find_duplicates(&data);
    println!("Duplicates: {:?}", dupes);  // [1, 3, 5] (order varies)
}
```

### A10 — Set operations
```rust
use std::collections::HashSet;

fn main() {
    let class_a: HashSet<&str> = ["Alice", "Bob", "Charlie", "Diana"]
        .into_iter().collect();
    let class_b: HashSet<&str> = ["Charlie", "Diana", "Eve", "Frank"]
        .into_iter().collect();

    // 1. Students in both classes
    let both: HashSet<_> = class_a.intersection(&class_b).copied().collect();
    println!("In both: {:?}", both);  // {"Charlie", "Diana"}

    // 2. Students only in Class A
    let only_a: HashSet<_> = class_a.difference(&class_b).copied().collect();
    println!("Only in A: {:?}", only_a);  // {"Alice", "Bob"}

    // 3. All unique students
    let all: HashSet<_> = class_a.union(&class_b).copied().collect();
    println!("All students: {:?}", all);
    // {"Alice", "Bob", "Charlie", "Diana", "Eve", "Frank"}
}
```

### A11 — Word frequency (sorted)
```rust
use std::collections::BTreeMap;

fn word_frequency(text: &str) {
    let mut counts = BTreeMap::new();

    for word in text.split_whitespace() {
        let word = word.to_lowercase();
        *counts.entry(word).or_insert(0) += 1;
    }

    for (word, count) in &counts {
        println!("{word}: {count}");
    }
}

fn main() {
    let text = "the quick brown fox jumps over the lazy dog the fox";
    word_frequency(text);
    // brown: 1
    // dog: 1
    // fox: 2
    // jumps: 1
    // lazy: 1
    // over: 1
    // quick: 1
    // the: 3
}
```

### A12 — Range query
```rust
use std::collections::BTreeMap;

fn main() {
    let mut products = BTreeMap::new();
    products.insert(50, "Widget".to_string());
    products.insert(100, "Gadget".to_string());
    products.insert(150, "Doohickey".to_string());
    products.insert(200, "Thingamajig".to_string());
    products.insert(250, "Whatchamacallit".to_string());
    products.insert(175, "Gizmo".to_string());

    println!("Products with ID 100-200:");
    for (id, name) in products.range(100..=200) {
        println!("  #{id}: {name}");
    }
    // #100: Gadget
    // #150: Doohickey
    // #175: Gizmo
    // #200: Thingamajig
}
```

---

## Section D

### A13 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `HashSet` order is unspecified and may change between runs |
| 2 | **True** | `BTreeMap` uses a B-tree which requires `Ord` for key comparison |
| 3 | **False** | `insert` returns `true` if the value was **new** (not already present) |
| 4 | **True** | `BTreeSet` supports `.range()` for ordered queries; `HashSet` doesn't |
| 5 | **False** | `f64` doesn't implement `Eq` or `Hash` because `NaN != NaN` |
| 6 | **True** | Both support `union`, `intersection`, `difference`, and `symmetric_difference` |

### A14 — When to prefer BTreeMap
1. **Sorted output** — When you need to iterate over keys in order (e.g., alphabetical listing, sorted leaderboard)
2. **Range queries** — When you need to find all entries within a key range (e.g., dates between Jan and March, IDs 100–200)
3. **Min/Max access** — When you frequently need the smallest or largest key (`.iter().next()` or `.iter().next_back()` in O(log n))

---

## 🏆 Lesson 32 Complete!

✅ HashSet creation, insertion, membership  
✅ Set operations: union, intersection, difference  
✅ BTreeMap — sorted key-value store  
✅ BTreeSet — sorted unique values  
✅ Range queries on B-tree collections  
✅ Choosing the right collection  
✅ Custom types in sets and maps  

**Next up:** [Lesson 33 — Iterators & Iterator Trait](../lesson_33_iterators/lesson_33_iterators.md) 🦀
