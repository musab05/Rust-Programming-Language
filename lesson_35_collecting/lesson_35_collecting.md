# 📘 Lesson 35 — Collecting & FromIterator (C7)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** C7 · Category: 📚 Collections  
> **Previous:** [Lesson 34 — Iterator Adaptors](../lesson_34_iterator_adaptors/lesson_34_iterator_adaptors.md)  
> **Next:** [Lesson 36 — Custom Error Types](../lesson_36_custom_errors/lesson_36_custom_errors.md)  
> **Practice:** [Questions](./lesson_35_questions.md) · [Answers](./lesson_35_answers.md)  
> **Practice Task:** Collect iterator of Results into Result<Vec<_>>

---

## Table of Contents

1. [collect() Basics](#1-collect-basics)
2. [Collecting into Different Types](#2-collecting-into-different-types)
3. [The Turbofish ::<>](#3-the-turbofish-)
4. [Collecting into HashMap](#4-collecting-into-hashmap)
5. [Collecting into String](#5-collecting-into-string)
6. [Collecting Results — The Magic Trick](#6-collecting-results--the-magic-trick)
7. [partition — Split into Two](#7-partition--split-into-two)
8. [unzip — Split Pairs](#8-unzip--split-pairs)
9. [The FromIterator Trait](#9-the-fromiterator-trait)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. collect() Basics

`collect()` gathers an iterator into a collection. It uses the `FromIterator` trait to know what to build:

```rust
fn main() {
    let v: Vec<i32> = (1..=5).collect();
    println!("{:?}", v);  // [1, 2, 3, 4, 5]

    // Type annotation tells collect what to build
    let v: Vec<_> = (1..=5).collect();        // Vec inferred
    let v = (1..=5).collect::<Vec<i32>>();     // turbofish syntax
    let v = (1..=5).collect::<Vec<_>>();       // turbofish with inference
}
```

---

## 2. Collecting into Different Types

```rust
use std::collections::{HashSet, BTreeSet, LinkedList, VecDeque};

fn main() {
    let data = vec![3, 1, 4, 1, 5, 9, 2, 6, 5];

    // Into Vec (preserves order, allows duplicates)
    let v: Vec<_> = data.iter().copied().collect();
    println!("Vec: {:?}", v);

    // Into HashSet (unique, unordered)
    let s: HashSet<_> = data.iter().copied().collect();
    println!("HashSet: {:?}", s);

    // Into BTreeSet (unique, sorted)
    let b: BTreeSet<_> = data.iter().copied().collect();
    println!("BTreeSet: {:?}", b);  // {1, 2, 3, 4, 5, 6, 9}

    // Into VecDeque
    let d: VecDeque<_> = data.iter().copied().collect();
    println!("VecDeque: {:?}", d);

    // Into LinkedList
    let l: LinkedList<_> = data.iter().copied().collect();
    println!("LinkedList: {:?}", l);
}
```

---

## 3. The Turbofish ::<>

When the compiler can't infer the target type, use the "turbofish" `::<>`:

```rust
fn main() {
    // These are equivalent:
    let v1: Vec<i32> = (1..=5).collect();
    let v2 = (1..=5).collect::<Vec<i32>>();
    let v3 = (1..=5).collect::<Vec<_>>();  // _ = let compiler figure out element type

    // Turbofish is essential when type isn't otherwise constrained
    println!("{}", (1..=5).collect::<Vec<_>>().len());
}
```

---

## 4. Collecting into HashMap

```rust
use std::collections::HashMap;

fn main() {
    // From iterator of (key, value) tuples
    let scores: HashMap<&str, i32> = vec![
        ("Alice", 95), ("Bob", 87), ("Charlie", 92),
    ].into_iter().collect();
    println!("{:?}", scores);

    // From two iterators via zip
    let names = vec!["Alice", "Bob", "Charlie"];
    let grades = vec![95, 87, 92];
    let map: HashMap<_, _> = names.iter().zip(grades.iter()).collect();
    println!("{:?}", map);

    // Word frequency counter
    let text = "the cat sat on the mat the cat";
    let freq: HashMap<&str, usize> = text.split_whitespace()
        .fold(HashMap::new(), |mut acc, word| {
            *acc.entry(word).or_insert(0) += 1;
            acc
        });
    println!("{:?}", freq);

    // Group by first letter
    let words = vec!["apple", "avocado", "banana", "blueberry", "cherry"];
    let grouped: HashMap<char, Vec<&&str>> = words.iter()
        .fold(HashMap::new(), |mut acc, word| {
            let key = word.chars().next().unwrap();
            acc.entry(key).or_insert_with(Vec::new).push(word);
            acc
        });
    println!("{:?}", grouped);
}
```

---

## 5. Collecting into String

```rust
fn main() {
    // Collect chars into String
    let s: String = vec!['h', 'e', 'l', 'l', 'o'].into_iter().collect();
    println!("{s}");  // "hello"

    // Collect &str into String
    let words = vec!["hello", " ", "world"];
    let joined: String = words.into_iter().collect();
    println!("{joined}");  // "hello world"

    // Filter chars
    let cleaned: String = "h3ll0 w0rld".chars()
        .filter(|c| c.is_alphabetic() || c.is_whitespace())
        .collect();
    println!("{cleaned}");  // "hll wrld"

    // Transform chars
    let upper: String = "hello".chars()
        .map(|c| c.to_uppercase().next().unwrap())
        .collect();
    println!("{upper}");  // "HELLO"
}
```

---

## 6. Collecting Results — The Magic Trick

This is the roadmap practice task. `collect()` can turn `Iterator<Item = Result<T, E>>` into `Result<Vec<T>, E>`:

```rust
fn main() {
    // Success case — all parse correctly
    let strings = vec!["1", "2", "3", "4", "5"];
    let numbers: Result<Vec<i32>, _> = strings.iter()
        .map(|s| s.parse::<i32>())
        .collect();
    println!("All good: {:?}", numbers);  // Ok([1, 2, 3, 4, 5])

    // Failure case — stops at first error
    let mixed = vec!["1", "two", "3"];
    let result: Result<Vec<i32>, _> = mixed.iter()
        .map(|s| s.parse::<i32>())
        .collect();
    println!("With error: {:?}", result);  // Err(ParseIntError)

    // Collect Options similarly: Iterator<Item=Option<T>> → Option<Vec<T>>
    let opts = vec![Some(1), Some(2), Some(3)];
    let all: Option<Vec<i32>> = opts.into_iter().collect();
    println!("All Some: {:?}", all);  // Some([1, 2, 3])

    let with_none = vec![Some(1), None, Some(3)];
    let failed: Option<Vec<i32>> = with_none.into_iter().collect();
    println!("Has None: {:?}", failed);  // None
}
```

### Real-world: reading multiple files

```rust
use std::fs;

fn read_configs(paths: &[&str]) -> Result<Vec<String>, std::io::Error> {
    paths.iter()
        .map(|path| fs::read_to_string(path))
        .collect()  // Result<Vec<String>, io::Error>
}
```

---

## 7. partition — Split into Two

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    let (evens, odds): (Vec<i32>, Vec<i32>) = numbers.iter()
        .partition(|&&x| x % 2 == 0);

    println!("Evens: {:?}", evens);  // [2, 4, 6, 8, 10]
    println!("Odds:  {:?}", odds);   // [1, 3, 5, 7, 9]

    // Partition Results
    let data = vec!["1", "bad", "3", "oops", "5"];
    let (oks, errs): (Vec<_>, Vec<_>) = data.iter()
        .map(|s| s.parse::<i32>())
        .partition(|r| r.is_ok());

    let oks: Vec<i32> = oks.into_iter().map(|r| r.unwrap()).collect();
    println!("Parsed: {:?}", oks);  // [1, 3, 5]
    println!("Errors: {}", errs.len());  // 2
}
```

---

## 8. unzip — Split Pairs

```rust
fn main() {
    let pairs = vec![(1, 'a'), (2, 'b'), (3, 'c')];
    let (nums, chars): (Vec<i32>, Vec<char>) = pairs.into_iter().unzip();
    println!("Numbers: {:?}", nums);   // [1, 2, 3]
    println!("Chars:   {:?}", chars);  // ['a', 'b', 'c']
}
```

---

## 9. The FromIterator Trait

`collect()` works via the `FromIterator` trait. You can implement it for your own types:

```rust
struct Sentence {
    words: Vec<String>,
}

impl std::iter::FromIterator<String> for Sentence {
    fn from_iter<I: IntoIterator<Item = String>>(iter: I) -> Self {
        Sentence {
            words: iter.into_iter().collect(),
        }
    }
}

impl std::fmt::Display for Sentence {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{}", self.words.join(" "))
    }
}

fn main() {
    let sentence: Sentence = vec!["Hello", "world", "from", "Rust"]
        .into_iter()
        .map(String::from)
        .collect();

    println!("{sentence}");  // "Hello world from Rust"
    println!("Words: {}", sentence.words.len());  // 4
}
```

---

## 10. Summary Cheat Sheet

```
COLLECTING INTO TYPES
────────────────────────────────────────────────────────────
.collect::<Vec<_>>()        gather into Vec
.collect::<HashSet<_>>()    gather into HashSet (dedup)
.collect::<BTreeSet<_>>()   gather into BTreeSet (sorted)
.collect::<HashMap<_,_>>()  from (K,V) pairs
.collect::<String>()        from chars or &str

THE MAGIC COLLECT
────────────────────────────────────────────────────────────
Iterator<Result<T,E>> → Result<Vec<T>, E>   stops at first Err
Iterator<Option<T>>   → Option<Vec<T>>      returns None if any None

SPLITTING
────────────────────────────────────────────────────────────
.partition(pred)    split into (matching, not_matching)
.unzip()           split (A,B) pairs into (Vec<A>, Vec<B>)

TURBOFISH
────────────────────────────────────────────────────────────
.collect::<Type>()     specify target type inline
Use _ for element type inference: Vec<_>, HashMap<_,_>

FromIterator TRAIT
────────────────────────────────────────────────────────────
impl FromIterator<T> for MyType → enables .collect::<MyType>()
```

---

## What's Next?

**Lesson 36 — Custom Error Types** — Build rich, descriptive error types for your applications. Learn `std::error::Error`, `Display`, and the `thiserror` crate.

## Further Reading
- [Iterator::collect](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect)
- [FromIterator trait](https://doc.rust-lang.org/std/iter/trait.FromIterator.html)

---

*collect() — the Swiss army knife of iterator consumption! 🦀*
