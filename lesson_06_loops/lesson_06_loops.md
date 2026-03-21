# 📘 Lesson 06 — Control Flow: Loops in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Beginner → Intermediate
> **Prerequisite:** [Lesson 05 — if / else](../lesson_05/lesson_05_if_else.md)
>
> **Files in this lesson:**
> - [`lesson_06_loops.md`](./lesson_06_loops.md) ← You are here
> - [`lesson_06_questions.md`](./lesson_06_questions.md) — Practice problems
> - [`lesson_06_answers.md`](./lesson_06_answers.md) — Solutions & explanations

---

## Table of Contents

1. [Why Loops?](#1-why-loops)
2. [The `loop` Expression](#2-the-loop-expression)
   - 2.1 [Basic `loop`](#21-basic-loop)
   - 2.2 [Breaking Out — `break`](#22-breaking-out--break)
   - 2.3 [Returning Values from `loop`](#23-returning-values-from-loop)
   - 2.4 [Loop Labels](#24-loop-labels)
3. [The `while` Loop](#3-the-while-loop)
   - 3.1 [Basic `while`](#31-basic-while)
   - 3.2 [`while` vs `loop` + `break`](#32-while-vs-loop--break)
   - 3.3 [`while let`](#33-while-let)
4. [The `for` Loop](#4-the-for-loop)
   - 4.1 [Ranges — Exclusive and Inclusive](#41-ranges--exclusive-and-inclusive)
   - 4.2 [Iterating Collections](#42-iterating-collections)
   - 4.3 [`.iter()` vs `.iter_mut()` vs `.into_iter()`](#43-iter-vs-iter_mut-vs-into_iter)
   - 4.4 [`.enumerate()` — Index + Value](#44-enumerate--index--value)
   - 4.5 [Destructuring in `for`](#45-destructuring-in-for)
5. [`continue` — Skip an Iteration](#5-continue--skip-an-iteration)
6. [Nested Loops and Labels](#6-nested-loops-and-labels)
7. [Iterator Adaptors](#7-iterator-adaptors)
   - 7.1 [`.map()`](#71-map)
   - 7.2 [`.filter()`](#72-filter)
   - 7.3 [`.fold()` and `.reduce()`](#73-fold-and-reduce)
   - 7.4 [`.take()` and `.skip()`](#74-take-and-skip)
   - 7.5 [`.zip()` and `.chain()`](#75-zip-and-chain)
   - 7.6 [Chaining Adaptors](#76-chaining-adaptors)
8. [Choosing the Right Loop](#8-choosing-the-right-loop)
9. [Common Loop Patterns](#9-common-loop-patterns)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. Why Loops?

A loop **repeats a block of code** multiple times. Without loops, every repetition must be written manually — which is impossible for runtime-determined counts, impractical for large data, and fragile to maintain.

Rust has **three loop constructs**, each designed for a specific context:

| Loop | Use when |
|------|----------|
| `loop` | Repeat until an explicit `break`; can return a value |
| `while` | Repeat while a boolean condition is true |
| `for` | Iterate over a sequence, range, or collection |

---

## 2. The `loop` Expression

### 2.1 Basic `loop`

`loop` runs its body **forever** until you tell it to stop. It is the lowest-level loop form.

```rust
fn main() {
    let mut count = 0;

    loop {
        count += 1;
        println!("count = {count}");
        if count == 3 {
            break;
        }
    }
    println!("Done!");
}
// Output:
// count = 1
// count = 2
// count = 3
// Done!
```

### 2.2 Breaking Out — `break`

`break` exits the **innermost** loop immediately:

```rust
fn main() {
    let mut n = 0;
    loop {
        n += 1;
        if n % 7 == 0 {
            println!("First multiple of 7: {n}");
            break;
        }
    }
}
// Output: First multiple of 7: 7
```

### 2.3 Returning Values from `loop`

`loop` is an **expression** — you can send a value back through `break`:

```rust
fn main() {
    let mut attempt = 0;

    let result = loop {
        attempt += 1;
        if attempt >= 3 {
            break attempt * 10; // break carries a value
        }
        println!("Attempt #{attempt}");
    };

    println!("Result: {result}"); // Result: 30
}
// Output:
// Attempt #1
// Attempt #2
// Result: 30
```

This is the **only** loop form that can return a value — `while` and `for` cannot.

### 2.4 Loop Labels

Labels let `break` and `continue` target a specific outer loop:

```rust
fn main() {
    'outer: loop {
        let mut i = 0;
        loop {
            i += 1;
            if i == 5 {
                break 'outer; // exits the outer loop
            }
            println!("inner i = {i}");
        }
    }
    println!("finished");
}
```

Labels always begin with a single quote: `'label_name`.

---

## 3. The `while` Loop

### 3.1 Basic `while`

Runs while its boolean condition is `true`:

```rust
fn main() {
    let mut n = 1;
    while n < 100 {
        n *= 2;
    }
    println!("First power of 2 >= 100: {n}"); // 128
}
```

The condition is checked **before** each iteration — if false from the start, the body never executes.

### 3.2 `while` vs `loop` + `break`

Both are equivalent in power, but clarity differs:

```rust
// while — condition up front, easy to read
let mut i = 0;
while i < 5 { println!("{i}"); i += 1; }

// loop + break — better when: condition is complex, value must be returned,
// or the body must run at least once before the check
let mut i = 0;
loop {
    println!("{i}");
    i += 1;
    if i >= 5 { break; }
}
```

### 3.3 `while let`

Loops while a **pattern** continues to match — especially useful with iterators and `Option`:

```rust
fn main() {
    let mut stack = vec![1, 2, 3, 4, 5];

    while let Some(top) = stack.pop() {
        println!("Popped: {top}");
    }
    println!("Empty: {}", stack.is_empty());
}
// Output:
// Popped: 5
// Popped: 4
// Popped: 3
// Popped: 2
// Popped: 1
// Empty: true
```

`stack.pop()` returns `Some(v)` while elements remain and `None` when exhausted. The loop runs exactly as long as the pattern matches.

---

## 4. The `for` Loop

`for` is the most commonly used loop in Rust. It iterates over anything implementing the `Iterator` trait.

### 4.1 Ranges — Exclusive and Inclusive

```rust
fn main() {
    for i in 0..5      { print!("{i} "); } // 0 1 2 3 4   (exclusive end)
    println!();
    for i in 1..=5     { print!("{i} "); } // 1 2 3 4 5   (inclusive end)
    println!();
    for i in (1..=5).rev()  { print!("{i} "); } // 5 4 3 2 1
    println!();
    for i in (0..20).step_by(4) { print!("{i} "); } // 0 4 8 12 16
    println!();
}
```

### 4.2 Iterating Collections

```rust
fn main() {
    let fruits = ["apple", "banana", "cherry"];
    for fruit in fruits {
        println!("{fruit}");
    }

    let numbers = vec![10, 20, 30];
    for n in &numbers {       // borrow — numbers is still usable after
        println!("{n}");
    }
    println!("Still have: {:?}", numbers);
}
```

### 4.3 `.iter()` vs `.iter_mut()` vs `.into_iter()`

| Method | Yields | Consumes? | Modify? |
|--------|--------|-----------|---------|
| `.iter()` | `&T` | No | No |
| `.iter_mut()` | `&mut T` | No | Yes |
| `.into_iter()` | `T` | Yes | N/A |

```rust
fn main() {
    let mut data = vec![1, 2, 3];

    // iter() — read only
    for x in data.iter() { print!("{x} "); }
    println!();

    // iter_mut() — modify in place
    for x in data.iter_mut() { *x *= 10; }
    println!("{data:?}"); // [10, 20, 30]

    // into_iter() — consumes the vec
    for x in data.into_iter() { print!("{x} "); }
    // data is moved; can't use it anymore
    println!();
}
```

### 4.4 `.enumerate()` — Index + Value

```rust
fn main() {
    let planets = ["Mercury", "Venus", "Earth", "Mars"];
    for (i, planet) in planets.iter().enumerate() {
        println!("Planet {}: {}", i + 1, planet);
    }
}
// Output:
// Planet 1: Mercury
// Planet 2: Venus
// Planet 3: Earth
// Planet 4: Mars
```

### 4.5 Destructuring in `for`

```rust
fn main() {
    let points = [(0, 0), (3, 4), (1, -1)];
    for (x, y) in points {
        let dist = ((x*x + y*y) as f64).sqrt();
        println!("({x},{y}) → {dist:.2}");
    }

    let names = ["Alice", "Bob"];
    let scores = [92, 78];
    for (name, score) in names.iter().zip(scores.iter()) {
        println!("{name}: {score}");
    }
}
```

---

## 5. `continue` — Skip an Iteration

`continue` skips the rest of the current iteration's body and moves to the next:

```rust
fn main() {
    // Print only odd numbers
    for i in 1..=10 {
        if i % 2 == 0 { continue; }
        print!("{i} ");
    }
    println!(); // 1 3 5 7 9

    // Sum only positives
    let data = [3, -1, 7, -4, 2, -9, 5];
    let mut sum = 0;
    for &val in data.iter() {
        if val < 0 { continue; }
        sum += val;
    }
    println!("Positive sum: {sum}"); // 17
}
```

`continue` also accepts labels to target outer loops:

```rust
fn main() {
    'outer: for i in 0..3 {
        for j in 0..3 {
            if j == 1 { continue 'outer; }
            println!("({i}, {j})");
        }
    }
}
// Output:
// (0, 0)
// (1, 0)
// (2, 0)
```

---

## 6. Nested Loops and Labels

```rust
fn main() {
    // Multiplication table
    for i in 1..=4 {
        for j in 1..=4 {
            print!("{:4}", i * j);
        }
        println!();
    }
}
```

**Searching with labeled break:**

```rust
fn main() {
    let matrix = [[1,2,3],[4,5,6],[7,8,9]];
    let target = 6;
    let mut found = None;

    'search: for (r, row) in matrix.iter().enumerate() {
        for (c, &val) in row.iter().enumerate() {
            if val == target {
                found = Some((r, c));
                break 'search; // stop all searching
            }
        }
    }

    match found {
        Some((r, c)) => println!("Found {target} at ({r},{c})"),
        None => println!("Not found"),
    }
}
// Output: Found 6 at (1,2)
```

---

## 7. Iterator Adaptors

Rust's iterator system lets you build **transformation pipelines** without mutation. Adaptors are **lazy** — no computation happens until a consumer (`.collect()`, `.sum()`, etc.) is called.

### 7.1 `.map()`

Transform every element:

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5];

    let doubled: Vec<i32> = numbers.iter().map(|&x| x * 2).collect();
    println!("{doubled:?}"); // [2, 4, 6, 8, 10]

    let labels: Vec<String> = numbers.iter()
        .map(|&x| format!("item_{x}"))
        .collect();
    println!("{labels:?}"); // ["item_1", "item_2", ...]
}
```

### 7.2 `.filter()`

Keep only elements matching a predicate:

```rust
fn main() {
    let nums = vec![1, 2, 3, 4, 5, 6, 7, 8];

    let evens: Vec<i32> = nums.iter().copied().filter(|&x| x % 2 == 0).collect();
    println!("{evens:?}"); // [2, 4, 6, 8]

    let words = vec!["hi", "hello", "rust", "ok"];
    let long: Vec<&&str> = words.iter().filter(|w| w.len() > 3).collect();
    println!("{long:?}"); // ["hello", "rust"]
}
```

### 7.3 `.fold()` and `.reduce()`

Accumulate elements into one value:

```rust
fn main() {
    let nums = vec![1, 2, 3, 4, 5];

    let sum  = nums.iter().fold(0,  |acc, &x| acc + x);
    let prod = nums.iter().fold(1,  |acc, &x| acc * x);
    let max  = nums.iter().copied().reduce(|a, b| if a > b { a } else { b });

    println!("sum={sum}, product={prod}, max={max:?}");
    // sum=15, product=120, max=Some(5)
}
```

### 7.4 `.take()` and `.skip()`

```rust
fn main() {
    let data: Vec<i32> = (1..=10).collect();

    let first3: Vec<i32> = data.iter().copied().take(3).collect();
    let last5:  Vec<i32> = data.iter().copied().skip(5).collect();
    let page:   Vec<i32> = data.iter().copied().skip(2).take(3).collect();

    println!("{first3:?}"); // [1, 2, 3]
    println!("{last5:?}");  // [6, 7, 8, 9, 10]
    println!("{page:?}");   // [3, 4, 5]

    let ascending: Vec<i32> = data.iter().copied().take_while(|&x| x < 5).collect();
    println!("{ascending:?}"); // [1, 2, 3, 4]
}
```

### 7.5 `.zip()` and `.chain()`

```rust
fn main() {
    let a = vec![1, 2, 3];
    let b = vec![10, 20, 30];

    // zip — pair elements from two iterators
    let pairs: Vec<_> = a.iter().zip(b.iter()).collect();
    println!("{pairs:?}"); // [(1, 10), (2, 20), (3, 30)]

    // chain — concatenate two iterators
    let combined: Vec<i32> = a.iter().chain(b.iter()).copied().collect();
    println!("{combined:?}"); // [1, 2, 3, 10, 20, 30]
}
```

### 7.6 Chaining Adaptors

```rust
fn main() {
    let students = vec![
        ("Alice", 92.0), ("Bob", 54.0), ("Charlie", 78.0), ("Diana", 88.0),
    ];

    // Names of passing students, uppercase, sorted
    let mut names: Vec<String> = students.iter()
        .filter(|(_, score)| *score >= 60.0)
        .map(|(name, _)| name.to_uppercase())
        .collect();
    names.sort();

    println!("{names:?}");
    // ["ALICE", "CHARLIE", "DIANA"]

    // Average of passing scores
    let passing: Vec<f64> = students.iter()
        .filter(|(_, s)| *s >= 60.0)
        .map(|(_, s)| *s)
        .collect();
    let avg = passing.iter().sum::<f64>() / passing.len() as f64;
    println!("Avg passing: {avg:.2}"); // 86.00
}
```

---

## 8. Choosing the Right Loop

```
Know the collection or range up front?   → for
Clear boolean condition each iteration?  → while
Need to return a value from the loop?    → loop
Loop body must run at least once first?  → loop (do-while equivalent)
Transforming/filtering data cleanly?     → iterator adaptors
```

---

## 9. Common Loop Patterns

### Pattern 1 — Accumulate with `fold`
```rust
let sum_of_squares: u32 = (1..=10).map(|i| i * i).sum();
println!("{sum_of_squares}"); // 385
```

### Pattern 2 — Find first match
```rust
fn first_over(data: &[i32], threshold: i32) -> Option<i32> {
    data.iter().copied().find(|&x| x > threshold)
}
println!("{:?}", first_over(&[3, 8, 1, 12, 5], 10)); // Some(12)
```

### Pattern 3 — Collatz sequence
```rust
fn collatz(mut n: u64) -> Vec<u64> {
    let mut seq = vec![n];
    while n != 1 {
        n = if n % 2 == 0 { n / 2 } else { 3 * n + 1 };
        seq.push(n);
    }
    seq
}
println!("{:?}", collatz(6)); // [6, 3, 10, 5, 16, 8, 4, 2, 1]
```

### Pattern 4 — Retry with `loop`
```rust
fn main() {
    let mut attempts = 0;
    let success = loop {
        attempts += 1;
        if attempts == 3 { break true; }
        println!("Attempt {attempts} failed, retrying...");
    };
    println!("Success: {success} after {attempts} attempts");
}
```

---

## 10. Summary Cheat Sheet

```rust
// ── loop ──────────────────────────────────────
loop { /* forever */ }
let v = loop { break value; };      // returns value
'label: loop { break 'label; }      // labeled

// ── while ─────────────────────────────────────
while condition { }
while let Some(x) = iter.next() { } // pattern-driven

// ── for ───────────────────────────────────────
for i in 0..5      { }  // 0,1,2,3,4
for i in 0..=5     { }  // 0,1,2,3,4,5
for i in (0..5).rev() { }
for i in (0..20).step_by(3) { }
for x in &vec      { }  // borrow
for x in &mut vec  { }  // mutable borrow
for (i,v) in arr.iter().enumerate() { }
for (a,b) in iter1.zip(iter2) { }

// ── control ───────────────────────────────────
break;            // exit loop
break value;      // exit with value
break 'label;     // exit labeled loop
continue;         // next iteration
continue 'label;  // next outer iteration

// ── iterator adaptors ─────────────────────────
.map(|x| x*2)
.filter(|&x| x>0)
.fold(0, |acc,x| acc+x)
.take(n) / .skip(n)
.take_while(|&x| x<5) / .skip_while(|&x| x<5)
.enumerate()
.zip(other) / .chain(other)
.flat_map(|x| x.iter())
.rev()
// consumers:
.collect::<Vec<_>>()
.sum::<i32>() / .product::<i32>()
.count() / .any(|x| ..) / .all(|x| ..)
.find(|&x| ..) / .position(|&x| ..)
.max() / .min()
```

---

## 📚 Further Reading

- [The Rust Book — Chapter 3.5: Loops](https://doc.rust-lang.org/book/ch03-05-control-flow.html)
- [The Rust Book — Chapter 13.2: Iterators](https://doc.rust-lang.org/book/ch13-02-iterators.html)
- [Rust by Example — Loops](https://doc.rust-lang.org/rust-by-example/flow_control/loop.html)
- [std::iter documentation](https://doc.rust-lang.org/std/iter/)

---

*"The best loop is the one that makes the intent unmistakable." 🦀*
