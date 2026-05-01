# 📘 Lesson 33 — Iterators & Iterator Trait (C5)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** C5 · Category: 📚 Collections  
> **Previous:** [Lesson 32 — HashSet, BTreeMap, BTreeSet](../lesson_32_hashset_btree/lesson_32_hashset_btree.md)  
> **Next:** [Lesson 34 — Iterator Adaptors](../lesson_34_iterator_adaptors/lesson_34_iterator_adaptors.md)  
> **Practice:** [Questions](./lesson_33_questions.md) · [Answers](./lesson_33_answers.md)  
> **Practice Task:** Chain 5 iterator adaptors to process a word list

---

## Table of Contents

1. [What Is an Iterator?](#1-what-is-an-iterator)
2. [The Iterator Trait](#2-the-iterator-trait)
3. [Three Ways to Iterate: iter, into_iter, iter_mut](#3-three-ways-to-iterate-iter-into_iter-iter_mut)
4. [for Loops Are Iterator Sugar](#4-for-loops-are-iterator-sugar)
5. [Lazy Evaluation](#5-lazy-evaluation)
6. [Consuming Adaptors](#6-consuming-adaptors)
7. [Creating Your Own Iterator](#7-creating-your-own-iterator)
8. [Iterating Over Ranges](#8-iterating-over-ranges)
9. [Common Iterator Patterns](#9-common-iterator-patterns)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Is an Iterator?

An iterator is a value that produces a sequence of items one at a time. In Rust, iterators are **lazy** — they do nothing until you consume them.

```rust
fn main() {
    let numbers = vec![10, 20, 30, 40, 50];

    // Create an iterator (lazy — does nothing yet)
    let iter = numbers.iter();

    // Consume it with a for loop
    for num in iter {
        println!("{num}");
    }
}
```

### Manual iteration with `next()`:

```rust
fn main() {
    let v = vec![1, 2, 3];
    let mut iter = v.iter();

    println!("{:?}", iter.next());  // Some(1)
    println!("{:?}", iter.next());  // Some(2)
    println!("{:?}", iter.next());  // Some(3)
    println!("{:?}", iter.next());  // None  (exhausted)
}
```

Each call to `next()` returns `Some(item)` or `None` when done.

---

## 2. The Iterator Trait

Every iterator implements this trait:

```rust
trait Iterator {
    type Item;                      // the type of elements produced
    fn next(&mut self) -> Option<Self::Item>;  // produce next element
}
```

That's it! Just **one required method**: `next()`. Everything else (map, filter, fold, etc.) is built on top of it.

```rust
// Example: what Vec::iter() returns
// impl<'a, T> Iterator for std::slice::Iter<'a, T> {
//     type Item = &'a T;
//     fn next(&mut self) -> Option<&'a T> { ... }
// }
```

### Key terminology:

| Term | Meaning |
|---|---|
| **Iterator** | A value that implements the `Iterator` trait |
| **Item** | The type of each element produced |
| **Iterator adaptor** | A method that transforms one iterator into another (lazy) |
| **Consuming adaptor** | A method that consumes the iterator, producing a final value |

---

## 3. Three Ways to Iterate: iter, into_iter, iter_mut

This is one of Rust's most important patterns:

```rust
fn main() {
    let mut names = vec![
        String::from("Alice"),
        String::from("Bob"),
        String::from("Charlie"),
    ];

    // 1. iter() — borrows each element (&T)
    println!("=== iter() — immutable references ===");
    for name in names.iter() {
        println!("  {name}");  // name is &String
    }
    // names is still usable — we only borrowed

    // 2. iter_mut() — mutably borrows each element (&mut T)
    println!("\n=== iter_mut() — mutable references ===");
    for name in names.iter_mut() {
        name.push_str("!");  // name is &mut String — we can modify
    }
    println!("  Modified: {:?}", names);

    // 3. into_iter() — takes ownership of each element (T)
    println!("\n=== into_iter() — owned values ===");
    for name in names.into_iter() {
        println!("  Owned: {name}");  // name is String
    }
    // names is MOVED — can't use it anymore
    // println!("{:?}", names);  // ❌ ERROR: value moved
}
```

### Summary table:

| Method | Yields | Ownership | Use When |
|---|---|---|---|
| `.iter()` | `&T` | Borrows | Read-only access |
| `.iter_mut()` | `&mut T` | Mutably borrows | Modify in place |
| `.into_iter()` | `T` | Takes ownership | Consume the collection |

### What `for` uses:

```rust
// for x in &collection      → calls collection.iter()
// for x in &mut collection  → calls collection.iter_mut()
// for x in collection       → calls collection.into_iter()

fn main() {
    let v = vec![1, 2, 3];

    for x in &v { println!("{x}"); }     // iter()
    // for x in &mut v { *x += 1; }      // iter_mut()
    for x in v { println!("{x}"); }       // into_iter() — v is consumed
}
```

---

## 4. for Loops Are Iterator Sugar

A `for` loop is syntactic sugar for calling `into_iter()` and `next()`:

```rust
fn main() {
    let v = vec![1, 2, 3];

    // This for loop:
    for x in &v {
        println!("{x}");
    }

    // Is equivalent to:
    let mut iter = (&v).into_iter();  // same as v.iter()
    loop {
        match iter.next() {
            Some(x) => println!("{x}"),
            None => break,
        }
    }
}
```

---

## 5. Lazy Evaluation

Iterator adaptors are **lazy** — they don't do anything until consumed:

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    // This does NOTHING — it just creates an iterator chain
    let _lazy = v.iter().map(|x| {
        println!("Processing {x}");  // never printed!
        x * 2
    });
    println!("Map created but nothing happened yet.");

    // Now consume it — THIS triggers the work
    println!("\nConsuming with collect:");
    let doubled: Vec<i32> = v.iter().map(|x| {
        println!("  Processing {x}");
        x * 2
    }).collect();
    println!("Result: {:?}", doubled);
}
```

**Output:**
```
Map created but nothing happened yet.

Consuming with collect:
  Processing 1
  Processing 2
  Processing 3
  Processing 4
  Processing 5
Result: [2, 4, 6, 8, 10]
```

### Why lazy is powerful:

```rust
fn main() {
    // Only processes first 3 elements — doesn't touch the rest
    let v: Vec<i32> = (1..1_000_000)
        .filter(|x| x % 2 == 0)
        .take(3)
        .collect();

    println!("{:?}", v);  // [2, 4, 6]
    // Did NOT iterate through 1 million numbers!
}
```

---

## 6. Consuming Adaptors

These methods consume the iterator and produce a final value:

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    // sum — add all elements
    let total: i32 = v.iter().sum();
    println!("Sum: {total}");  // 15

    // count — number of elements
    let count = v.iter().count();
    println!("Count: {count}");  // 5

    // min, max
    println!("Min: {:?}", v.iter().min());  // Some(1)
    println!("Max: {:?}", v.iter().max());  // Some(5)

    // any — does ANY element match?
    let has_even = v.iter().any(|x| x % 2 == 0);
    println!("Has even: {has_even}");  // true

    // all — do ALL elements match?
    let all_positive = v.iter().all(|x| *x > 0);
    println!("All positive: {all_positive}");  // true

    // find — first element matching predicate
    let first_even = v.iter().find(|&&x| x % 2 == 0);
    println!("First even: {:?}", first_even);  // Some(2)

    // position — index of first match
    let pos = v.iter().position(|&x| x == 3);
    println!("Position of 3: {:?}", pos);  // Some(2)

    // collect — gather into a collection
    let doubled: Vec<i32> = v.iter().map(|x| x * 2).collect();
    println!("Doubled: {:?}", doubled);  // [2, 4, 6, 8, 10]

    // for_each — like for loop but as a method
    v.iter().for_each(|x| print!("{x} "));
    println!();
}
```

### `fold` — the most powerful consumer:

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    // fold(initial_value, |accumulator, item| new_accumulator)
    let sum = v.iter().fold(0, |acc, x| acc + x);
    println!("Sum via fold: {sum}");  // 15

    let product = v.iter().fold(1, |acc, x| acc * x);
    println!("Product: {product}");  // 120

    // Build a string
    let csv = v.iter().fold(String::new(), |mut acc, x| {
        if !acc.is_empty() { acc.push(','); }
        acc.push_str(&x.to_string());
        acc
    });
    println!("CSV: {csv}");  // "1,2,3,4,5"
}
```

---

## 7. Creating Your Own Iterator

Implement the `Iterator` trait for your own types:

```rust
struct Counter {
    current: u32,
    max: u32,
}

impl Counter {
    fn new(max: u32) -> Counter {
        Counter { current: 0, max }
    }
}

impl Iterator for Counter {
    type Item = u32;

    fn next(&mut self) -> Option<u32> {
        if self.current < self.max {
            self.current += 1;
            Some(self.current)
        } else {
            None
        }
    }
}

fn main() {
    // Use in a for loop
    for i in Counter::new(5) {
        print!("{i} ");
    }
    println!();  // 1 2 3 4 5

    // Use iterator methods — they all work!
    let sum: u32 = Counter::new(5).sum();
    println!("Sum: {sum}");  // 15

    let doubled: Vec<u32> = Counter::new(3).map(|x| x * 2).collect();
    println!("Doubled: {:?}", doubled);  // [2, 4, 6]

    // Zip two counters
    let pairs: Vec<(u32, u32)> = Counter::new(3)
        .zip(Counter::new(3).map(|x| x * 10))
        .collect();
    println!("Pairs: {:?}", pairs);  // [(1, 10), (2, 20), (3, 30)]
}
```

### Fibonacci iterator:

```rust
struct Fibonacci {
    a: u64,
    b: u64,
}

impl Fibonacci {
    fn new() -> Fibonacci {
        Fibonacci { a: 0, b: 1 }
    }
}

impl Iterator for Fibonacci {
    type Item = u64;

    fn next(&mut self) -> Option<u64> {
        let result = self.a;
        let next = self.a + self.b;
        self.a = self.b;
        self.b = next;
        Some(result)  // infinite iterator — never returns None
    }
}

fn main() {
    // First 10 Fibonacci numbers
    let fibs: Vec<u64> = Fibonacci::new().take(10).collect();
    println!("{:?}", fibs);
    // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]

    // Sum of first 20 Fibonacci numbers
    let sum: u64 = Fibonacci::new().take(20).sum();
    println!("Sum of first 20: {sum}");

    // First Fibonacci number over 1000
    let big = Fibonacci::new().find(|&x| x > 1000);
    println!("First > 1000: {:?}", big);  // Some(1597)
}
```

---

## 8. Iterating Over Ranges

Ranges implement `Iterator`:

```rust
fn main() {
    // Exclusive range
    for i in 0..5 {
        print!("{i} ");
    }
    println!();  // 0 1 2 3 4

    // Inclusive range
    for i in 1..=5 {
        print!("{i} ");
    }
    println!();  // 1 2 3 4 5

    // Character ranges
    for c in 'a'..='f' {
        print!("{c} ");
    }
    println!();  // a b c d e f

    // Using range as an iterator
    let squares: Vec<i32> = (1..=5).map(|x| x * x).collect();
    println!("{:?}", squares);  // [1, 4, 9, 16, 25]

    // rev() — reverse
    for i in (1..=5).rev() {
        print!("{i} ");
    }
    println!();  // 5 4 3 2 1

    // step_by()
    for i in (0..20).step_by(3) {
        print!("{i} ");
    }
    println!();  // 0 3 6 9 12 15 18
}
```

---

## 9. Common Iterator Patterns

### Pattern 1: Transform and collect

```rust
fn main() {
    let names = vec!["alice", "bob", "charlie"];
    let capitalized: Vec<String> = names
        .iter()
        .map(|name| {
            let mut chars = name.chars();
            match chars.next() {
                Some(c) => c.to_uppercase().to_string() + chars.as_str(),
                None => String::new(),
            }
        })
        .collect();
    println!("{:?}", capitalized);  // ["Alice", "Bob", "Charlie"]
}
```

### Pattern 2: Filter and count

```rust
fn main() {
    let numbers = vec![1, -2, 3, -4, 5, -6, 7];

    let positive_count = numbers.iter().filter(|&&x| x > 0).count();
    let negative_sum: i32 = numbers.iter().filter(|&&x| x < 0).sum();

    println!("Positive count: {positive_count}");  // 4
    println!("Negative sum: {negative_sum}");       // -12
}
```

### Pattern 3: Enumerate for index + value

```rust
fn main() {
    let fruits = vec!["apple", "banana", "cherry"];

    for (i, fruit) in fruits.iter().enumerate() {
        println!("{i}: {fruit}");
    }
    // 0: apple
    // 1: banana
    // 2: cherry

    // Find index of longest word
    let longest_idx = fruits.iter()
        .enumerate()
        .max_by_key(|(_, fruit)| fruit.len())
        .map(|(i, _)| i);
    println!("Longest at index: {:?}", longest_idx);  // Some(2)
}
```

### Pattern 4: Chain multiple iterators

```rust
fn main() {
    let first = vec![1, 2, 3];
    let second = vec![4, 5, 6];

    let combined: Vec<i32> = first.iter()
        .chain(second.iter())
        .copied()
        .collect();
    println!("{:?}", combined);  // [1, 2, 3, 4, 5, 6]
}
```

---

## 10. Summary Cheat Sheet

```
THE ITERATOR TRAIT
────────────────────────────────────────────────────────────
trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

THREE WAYS TO ITERATE
────────────────────────────────────────────────────────────
.iter()        → &T      borrow elements
.iter_mut()    → &mut T  mutably borrow elements
.into_iter()   → T       take ownership of elements

for x in &v      → iter()
for x in &mut v  → iter_mut()
for x in v       → into_iter()

CONSUMING ADAPTORS (use up the iterator)
────────────────────────────────────────────────────────────
.sum()           add all elements
.count()         count elements
.min() / .max()  smallest / largest
.any(|x| pred)   any match?
.all(|x| pred)   all match?
.find(|x| pred)  first match → Option<T>
.position(pred)  index of first match → Option<usize>
.collect()       gather into a collection
.fold(init, f)   accumulate into a single value
.for_each(f)     run function on each element

LAZY EVALUATION
────────────────────────────────────────────────────────────
Adaptors do nothing until consumed
.map(), .filter(), .take() etc. build a chain
Only the final consumer triggers computation

CUSTOM ITERATORS
────────────────────────────────────────────────────────────
struct MyIter { ... }
impl Iterator for MyIter {
    type Item = ...;
    fn next(&mut self) -> Option<Self::Item> { ... }
}
```

---

## What's Next?

**Lesson 34 — Iterator Adaptors** — Deep dive into `map`, `filter`, `flat_map`, `zip`, `enumerate`, `take`, `skip`, `chain`, and more. Learn to write elegant, loop-free data pipelines.

## Further Reading
- [The Rust Book — Ch 13.2: Iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html)
- [std::iter module](https://doc.rust-lang.org/std/iter/index.html)
- [Iterator trait documentation](https://doc.rust-lang.org/std/iter/trait.Iterator.html)

---

*Iterators: where elegance meets zero-cost abstraction! 🦀*
