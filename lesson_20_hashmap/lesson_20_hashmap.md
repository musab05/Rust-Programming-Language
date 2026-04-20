# 📘 Lesson 20 — HashMap\<K,V\> (C3)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** C3 · Category: 📚 Collections  
> **Previous:** [Lesson 19 — Vec\<T\>](./lesson_19_vec.md)  
> **Next:** Lesson 21 — panic! & unwrap *(coming soon)*  
> **Practice:** [Questions](./lesson_20_questions.md) · [Answers](./lesson_20_answers.md)  
> **Practice Task:** Word frequency counter from a string

---

## Table of Contents

1. [What Is a HashMap?](#1-what-is-a-hashmap)
2. [Creating a HashMap](#2-creating-a-hashmap)
3. [Inserting and Accessing](#3-inserting-and-accessing)
4. [The `entry` API](#4-the-entry-api)
5. [Iterating Over a HashMap](#5-iterating-over-a-hashmap)
6. [Updating Values](#6-updating-values)
7. [Removing Entries](#7-removing-entries)
8. [HashMap and Ownership](#8-hashmap-and-ownership)
9. [Useful Methods](#9-useful-methods)
10. [Custom Hash Keys](#10-custom-hash-keys)
11. [Common Patterns & Idioms](#11-common-patterns--idioms)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. What Is a HashMap?

`HashMap<K, V>` stores key-value pairs where each key is unique. Looking up a value by key is O(1) average time.

```
HashMap<String, u32>:
  "Alice" → 30
  "Bob"   → 25
  "Cara"  → 28
```

Use it when you need to:
- Look up data by a key (not by position)
- Count occurrences
- Group data
- Cache computed results

---

## 2. Creating a HashMap

```rust
use std::collections::HashMap;

fn main() {
    // Empty — must specify types or let inference work
    let mut scores: HashMap<String, u32> = HashMap::new();

    // From an iterator of pairs
    let pairs = [("Alice", 100u32), ("Bob", 85)];
    let map: HashMap<&str, u32> = pairs.into_iter().collect();
    println!("{map:?}");

    // From two Vecs zipped together
    let names  = vec!["Alice", "Bob", "Cara"];
    let scores2 = vec![100u32, 85, 92];
    let map2: HashMap<&str, u32> = names.into_iter().zip(scores2).collect();
    println!("{map2:?}");
}
```

---

## 3. Inserting and Accessing

```rust
use std::collections::HashMap;

fn main() {
    let mut map: HashMap<String, i32> = HashMap::new();

    // Insert
    map.insert(String::from("Alice"), 10);
    map.insert(String::from("Bob"),   20);

    // Access — returns Option<&V>
    println!("{:?}", map.get("Alice")); // Some(10)
    println!("{:?}", map.get("Dave"));  // None

    // Direct indexing with & — PANICS if key missing
    println!("{}", map["Alice"]);  // 10
    // println!("{}", map["Dave"]); // PANIC!

    // Check existence
    println!("{}", map.contains_key("Bob")); // true
    println!("{}", map.contains_key("Eve")); // false

    // len
    println!("{}", map.len()); // 2
}
```

---

## 4. The `entry` API

The `entry` API is the idiomatic way to insert-or-update:

```rust
use std::collections::HashMap;

fn main() {
    let mut scores: HashMap<String, u32> = HashMap::new();

    // or_insert: insert default if key absent, return &mut V
    scores.entry(String::from("Alice")).or_insert(0);
    scores.entry(String::from("Alice")).or_insert(999); // Alice already exists → no change
    println!("{:?}", scores.get("Alice")); // Some(0)

    // or_insert_with: lazily compute default
    scores.entry(String::from("Bob")).or_insert_with(|| 42);
    println!("{}", scores["Bob"]); // 42

    // Modify through the returned mutable reference
    let count = scores.entry(String::from("Alice")).or_insert(0);
    *count += 10;
    println!("{}", scores["Alice"]); // 10

    // or_default: use the Default value for the type
    let mut word_count: HashMap<&str, usize> = HashMap::new();
    *word_count.entry("hello").or_default() += 1;
    *word_count.entry("hello").or_default() += 1;
    *word_count.entry("world").or_default() += 1;
    println!("{word_count:?}"); // {"hello": 2, "world": 1}
}
```

### Word frequency — the canonical `entry` example

```rust
use std::collections::HashMap;

fn word_frequency(text: &str) -> HashMap<&str, usize> {
    let mut freq = HashMap::new();
    for word in text.split_whitespace() {
        *freq.entry(word).or_insert(0) += 1;
    }
    freq
}

fn main() {
    let text = "the quick brown fox the fox the";
    let freq = word_frequency(text);
    let mut pairs: Vec<(&&str, &usize)> = freq.iter().collect();
    pairs.sort_by_key(|(_, c)| std::cmp::Reverse(**c));
    for (word, count) in pairs {
        println!("{word}: {count}");
    }
}
```

---

## 5. Iterating Over a HashMap

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert("a", 1i32);
    map.insert("b", 2);
    map.insert("c", 3);

    // Iterate key-value pairs (order NOT guaranteed)
    for (k, v) in &map {
        println!("{k} → {v}");
    }

    // Keys only
    let mut keys: Vec<&&str> = map.keys().collect();
    keys.sort();
    println!("keys: {keys:?}");

    // Values only
    let sum: i32 = map.values().sum();
    println!("sum: {sum}");

    // Mutable values
    for v in map.values_mut() { *v *= 10; }
    println!("{map:?}");
}
```

**Important:** HashMap does not guarantee insertion order. If you need sorted output, collect keys/values and sort them.

---

## 6. Updating Values

```rust
use std::collections::HashMap;

fn main() {
    let mut map: HashMap<&str, i32> = HashMap::new();
    map.insert("score", 0);

    // Overwrite — insert replaces existing value
    map.insert("score", 100);
    println!("{}", map["score"]); // 100

    // Only insert if absent
    map.entry("score").or_insert(999);  // no change
    map.entry("bonus").or_insert(50);   // inserts 50
    println!("{} {}", map["score"], map["bonus"]); // 100 50

    // Modify in place
    *map.entry("score").or_insert(0) += 50;
    println!("{}", map["score"]); // 150

    // Get mutable reference and modify
    if let Some(v) = map.get_mut("score") {
        *v *= 2;
    }
    println!("{}", map["score"]); // 300
}
```

---

## 7. Removing Entries

```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::from([
        ("a", 1), ("b", 2), ("c", 3), ("d", 4)
    ]);

    // Remove by key — returns Option<V>
    let removed = map.remove("b");
    println!("{:?}", removed);   // Some(2)
    println!("{:?}", map.remove("z")); // None

    // Remove by key and value predicate (retain only matching)
    map.retain(|_k, v| *v > 1);
    println!("{map:?}"); // {"c": 3, "d": 4}

    // Clear all
    map.clear();
    println!("{}", map.is_empty()); // true
}
```

---

## 8. HashMap and Ownership

`HashMap` takes **ownership** of keys and values when inserted:

```rust
use std::collections::HashMap;

fn main() {
    let key   = String::from("Alice");
    let value = String::from("Engineer");

    let mut map = HashMap::new();
    map.insert(key, value);

    // println!("{key}");   // ❌ key was moved into the HashMap
    // println!("{value}"); // ❌ value was moved

    // Access values via references — no move
    if let Some(job) = map.get("Alice") {
        println!("{job}"); // ✅ borrowed, not moved
    }
}
```

For `Copy` types (integers, bools, chars), values are copied:

```rust
fn main() {
    let key = String::from("score");
    let val = 42i32;          // Copy
    let mut map = HashMap::new();
    map.insert(key, val);
    println!("{val}");         // ✅ val was copied, not moved
}
```

---

## 9. Useful Methods

```rust
use std::collections::HashMap;

fn main() {
    let mut map: HashMap<&str, i32> = HashMap::from([
        ("a", 1), ("b", 2), ("c", 3)
    ]);

    // len / is_empty / capacity
    println!("len={} empty={}", map.len(), map.is_empty());

    // contains_key / get / get_mut
    println!("{}", map.contains_key("a")); // true
    println!("{:?}", map.get("z"));        // None
    *map.get_mut("a").unwrap() += 100;
    println!("{}", map["a"]);              // 101

    // keys / values / iter (all return iterators)
    let total: i32 = map.values().sum();
    println!("total = {total}");

    // entry variants
    map.entry("d").or_insert(4);
    map.entry("d").or_insert_with(|| 99);  // not called — d exists
    map.entry("d").and_modify(|v| *v *= 10); // modify if exists
    println!("{}", map["d"]); // 40

    // HashMap::from() shorthand
    let m2 = HashMap::from([("x", 10), ("y", 20)]);
    println!("{:?}", m2);
}
```

---

## 10. Custom Hash Keys

Any type that implements `Eq + Hash` can be a key. Most primitives do. For your own types, derive both:

```rust
use std::collections::HashMap;

#[derive(Debug, PartialEq, Eq, Hash)]
struct Point { x: i32, y: i32 }

fn main() {
    let mut grid: HashMap<Point, char> = HashMap::new();
    grid.insert(Point { x: 0, y: 0 }, 'S');
    grid.insert(Point { x: 5, y: 3 }, 'E');
    grid.insert(Point { x: 2, y: 1 }, '#');

    println!("{:?}", grid.get(&Point { x: 0, y: 0 })); // Some('S')
    println!("{:?}", grid.get(&Point { x: 1, y: 1 })); // None
}
```

---

## 11. Common Patterns & Idioms

### Count occurrences

```rust
let words = vec!["apple", "banana", "apple", "cherry", "banana", "apple"];
let mut freq: HashMap<&str, usize> = HashMap::new();
for word in &words {
    *freq.entry(word).or_insert(0) += 1;
}
```

### Group by a property

```rust
let names = vec!["Alice", "Bob", "Anna", "Betty", "Carl"];
let mut by_letter: HashMap<char, Vec<&str>> = HashMap::new();
for name in &names {
    by_letter.entry(name.chars().next().unwrap())
             .or_insert_with(Vec::new)
             .push(name);
}
```

### Invert a HashMap

```rust
let original: HashMap<&str, u32> = HashMap::from([("a", 1), ("b", 2), ("c", 3)]);
let inverted: HashMap<u32, &str> = original.iter().map(|(&k, &v)| (v, k)).collect();
```

### Memoisation / caching

```rust
fn fib_memo(n: u64, cache: &mut HashMap<u64, u64>) -> u64 {
    if n <= 1 { return n; }
    if let Some(&v) = cache.get(&n) { return v; }
    let result = fib_memo(n-1, cache) + fib_memo(n-2, cache);
    cache.insert(n, result);
    result
}

fn main() {
    let mut cache = HashMap::new();
    println!("{}", fib_memo(50, &mut cache)); // fast!
}
```

---

## 12. Summary Cheat Sheet

```
CREATING
────────────────────────────────────────────────────────────
HashMap::new()                  empty
HashMap::from([("k",v), ...])   from literal pairs
iter.collect()                  from iterator of (K,V) pairs

INSERTING / ACCESSING
────────────────────────────────────────────────────────────
map.insert(k, v)         insert or overwrite → Option<V> (old value)
map.get(&k)              → Option<&V>
map[&k]                  → &V, PANICS if absent
map.get_mut(&k)          → Option<&mut V>
map.contains_key(&k)     → bool

ENTRY API (preferred for insert-or-update)
────────────────────────────────────────────────────────────
map.entry(k).or_insert(v)        insert v if absent, return &mut V
map.entry(k).or_insert_with(f)   insert f() if absent (lazy)
map.entry(k).or_default()        insert Default::default() if absent
map.entry(k).and_modify(|v| ...) modify existing value if present

ITERATING
────────────────────────────────────────────────────────────
for (k, v) in &map          borrows → (&K, &V)
for (k, v) in &mut map      → (&K, &mut V)
for (k, v) in map           consumes → (K, V)
map.keys()   map.values()   map.values_mut()
ORDER IS NOT GUARANTEED — sort if you need it

REMOVING
────────────────────────────────────────────────────────────
map.remove(&k)             → Option<V>
map.retain(|k,v| ...)      keep matching entries in place
map.clear()                remove all

OWNERSHIP
────────────────────────────────────────────────────────────
insert(k, v) MOVES k and v into the map
get(&k)      BORROWS — returns reference
Copy types   are copied, not moved

KEY REQUIREMENTS
────────────────────────────────────────────────────────────
#[derive(PartialEq, Eq, Hash)]  on custom key types
```

---

## What's Next?

**Lesson 21 — panic! & unwrap** — When to panic, how to use `unwrap` and `expect` safely, and when to reach for `Result` instead.

## Further Reading
- [The Rust Book — Ch 8.3: Storing Keys with Associated Values in Hash Maps](https://doc.rust-lang.org/book/ch08-03-hash-maps.html)
- [std::collections::HashMap](https://doc.rust-lang.org/std/collections/struct.HashMap.html)
