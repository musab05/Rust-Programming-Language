# 📘 Lesson 34 — Iterator Adaptors (C6)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** C6 · Category: 📚 Collections  
> **Previous:** [Lesson 33 — Iterators & Iterator Trait](../lesson_33_iterators/lesson_33_iterators.md)  
> **Next:** [Lesson 35 — Collecting & FromIterator](../lesson_35_collecting/lesson_35_collecting.md)  
> **Practice:** [Questions](./lesson_34_questions.md) · [Answers](./lesson_34_answers.md)  
> **Practice Task:** Re-implement a pipeline without explicit loops

---

## Table of Contents

1. [map — Transform Each Element](#1-map)
2. [filter — Keep Matching Elements](#2-filter)
3. [filter_map — Filter + Transform](#3-filter_map)
4. [flat_map & flatten](#4-flat_map--flatten)
5. [enumerate — Index + Value](#5-enumerate)
6. [zip — Pair Two Iterators](#6-zip)
7. [take, skip, take_while, skip_while](#7-take-skip-take_while-skip_while)
8. [chain — Concatenate Iterators](#8-chain)
9. [peekable, inspect, copied, cloned](#9-peekable-inspect-copied-cloned)
10. [Chaining It All Together](#10-chaining-it-all-together)
11. [Summary Cheat Sheet](#11-summary-cheat-sheet)

---

## 1. map

`map` applies a function to each element:

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let squares: Vec<i32> = v.iter().map(|x| x * x).collect();
    println!("{:?}", squares);  // [1, 4, 9, 16, 25]

    let strings: Vec<String> = v.iter().map(|x| x.to_string()).collect();
    println!("{:?}", strings);  // ["1", "2", "3", "4", "5"]
}
```

---

## 2. filter

`filter` keeps elements where the predicate returns `true`:

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let evens: Vec<&i32> = v.iter().filter(|&&x| x % 2 == 0).collect();
    println!("Evens: {:?}", evens);  // [2, 4, 6, 8, 10]

    // Combine filter + map
    let even_squares: Vec<i32> = v.iter()
        .filter(|&&x| x % 2 == 0)
        .map(|x| x * x)
        .collect();
    println!("Even squares: {:?}", even_squares);  // [4, 16, 36, 64, 100]
}
```

**Why `&&x`?** `iter()` yields `&i32`, and `filter` gives you `&&i32` (a reference to the reference).

---

## 3. filter_map

Combines `filter` + `map` — return `Some(value)` to keep, `None` to skip:

```rust
fn main() {
    let strings = vec!["1", "two", "3", "four", "5"];
    let numbers: Vec<i32> = strings.iter()
        .filter_map(|s| s.parse::<i32>().ok())
        .collect();
    println!("{:?}", numbers);  // [1, 3, 5]

    // Extract Some values from Options
    let opts = vec![Some(1), None, Some(3), None, Some(5)];
    let vals: Vec<i32> = opts.into_iter().flatten().collect();
    println!("{:?}", vals);  // [1, 3, 5]
}
```

---

## 4. flat_map & flatten

`flat_map` maps each element to an iterator, then flattens everything:

```rust
fn main() {
    let sentences = vec!["hello world", "foo bar baz"];
    let words: Vec<&str> = sentences.iter()
        .flat_map(|s| s.split_whitespace())
        .collect();
    println!("{:?}", words);  // ["hello", "world", "foo", "bar", "baz"]

    // flatten nested vecs
    let nested = vec![vec![1, 2], vec![3, 4], vec![5, 6]];
    let flat: Vec<i32> = nested.into_iter().flatten().collect();
    println!("{:?}", flat);  // [1, 2, 3, 4, 5, 6]
}
```

---

## 5. enumerate

```rust
fn main() {
    let fruits = vec!["apple", "banana", "cherry"];
    for (i, fruit) in fruits.iter().enumerate() {
        println!("{i}: {fruit}");
    }

    // Find index of longest word
    let idx = fruits.iter().enumerate()
        .max_by_key(|(_, f)| f.len())
        .map(|(i, _)| i);
    println!("Longest at: {:?}", idx);  // Some(2)
}
```

---

## 6. zip

Pairs elements from two iterators (stops at shorter):

```rust
fn main() {
    let names = vec!["Alice", "Bob", "Charlie"];
    let scores = vec![95, 87, 92];

    let results: Vec<_> = names.iter().zip(scores.iter()).collect();
    println!("{:?}", results);  // [("Alice", 95), ("Bob", 87), ("Charlie", 92)]

    // Dot product
    let dot: i32 = vec![1, 2, 3].iter()
        .zip(vec![4, 5, 6].iter())
        .map(|(a, b)| a * b)
        .sum();
    println!("Dot: {dot}");  // 32

    // unzip
    let (ns, ss): (Vec<&&str>, Vec<&&i32>) = names.iter().zip(scores.iter()).unzip();
    println!("{:?} {:?}", ns, ss);
}
```

---

## 7. take, skip, take_while, skip_while

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let first_3: Vec<_> = v.iter().take(3).collect();
    println!("Take 3: {:?}", first_3);  // [1, 2, 3]

    let after_3: Vec<_> = v.iter().skip(3).collect();
    println!("Skip 3: {:?}", after_3);  // [4, 5, 6, 7, 8, 9, 10]

    // Pagination: page 2, 3 per page
    let page: Vec<_> = v.iter().skip(3).take(3).collect();
    println!("Page 2: {:?}", page);  // [4, 5, 6]

    // take_while — take while predicate holds, stop at first false
    let small: Vec<_> = v.iter().take_while(|&&x| x < 5).collect();
    println!("While < 5: {:?}", small);  // [1, 2, 3, 4]

    // skip_while — skip prefix while predicate holds
    let big: Vec<_> = v.iter().skip_while(|&&x| x < 5).collect();
    println!("After < 5: {:?}", big);  // [5, 6, 7, 8, 9, 10]
}
```

---

## 8. chain

Concatenate iterators:

```rust
fn main() {
    let a = vec![1, 2, 3];
    let b = vec![4, 5, 6];

    let combined: Vec<i32> = a.iter().chain(b.iter()).copied().collect();
    println!("{:?}", combined);  // [1, 2, 3, 4, 5, 6]

    // Chain with ranges
    let mixed: Vec<i32> = a.iter().copied().chain(10..=12).collect();
    println!("{:?}", mixed);  // [1, 2, 3, 10, 11, 12]
}
```

---

## 9. peekable, inspect, copied, cloned

```rust
fn main() {
    // peekable — look ahead without consuming
    let mut iter = vec![1, 2, 3].into_iter().peekable();
    println!("Peek: {:?}", iter.peek());  // Some(1)
    println!("Next: {:?}", iter.next());  // Some(1) — now consumed

    // inspect — debug mid-chain (side effect only)
    let sum: i32 = (1..=5)
        .inspect(|x| print!("{x} "))
        .sum();
    println!("= {sum}");  // 1 2 3 4 5 = 15

    // copied — copy &T to T (for Copy types)
    let v = vec![1, 2, 3];
    let owned: Vec<i32> = v.iter().copied().collect();

    // cloned — clone &T to T (for Clone types)
    let strings = vec![String::from("a"), String::from("b")];
    let cloned: Vec<String> = strings.iter().cloned().collect();
    println!("{:?}", cloned);
}
```

---

## 10. Chaining It All Together

### Roadmap Practice Task — loop-free pipeline:

```rust
fn main() {
    let data = vec![
        ("Alice", 85), ("Bob", 92), ("Charlie", 78),
        ("Diana", 95), ("Eve", 88), ("Frank", 65),
    ];

    // Build honor roll — no loops!
    let honor_roll: Vec<String> = data.iter()
        .filter(|(_, score)| *score >= 85)
        .enumerate()
        .map(|(i, (name, score))| format!("{}. {} — {}pts ⭐", i + 1, name, score))
        .collect();

    println!("=== Honor Roll ===");
    honor_roll.iter().for_each(|e| println!("  {e}"));

    // Stats
    let avg: f64 = data.iter().map(|(_, s)| *s as f64).sum::<f64>() / data.len() as f64;
    println!("\nAverage: {avg:.1}");
}
```

### Log processing pipeline:

```rust
fn main() {
    let log = "INFO Server started\nDEBUG Pool init\nERROR DB failed\nINFO Retrying\nERROR Timeout";

    let errors: Vec<String> = log.lines()
        .filter(|l| l.starts_with("ERROR"))
        .enumerate()
        .map(|(i, l)| format!("#{}: {}", i + 1, &l[6..]))
        .collect();

    println!("Errors:");
    errors.iter().for_each(|e| println!("  {e}"));
}
```

---

## 11. Summary Cheat Sheet

```
TRANSFORMING           .map(|x| expr)         transform each element
                       .filter(|x| bool)      keep matching elements
                       .filter_map(|x| Opt)   filter + transform
                       .flat_map(|x| iter)    map then flatten
                       .flatten()             flatten nested iterators

INDEXING & PAIRING     .enumerate()           (index, value) pairs
                       .zip(other)            pair two iterators
                       .unzip()               split pairs

SLICING                .take(n)               first n elements
                       .skip(n)               skip first n
                       .take_while(pred)      take while true
                       .skip_while(pred)      skip while true

COMBINING              .chain(other)          concatenate iterators

UTILITIES              .peekable()            look ahead
                       .inspect(|x| ...)      debug mid-chain
                       .copied() / .cloned()  dereference values
                       .rev()                 reverse iteration
```

---

## What's Next?

**Lesson 35 — Collecting & FromIterator** — Master `.collect()` and learn to gather iterators into `Vec`, `HashMap`, `String`, `Result<Vec<_>>`, and your own types.

## Further Reading
- [Iterator trait — all methods](https://doc.rust-lang.org/std/iter/trait.Iterator.html)
- [Rust by Example — Iterators](https://doc.rust-lang.org/rust-by-example/trait/iter.html)

---

*Iterator adaptors: data pipelines without a single loop! 🦀*
