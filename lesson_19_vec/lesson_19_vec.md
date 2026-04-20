# 📘 Lesson 19 — Vec\<T\> (C2)

> **Series:** Rust From Zero · Beginner Level  
> **Roadmap ID:** C2 · Category: 📚 Collections  
> **Previous:** [Lesson 18 — String vs &str](./lesson_18_string_vs_str.md)  
> **Next:** [Lesson 20 — HashMap<K,V>](./lesson_20_hashmap.md)  
> **Practice:** [Questions](./lesson_19_questions.md) · [Answers](./lesson_19_answers.md)  
> **Practice Task:** Median and mode of a Vec<i32>

---

## Table of Contents

1. [What Is Vec\<T\>?](#1-what-is-vect)
2. [Creating Vectors](#2-creating-vectors)
3. [Adding and Removing Elements](#3-adding-and-removing-elements)
4. [Reading Elements — Indexing vs .get()](#4-reading-elements--indexing-vs-get)
5. [Iterating Over a Vec](#5-iterating-over-a-vec)
6. [Slicing a Vec](#6-slicing-a-vec)
7. [Sorting and Searching](#7-sorting-and-searching)
8. [Useful Vec Methods](#8-useful-vec-methods)
9. [Vec of Enums — Storing Mixed Types](#9-vec-of-enums--storing-mixed-types)
10. [Vec and Ownership](#10-vec-and-ownership)
11. [Performance — Capacity and Reallocation](#11-performance--capacity-and-reallocation)
12. [Summary Cheat Sheet](#12-summary-cheat-sheet)

---

## 1. What Is Vec\<T\>?

`Vec<T>` ("vector") is Rust's **growable, heap-allocated array**. It stores values of type `T` contiguously in memory:

```
Vec<i32>: [1][2][3][4][5]  ← contiguous heap memory
           ptr: 0x...
           len: 5
           cap: 8
```

- `len` — number of elements currently stored
- `cap` — number of elements that fit before reallocation
- Elements are stored contiguously — fast random access and iteration

---

## 2. Creating Vectors

```rust
fn main() {
    // Empty Vec — type must be annotated
    let v: Vec<i32> = Vec::new();

    // With initial values — vec! macro infers type
    let v2 = vec![1, 2, 3, 4, 5];

    // With capacity pre-allocated
    let mut v3: Vec<String> = Vec::with_capacity(10);

    // Repeated value: [0, 0, 0, 0, 0]
    let zeros = vec![0i32; 5];

    // From an iterator
    let squares: Vec<i32> = (1..=5).map(|x| x * x).collect();
    println!("{:?}", squares); // [1, 4, 9, 16, 25]
}
```

---

## 3. Adding and Removing Elements

```rust
fn main() {
    let mut v = vec![1, 2, 3];

    v.push(4);                  // append to end
    v.push(5);
    println!("{:?}", v);        // [1, 2, 3, 4, 5]

    let last = v.pop();         // remove from end → Option<T>
    println!("{:?}", last);     // Some(5)

    v.insert(0, 99);            // insert at index 0
    println!("{:?}", v);        // [99, 1, 2, 3, 4]

    v.remove(0);                // remove at index 0, shifts right
    println!("{:?}", v);        // [1, 2, 3, 4]

    v.retain(|&x| x % 2 == 0); // keep only even
    println!("{:?}", v);        // [2, 4]

    v.clear();                  // remove all
    println!("{:?}", v);        // []
    println!("{}", v.is_empty()); // true
}
```

---

## 4. Reading Elements — Indexing vs `.get()`

```rust
fn main() {
    let v = vec![10, 20, 30, 40, 50];

    // Direct indexing — PANICS if out of bounds
    println!("{}", v[2]);   // 30
    // println!("{}", v[99]); // PANIC!

    // .get() — returns Option<&T>, never panics
    println!("{:?}", v.get(2));   // Some(30)
    println!("{:?}", v.get(99));  // None

    // Safe pattern
    if let Some(val) = v.get(2) {
        println!("found: {val}");
    }

    // first() and last()
    println!("{:?}", v.first()); // Some(10)
    println!("{:?}", v.last());  // Some(50)
}
```

**Rule:** Use `v[i]` when you're certain the index is valid; use `v.get(i)` when it might not be.

---

## 5. Iterating Over a Vec

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];

    // Immutable iteration — yields &T
    for x in &v {
        print!("{x} ");
    }
    println!();

    // Mutable iteration — yields &mut T
    let mut v2 = vec![1, 2, 3];
    for x in &mut v2 {
        *x *= 2;
    }
    println!("{:?}", v2); // [2, 4, 6]

    // Consuming iteration — yields T (Vec is moved)
    let v3 = vec![10, 20, 30];
    for x in v3 {          // v3 is moved
        print!("{x} ");
    }
    println!();

    // Indexed iteration
    let v4 = vec!["a", "b", "c"];
    for (i, val) in v4.iter().enumerate() {
        println!("{i}: {val}");
    }
}
```

---

## 6. Slicing a Vec

A slice `&[T]` is a view into part or all of a Vec:

```rust
fn sum(data: &[i32]) -> i32 { data.iter().sum() }

fn main() {
    let v = vec![1, 2, 3, 4, 5];

    let all:   &[i32] = &v;          // whole Vec as slice
    let first3: &[i32] = &v[..3];    // [1, 2, 3]
    let last2:  &[i32] = &v[3..];    // [4, 5]
    let mid:    &[i32] = &v[1..4];   // [2, 3, 4]

    println!("{}", sum(&v));           // 15
    println!("{}", sum(first3));       // 6
    println!("{}", sum(last2));        // 9
}
```

---

## 7. Sorting and Searching

```rust
fn main() {
    let mut v = vec![3, 1, 4, 1, 5, 9, 2, 6];

    // Sort ascending
    v.sort();
    println!("{:?}", v); // [1, 1, 2, 3, 4, 5, 6, 9]

    // Sort descending
    v.sort_by(|a, b| b.cmp(a));
    println!("{:?}", v);

    // Sort by key
    let mut words = vec!["banana", "apple", "cherry", "date"];
    words.sort_by_key(|s| s.len());
    println!("{:?}", words); // ["date", "apple", "banana", "cherry"]

    // Floats need sort_by + partial_cmp
    let mut floats = vec![3.1, 1.4, 2.7, 0.5];
    floats.sort_by(|a, b| a.partial_cmp(b).unwrap());
    println!("{:?}", floats);

    // Binary search (slice must be sorted first!)
    let sorted = vec![1, 3, 5, 7, 9];
    println!("{:?}", sorted.binary_search(&5));  // Ok(2)
    println!("{:?}", sorted.binary_search(&4));  // Err(2) — where 4 would go

    // Linear search
    println!("{:?}", v.iter().position(|&x| x == 5)); // Some(index)
    println!("{}", v.contains(&5));
}
```

---

## 8. Useful Vec Methods

```rust
fn main() {
    let mut v = vec![1, 2, 3, 2, 1];

    // Length and capacity
    println!("len={} cap={}", v.len(), v.capacity());

    // Deduplication (requires sorted!)
    v.sort(); v.dedup();
    println!("{:?}", v); // [1, 2, 3]

    // Extend — append all elements from an iterator
    let more = vec![4, 5, 6];
    v.extend(more.iter());
    println!("{:?}", v); // [1, 2, 3, 4, 5, 6]

    // Concatenate two Vecs
    let a = vec![1, 2];
    let mut b = vec![3, 4];
    b.extend_from_slice(&a);
    println!("{:?}", b); // [3, 4, 1, 2]

    // Split at index
    let data = vec![1, 2, 3, 4, 5];
    let (left, right) = data.split_at(3);
    println!("{:?} {:?}", left, right); // [1,2,3] [4,5]

    // Windows and chunks
    for w in data.windows(3) { print!("{w:?} "); } println!();
    for c in data.chunks(2)  { print!("{c:?} "); } println!();

    // Drain — remove a range and get an iterator
    let mut v2 = vec![1, 2, 3, 4, 5];
    let drained: Vec<i32> = v2.drain(1..3).collect();
    println!("drained={:?} remaining={:?}", drained, v2);
    // drained=[2,3] remaining=[1,4,5]

    // Truncate / resize
    let mut v3 = vec![1, 2, 3, 4, 5];
    v3.truncate(3);
    println!("{:?}", v3); // [1, 2, 3]
}
```

---

## 9. Vec of Enums — Storing Mixed Types

A Vec must hold one type, but an enum can represent multiple shapes:

```rust
#[derive(Debug)]
enum Value { Int(i32), Float(f64), Text(String) }

fn main() {
    let mixed: Vec<Value> = vec![
        Value::Int(42),
        Value::Float(3.14),
        Value::Text(String::from("hello")),
        Value::Int(-1),
    ];

    for v in &mixed {
        match v {
            Value::Int(n)   => println!("int: {n}"),
            Value::Float(f) => println!("float: {f:.2}"),
            Value::Text(s)  => println!("text: {s}"),
        }
    }
}
```

---

## 10. Vec and Ownership

A Vec owns its elements. When the Vec is dropped, so are all its elements:

```rust
fn main() {
    {
        let v = vec![String::from("hello"), String::from("world")];
    } // v dropped here — Strings inside are also dropped

    // Moving elements out
    let mut v = vec![String::from("a"), String::from("b")];
    let first = v.remove(0);   // moves "a" out
    println!("{first}");
    println!("{:?}", v);       // ["b"]

    // Borrow vs move
    let v2 = vec![1, 2, 3];
    let x = v2[0];             // i32 is Copy — copied, not moved
    println!("{x} {:?}", v2); // 1 [1, 2, 3]

    let v3 = vec![String::from("hello")];
    // let s = v3[0]; // ❌ can't move out of index — String is not Copy
    let s = &v3[0];   // ✅ borrow it
    println!("{s}");
}
```

---

## 11. Performance — Capacity and Reallocation

```rust
fn main() {
    // Without pre-allocation — multiple reallocations
    let mut v: Vec<i32> = Vec::new();
    for i in 0..100 { v.push(i); }
    println!("final cap: {}", v.capacity()); // some power of 2 ≥ 100

    // With pre-allocation — zero reallocations
    let mut v2: Vec<i32> = Vec::with_capacity(100);
    for i in 0..100 { v2.push(i); }
    println!("cap was always: {}", v2.capacity()); // 100
}
```

Rules of thumb:
- If you know the size in advance, use `Vec::with_capacity(n)`
- Rust doubles capacity on each reallocation (amortised O(1) push)
- `.shrink_to_fit()` reduces capacity to exactly the current length

---

## 12. Summary Cheat Sheet

```
CREATING
────────────────────────────────────────────────────────────
Vec::new()              empty Vec
vec![1, 2, 3]           with initial values
vec![0; n]              n copies of value
Vec::with_capacity(n)   empty with pre-allocated space
(iter).collect()        build from iterator

ADDING / REMOVING
────────────────────────────────────────────────────────────
v.push(x)           append to end
v.pop()             remove from end → Option<T>
v.insert(i, x)      insert at index i (shifts right)
v.remove(i)         remove at index i (shifts left) → T
v.retain(|x| ...)   keep only matching elements
v.clear()           remove all
v.extend(iter)      append all from iterator

READING
────────────────────────────────────────────────────────────
v[i]                direct access — PANICS if out of bounds
v.get(i)            safe access → Option<&T>
v.first()           → Option<&T>
v.last()            → Option<&T>
v.len()             element count
v.is_empty()        bool

ITERATING
────────────────────────────────────────────────────────────
for x in &v         borrows → &T
for x in &mut v     mutable borrows → &mut T
for x in v          moves → T (consumes v)
v.iter().enumerate() (index, &T) pairs

SORTING & SEARCHING
────────────────────────────────────────────────────────────
v.sort()                  ascending (T: Ord)
v.sort_by(|a,b| ...)      custom comparator
v.sort_by_key(|x| ...)    sort by computed key
v.binary_search(&x)       Ok(i) or Err(insert_pos) — sorted!
v.contains(&x)            linear search → bool
v.iter().position(|x| ..) linear → Option<usize>

OTHER
────────────────────────────────────────────────────────────
v.dedup()           remove consecutive duplicates (sort first!)
v.drain(range)      remove range, return iterator
v.truncate(n)       keep only first n elements
v.split_at(i)       (&[..i], &[i..])
v.windows(n)        overlapping sub-slices
v.chunks(n)         non-overlapping sub-slices
```

---

## What's Next?

**Lesson 20 — HashMap\<K,V\>** — Key-value storage, the `entry` API, ownership of keys, and word frequency counting.

## Further Reading
- [The Rust Book — Ch 8.1: Storing Lists of Values with Vectors](https://doc.rust-lang.org/book/ch08-01-vectors.html)
- [std::Vec documentation](https://doc.rust-lang.org/std/vec/struct.Vec.html)
