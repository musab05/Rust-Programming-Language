# 📘 Lesson 32 — HashSet, BTreeMap, BTreeSet (C4)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** C4 · Category: 📚 Collections  
> **Previous:** [Lesson 31 — Lifetimes in Structs & Advanced](../lesson_31_lifetimes_advanced/lesson_31_lifetimes_advanced.md)  
> **Next:** [Lesson 33 — Iterators & Iterator Trait](../lesson_33_iterators/lesson_33_iterators.md)  
> **Practice:** [Questions](./lesson_32_questions.md) · [Answers](./lesson_32_answers.md)  
> **Practice Task:** Find duplicates in a Vec using HashSet

---

## Table of Contents

1. [HashSet Basics](#1-hashset-basics)
2. [HashSet Operations](#2-hashset-operations)
3. [Set Operations — Union, Intersection, Difference](#3-set-operations--union-intersection-difference)
4. [BTreeMap — Sorted Map](#4-btreemap--sorted-map)
5. [BTreeSet — Sorted Set](#5-btreeset--sorted-set)
6. [Choosing the Right Collection](#6-choosing-the-right-collection)
7. [Custom Types in Sets and Maps](#7-custom-types-in-sets-and-maps)
8. [Real-World Example: Duplicate Finder](#8-real-world-example-duplicate-finder)
9. [Performance Comparison](#9-performance-comparison)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. HashSet Basics

A `HashSet<T>` stores **unique** values with O(1) average lookup, insertion, and removal.

```rust
use std::collections::HashSet;

fn main() {
    // Creating a HashSet
    let mut fruits = HashSet::new();

    // Inserting — returns true if the value was new
    println!("{}", fruits.insert("apple"));   // true  (new)
    println!("{}", fruits.insert("banana"));  // true  (new)
    println!("{}", fruits.insert("apple"));   // false (duplicate!)

    // Checking membership
    println!("Has apple?  {}", fruits.contains("apple"));   // true
    println!("Has cherry? {}", fruits.contains("cherry"));  // false

    // Size
    println!("Count: {}", fruits.len());  // 2

    // Iterating (order is NOT guaranteed)
    for fruit in &fruits {
        println!("  {fruit}");
    }

    // Removing
    fruits.remove("banana");
    println!("After remove: {:?}", fruits);  // {"apple"}
}
```

### Creating from an iterator:

```rust
use std::collections::HashSet;

fn main() {
    // From a Vec (deduplicates automatically)
    let numbers = vec![1, 2, 3, 2, 1, 4, 3, 5];
    let unique: HashSet<_> = numbers.into_iter().collect();
    println!("{:?}", unique);  // {1, 2, 3, 4, 5} (order may vary)

    // From an array
    let vowels: HashSet<char> = ['a', 'e', 'i', 'o', 'u'].into_iter().collect();
    println!("{:?}", vowels);
}
```

---

## 2. HashSet Operations

```rust
use std::collections::HashSet;

fn main() {
    let mut set: HashSet<i32> = HashSet::new();

    // insert — add a value
    set.insert(10);
    set.insert(20);
    set.insert(30);

    // contains — check membership (O(1))
    assert!(set.contains(&20));

    // remove — remove a value, returns bool
    assert!(set.remove(&20));    // true  — was present
    assert!(!set.remove(&99));   // false — wasn't present

    // len, is_empty
    println!("Size: {}, Empty: {}", set.len(), set.is_empty());

    // get — returns Option<&T> (useful for borrowed lookups)
    if let Some(val) = set.get(&10) {
        println!("Found: {val}");
    }

    // retain — keep only elements matching predicate
    set.insert(5);
    set.insert(15);
    set.insert(25);
    set.retain(|&x| x > 10);
    println!("After retain(>10): {:?}", set);  // {15, 25, 30}

    // clear
    set.clear();
    println!("After clear: {:?}", set);  // {}
}
```

---

## 3. Set Operations — Union, Intersection, Difference

These are the mathematical set operations — extremely useful!

```rust
use std::collections::HashSet;

fn main() {
    let a: HashSet<i32> = [1, 2, 3, 4, 5].into_iter().collect();
    let b: HashSet<i32> = [3, 4, 5, 6, 7].into_iter().collect();

    // Union — all elements in either set
    let union: HashSet<_> = a.union(&b).copied().collect();
    println!("Union:        {:?}", union);
    // {1, 2, 3, 4, 5, 6, 7}

    // Intersection — elements in BOTH sets
    let inter: HashSet<_> = a.intersection(&b).copied().collect();
    println!("Intersection: {:?}", inter);
    // {3, 4, 5}

    // Difference — elements in a but NOT in b
    let diff: HashSet<_> = a.difference(&b).copied().collect();
    println!("A - B:        {:?}", diff);
    // {1, 2}

    // Symmetric difference — elements in one but not both
    let sym: HashSet<_> = a.symmetric_difference(&b).copied().collect();
    println!("Sym diff:     {:?}", sym);
    // {1, 2, 6, 7}

    // Subset / superset checks
    let small: HashSet<i32> = [3, 4].into_iter().collect();
    println!("small ⊂ a? {}", small.is_subset(&a));    // true
    println!("a ⊃ small? {}", a.is_superset(&small));  // true

    // Disjoint — no common elements
    let c: HashSet<i32> = [10, 20].into_iter().collect();
    println!("a disjoint c? {}", a.is_disjoint(&c));  // true
}
```

### Using `&` operator syntax:

```rust
use std::collections::HashSet;

fn main() {
    let a: HashSet<i32> = [1, 2, 3].into_iter().collect();
    let b: HashSet<i32> = [2, 3, 4].into_iter().collect();

    // Operator overloads (return new HashSets)
    let union = &a | &b;       // union
    let inter = &a & &b;       // intersection
    let diff  = &a - &b;       // difference
    let sym   = &a ^ &b;       // symmetric difference

    println!("a | b = {:?}", union);
    println!("a & b = {:?}", inter);
    println!("a - b = {:?}", diff);
    println!("a ^ b = {:?}", sym);
}
```

---

## 4. BTreeMap — Sorted Map

`BTreeMap<K, V>` works like `HashMap` but keeps keys **sorted**. O(log n) operations.

```rust
use std::collections::BTreeMap;

fn main() {
    let mut scores = BTreeMap::new();

    scores.insert("Charlie", 85);
    scores.insert("Alice", 95);
    scores.insert("Bob", 88);
    scores.insert("Diana", 92);

    // Iteration is SORTED by key
    for (name, score) in &scores {
        println!("{name}: {score}");
    }
    // Alice: 95
    // Bob: 88
    // Charlie: 85
    // Diana: 92

    // Range queries!
    println!("\nNames B-D:");
    for (name, score) in scores.range("B".."D") {
        println!("  {name}: {score}");
    }
    // Bob: 88
    // Charlie: 85

    // First and last
    println!("\nFirst: {:?}", scores.iter().next());
    println!("Last:  {:?}", scores.iter().next_back());
}
```

### BTreeMap vs HashMap:

```rust
use std::collections::{BTreeMap, HashMap};

fn main() {
    // HashMap — fast, unordered
    let mut hmap = HashMap::new();
    hmap.insert(3, "three");
    hmap.insert(1, "one");
    hmap.insert(2, "two");
    print!("HashMap: ");
    for (k, v) in &hmap {
        print!("{k}={v} ");
    }
    println!();  // order is unpredictable

    // BTreeMap — sorted by key
    let mut bmap = BTreeMap::new();
    bmap.insert(3, "three");
    bmap.insert(1, "one");
    bmap.insert(2, "two");
    print!("BTreeMap: ");
    for (k, v) in &bmap {
        print!("{k}={v} ");
    }
    println!();  // always: 1=one 2=two 3=three
}
```

---

## 5. BTreeSet — Sorted Set

`BTreeSet<T>` is like `HashSet` but keeps elements **sorted**:

```rust
use std::collections::BTreeSet;

fn main() {
    let mut set = BTreeSet::new();
    set.insert(30);
    set.insert(10);
    set.insert(20);
    set.insert(10);  // duplicate — ignored

    // Always iterated in sorted order
    for val in &set {
        print!("{val} ");
    }
    println!();  // 10 20 30

    // Range queries
    println!("Values >= 15:");
    for val in set.range(15..) {
        print!("{val} ");
    }
    println!();  // 20 30

    // First and last
    println!("Min: {:?}", set.iter().next());       // Some(10)
    println!("Max: {:?}", set.iter().next_back());   // Some(30)

    // All set operations work (union, intersection, etc.)
    let other: BTreeSet<i32> = [20, 40, 50].into_iter().collect();
    let inter: BTreeSet<_> = set.intersection(&other).copied().collect();
    println!("Intersection: {:?}", inter);  // {20}
}
```

---

## 6. Choosing the Right Collection

| Need | Collection | Time Complexity |
|---|---|---|
| Unique values, fast lookup | `HashSet<T>` | O(1) avg |
| Unique values, sorted | `BTreeSet<T>` | O(log n) |
| Key-value, fast lookup | `HashMap<K, V>` | O(1) avg |
| Key-value, sorted by key | `BTreeMap<K, V>` | O(log n) |
| Range queries | `BTreeMap` / `BTreeSet` | O(log n + k) |
| Ordered sequence | `Vec<T>` | O(1) index |

### Decision flowchart:

```
Do you need key-value pairs?
├── Yes → Do you need sorted keys or range queries?
│         ├── Yes → BTreeMap
│         └── No  → HashMap (faster)
└── No  → Do you need unique values?
          ├── Yes → Do you need sorted order?
          │         ├── Yes → BTreeSet
          │         └── No  → HashSet (faster)
          └── No  → Vec
```

---

## 7. Custom Types in Sets and Maps

### For `HashSet` / `HashMap` — need `Hash + Eq`:

```rust
use std::collections::HashSet;

#[derive(Debug, Hash, Eq, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

fn main() {
    let mut visited = HashSet::new();
    visited.insert(Point { x: 0, y: 0 });
    visited.insert(Point { x: 1, y: 0 });
    visited.insert(Point { x: 0, y: 0 });  // duplicate

    println!("Visited {} points: {:?}", visited.len(), visited);
    // Visited 2 points
}
```

### For `BTreeSet` / `BTreeMap` — need `Ord + Eq`:

```rust
use std::collections::BTreeSet;

#[derive(Debug, Eq, PartialEq, Ord, PartialOrd)]
struct Score {
    value: u32,
    name: String,
}

fn main() {
    let mut leaderboard = BTreeSet::new();
    leaderboard.insert(Score { value: 100, name: "Alice".into() });
    leaderboard.insert(Score { value: 85,  name: "Bob".into() });
    leaderboard.insert(Score { value: 95,  name: "Charlie".into() });

    for entry in &leaderboard {
        println!("{}: {}", entry.name, entry.value);
    }
    // Sorted by value first (then name): Bob:85, Charlie:95, Alice:100
}
```

---

## 8. Real-World Example: Duplicate Finder

This is the roadmap practice task:

```rust
use std::collections::HashSet;

fn find_duplicates<T: std::hash::Hash + Eq + Clone>(items: &[T]) -> Vec<T> {
    let mut seen = HashSet::new();
    let mut duplicates = HashSet::new();

    for item in items {
        if !seen.insert(item) {
            // insert returns false if already present
            duplicates.insert(item.clone());
        }
    }

    duplicates.into_iter().collect()
}

fn find_unique<T: std::hash::Hash + Eq>(items: &[T]) -> Vec<&T> {
    let mut seen = HashSet::new();
    items.iter().filter(|item| seen.insert(*item)).collect()
}

fn main() {
    // Find duplicates in numbers
    let numbers = vec![1, 5, 3, 2, 5, 1, 8, 3, 9, 1];
    let dupes = find_duplicates(&numbers);
    println!("Duplicates: {:?}", dupes);  // [1, 3, 5] (order may vary)

    // Find duplicates in strings
    let words = vec!["hello", "world", "hello", "rust", "world"];
    let dupes = find_duplicates(&words);
    println!("Duplicate words: {:?}", dupes);  // ["hello", "world"]

    // Deduplicate preserving first occurrence order
    let data = vec![3, 1, 4, 1, 5, 9, 2, 6, 5, 3];
    let unique = find_unique(&data);
    println!("Unique (ordered): {:?}", unique);
    // [3, 1, 4, 5, 9, 2, 6]

    // Count unique elements
    let set: HashSet<_> = numbers.iter().collect();
    println!("Unique count: {} out of {}", set.len(), numbers.len());
}
```

---

## 9. Performance Comparison

```rust
use std::collections::{BTreeMap, BTreeSet, HashMap, HashSet};
use std::time::Instant;

fn main() {
    let n = 100_000;

    // HashMap vs BTreeMap insert
    let start = Instant::now();
    let mut hmap = HashMap::new();
    for i in 0..n {
        hmap.insert(i, i * 2);
    }
    let hash_time = start.elapsed();

    let start = Instant::now();
    let mut bmap = BTreeMap::new();
    for i in 0..n {
        bmap.insert(i, i * 2);
    }
    let btree_time = start.elapsed();

    println!("Insert {n} items:");
    println!("  HashMap:  {:?}", hash_time);
    println!("  BTreeMap: {:?}", btree_time);

    // Lookup
    let start = Instant::now();
    for i in 0..n {
        hmap.get(&i);
    }
    let hash_lookup = start.elapsed();

    let start = Instant::now();
    for i in 0..n {
        bmap.get(&i);
    }
    let btree_lookup = start.elapsed();

    println!("Lookup {n} items:");
    println!("  HashMap:  {:?}", hash_lookup);
    println!("  BTreeMap: {:?}", btree_lookup);

    // BTreeMap advantage: range query
    let start = Instant::now();
    let _range: Vec<_> = bmap.range(1000..2000).collect();
    let range_time = start.elapsed();
    println!("Range query (1000 items): {:?}", range_time);
}
```

### General guidelines:

| Operation | HashMap/HashSet | BTreeMap/BTreeSet |
|---|---|---|
| Insert | O(1) avg | O(log n) |
| Lookup | O(1) avg | O(log n) |
| Remove | O(1) avg | O(log n) |
| Iterate | Unordered | Sorted |
| Range query | ❌ Not supported | O(log n + k) ✅ |
| Min/Max | O(n) | O(log n) ✅ |

---

## 10. Summary Cheat Sheet

```
HASHSET<T> (requires T: Hash + Eq)
────────────────────────────────────────────────────────────
insert(val)         → bool (true if new)
contains(&val)      → bool
remove(&val)        → bool
len(), is_empty()
retain(|v| pred)    keep matching elements

SET OPERATIONS
────────────────────────────────────────────────────────────
a.union(&b)                 or   &a | &b
a.intersection(&b)          or   &a & &b
a.difference(&b)            or   &a - &b
a.symmetric_difference(&b)  or   &a ^ &b
a.is_subset(&b), a.is_superset(&b), a.is_disjoint(&b)

BTREEMAP<K, V> (requires K: Ord)
────────────────────────────────────────────────────────────
insert(k, v), get(&k), remove(&k)    — same as HashMap
range(start..end)                     — iterate a key range
iter() → sorted by key
first_key_value(), last_key_value()   — min/max

BTREESET<T> (requires T: Ord)
────────────────────────────────────────────────────────────
Same as HashSet API + sorted iteration + range()

CHOOSING
────────────────────────────────────────────────────────────
Need speed? → Hash variants
Need order? → BTree variants
Need ranges? → BTree variants
Need both key + value? → Map
Need just values? → Set
```

---

## What's Next?

**Lesson 33 — Iterators & Iterator Trait** — Master Rust's most powerful abstraction. Learn `iter()`, `into_iter()`, `iter_mut()`, and how lazy evaluation makes your code both elegant and fast.

## Further Reading
- [std::collections::HashSet](https://doc.rust-lang.org/std/collections/struct.HashSet.html)
- [std::collections::BTreeMap](https://doc.rust-lang.org/std/collections/struct.BTreeMap.html)
- [The Rust Book — Collections](https://doc.rust-lang.org/book/ch08-00-common-collections.html)

---

*Sets and sorted maps: powerful tools for every Rustacean's toolbox! 🦀*
