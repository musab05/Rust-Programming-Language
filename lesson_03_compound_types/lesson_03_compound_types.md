# 📘 Lesson 03 — Data Types: Compound Types in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Beginner → Intermediate
> **Prerequisite:** [Lesson 02 — Scalar Types](../lesson_02/lesson_02_scalar_types.md)
>
> **Files in this lesson:**
> - [`lesson_03_compound_types.md`](./lesson_03_compound_types.md) ← You are here
> - [`lesson_03_questions.md`](./lesson_03_questions.md) — Practice problems
> - [`lesson_03_answers.md`](./lesson_03_answers.md) — Solutions & explanations

---

## Table of Contents

1. [What Are Compound Types?](#1-what-are-compound-types)
2. [Tuples](#2-tuples)
   - 2.1 [Declaring a Tuple](#21-declaring-a-tuple)
   - 2.2 [Accessing Tuple Elements](#22-accessing-tuple-elements)
   - 2.3 [Destructuring a Tuple](#23-destructuring-a-tuple)
   - 2.4 [The Unit Type — ()](#24-the-unit-type---)
   - 2.5 [Nested Tuples](#25-nested-tuples)
   - 2.6 [Mutable Tuples](#26-mutable-tuples)
   - 2.7 [Tuples as Return Values](#27-tuples-as-return-values)
   - 2.8 [Tuple Limitations](#28-tuple-limitations)
3. [Arrays](#3-arrays)
   - 3.1 [Declaring an Array](#31-declaring-an-array)
   - 3.2 [Array Type Syntax](#32-array-type-syntax)
   - 3.3 [Accessing Array Elements](#33-accessing-array-elements)
   - 3.4 [Mutable Arrays](#34-mutable-arrays)
   - 3.5 [Array Initialization Shorthand](#35-array-initialization-shorthand)
   - 3.6 [Array Length](#36-array-length)
   - 3.7 [Iterating Over Arrays](#37-iterating-over-arrays)
   - 3.8 [Out-of-Bounds Access](#38-out-of-bounds-access)
   - 3.9 [Multidimensional Arrays](#39-multidimensional-arrays)
   - 3.10 [Array Slices](#310-array-slices)
4. [Tuples vs Arrays — When to Use Which](#4-tuples-vs-arrays--when-to-use-which)
5. [Arrays vs Vectors — A Preview](#5-arrays-vs-vectors--a-preview)
6. [Common Patterns and Idioms](#6-common-patterns-and-idioms)
7. [Summary Cheat Sheet](#7-summary-cheat-sheet)

---

## 1. What Are Compound Types?

A **compound type** groups **multiple values** into a single type. Unlike scalar types (which hold exactly one value), compound types let you bundle related data together.

Rust has two **primitive** compound types:

| Type | Elements | Types of elements | Size |
|------|----------|-------------------|------|
| **Tuple** | Fixed count | Can be **different** types | Fixed at compile time |
| **Array** | Fixed count | Must be the **same** type | Fixed at compile time |

```rust
// Scalar — one value
let age: i32 = 25;

// Tuple — multiple values, mixed types
let person: (i32, &str, bool) = (25, "Alice", true);

// Array — multiple values, same type
let scores: [i32; 5] = [95, 87, 72, 100, 88];
```

Both are **stack-allocated** — their sizes are known at compile time, so Rust stores them directly on the stack (fast, no heap allocation needed).

---

## 2. Tuples

A **tuple** is an ordered, fixed-size collection of values where each value can have a **different type**.

Think of a tuple as a lightweight record or a mini-struct — it groups related pieces of data that belong together but may be of different types.

### 2.1 Declaring a Tuple

```rust
fn main() {
    // Basic tuple
    let point = (3, 7);

    // Typed tuple
    let point: (i32, i32) = (3, 7);

    // Mixed types
    let user: (u32, &str, f64, bool) = (1, "Alice", 4.8, true);
    //         id    name  rating active

    // Single-element tuple — the trailing comma is REQUIRED
    let single: (i32,) = (42,);
    // Without the comma: (42) is just a parenthesized expression, NOT a tuple

    println!("{:?}", point);   // (3, 7)
    println!("{:?}", user);    // (1, "Alice", 4.8, true)
    println!("{:?}", single);  // (42,)
}
```

> **`{:?}` — Debug formatting:** The normal `{}` formatter (Display) doesn't work on tuples. Use `{:?}` (Debug) or `{:#?}` (pretty-printed Debug) instead.

### 2.2 Accessing Tuple Elements

Access individual elements using **dot notation** with a zero-based index:

```rust
fn main() {
    let color: (u8, u8, u8) = (255, 128, 0); // RGB: orange

    let red   = color.0;
    let green = color.1;
    let blue  = color.2;

    println!("R: {red}, G: {green}, B: {blue}");
    // R: 255, G: 128, B: 0

    // Direct access in expressions
    println!("Red channel: {}", color.0);
}
```

**Why `.0`, `.1`, `.2`?** Tuple fields don't have names — they're accessed by their positional index. This is different from structs (which you'll learn later) where fields have names.

### 2.3 Destructuring a Tuple

**Destructuring** lets you unpack a tuple's values into separate named variables in one line:

```rust
fn main() {
    let point = (10, 25, -5);

    // Destructure into separate variables
    let (x, y, z) = point;

    println!("x={x}, y={y}, z={z}"); // x=10, y=25, z=-5
}
```

You can also **ignore** elements you don't need using `_`:

```rust
fn main() {
    let record = (42, "important", 3.14, true);

    // Only extract the second and fourth elements
    let (_, label, _, is_active) = record;

    println!("{label} is active: {is_active}");
    // important is active: true
}
```

### Destructuring in for loops

```rust
fn main() {
    let points = [(0, 0), (1, 5), (10, -3)];

    for (x, y) in points {
        println!("Point: ({x}, {y})");
    }
}
```

**Output:**
```
Point: (0, 0)
Point: (1, 5)
Point: (10, -3)
```

### 2.4 The Unit Type — `()`

The **unit type** `()` is a special tuple with **zero elements**. It represents "nothing" or "no meaningful value."

```rust
fn main() {
    let nothing: () = ();         // the unit value
    println!("{:?}", nothing);    // ()
}
```

You'll see `()` frequently in Rust:
- Functions that don't return a value implicitly return `()`
- It's the return type shown in error messages for void functions
- `Result<(), Error>` means "success has no value, but failure has an error"

```rust
fn print_hello() {       // implicit return type is ()
    println!("Hello!");
}

fn print_hello_explicit() -> () {  // same thing, explicit
    println!("Hello!");
}
```

### 2.5 Nested Tuples

Tuples can contain other tuples:

```rust
fn main() {
    let nested = ((1, 2), (3, 4), (5, 6));

    println!("{:?}", nested);       // ((1, 2), (3, 4), (5, 6))
    println!("{}", nested.0.0);     // 1
    println!("{}", nested.0.1);     // 2
    println!("{}", nested.2.0);     // 5

    let (a, b, c) = nested;
    println!("{:?} {:?} {:?}", a, b, c); // (1, 2) (3, 4) (5, 6)

    let ((x1, y1), _, (x3, y3)) = nested;
    println!("First: ({x1},{y1}), Third: ({x3},{y3})");
    // First: (1,2), Third: (5,6)
}
```

### 2.6 Mutable Tuples

If a tuple is declared with `mut`, you can change its element values (but not the tuple's structure or types):

```rust
fn main() {
    let mut position = (0, 0);
    println!("Start: {:?}", position); // (0, 0)

    position.0 += 5;
    position.1 -= 3;
    println!("Moved: {:?}", position); // (5, -3)

    // You CAN reassign element values
    position.0 = 100;

    // You CANNOT change the type of an element
    // position.0 = "hello"; // ❌ ERROR: expected i32, found &str
}
```

### 2.7 Tuples as Return Values

One of the most common uses of tuples is **returning multiple values** from a function:

```rust
fn min_max(a: i32, b: i32, c: i32) -> (i32, i32) {
    let min = a.min(b).min(c);
    let max = a.max(b).max(c);
    (min, max) // return a tuple — no semicolon = expression = return value
}

fn divide(dividend: f64, divisor: f64) -> (f64, f64) {
    let quotient  = dividend / divisor;
    let remainder = dividend % divisor;
    (quotient, remainder)
}

fn main() {
    let (lo, hi) = min_max(3, 1, 7);
    println!("Min: {lo}, Max: {hi}"); // Min: 1, Max: 7

    let (q, r) = divide(17.0, 5.0);
    println!("17 / 5 = {q} remainder {r}"); // 17 / 5 = 3.4 remainder 2
}
```

### 2.8 Tuple Limitations

Tuples are powerful but have limits you should know:

```rust
// ✅ Tuples support equality if all elements support it
let a = (1, "hello", true);
let b = (1, "hello", true);
println!("{}", a == b); // true

// ✅ Tuples support comparison (lexicographic)
let p = (1, 5);
let q = (1, 6);
println!("{}", p < q); // true

// ❌ You cannot iterate over a tuple (elements may have different types)
// for item in (1, "hello", true) { ... } // does NOT compile

// ❌ Tuple size is limited (Rust supports up to 12-element tuples for traits)
// Very large tuples lose trait implementations like Debug, PartialEq etc.

// ❌ No named fields — for complex data, use a struct instead
// tuple.0, tuple.1 can get confusing; structs are clearer
```

---

## 3. Arrays

An **array** is a fixed-size collection of values where **every element has the same type**. Arrays live on the **stack**.

### 3.1 Declaring an Array

```rust
fn main() {
    // Type inferred
    let primes = [2, 3, 5, 7, 11];

    // Explicit type annotation: [type; length]
    let primes: [i32; 5] = [2, 3, 5, 7, 11];

    // Mixed with let mut
    let mut temps: [f64; 4] = [36.5, 37.0, 38.2, 36.8];

    println!("{:?}", primes); // [2, 3, 5, 7, 11]
    println!("{:?}", temps);  // [36.5, 37.0, 38.2, 36.8]
}
```

### 3.2 Array Type Syntax

The type of an array is written as `[T; N]` where:
- `T` = the element type
- `N` = the **exact** number of elements (a compile-time constant)

```rust
let a: [i32; 3]   = [1, 2, 3];      // array of 3 i32s
let b: [bool; 2]  = [true, false];   // array of 2 bools
let c: [f64; 0]   = [];              // empty array — valid!
let d: [char; 4]  = ['R', 'u', 's', 't'];
```

The length `N` is **part of the type**. This means `[i32; 3]` and `[i32; 4]` are **different types** and cannot be used interchangeably.

```rust
fn takes_three(arr: [i32; 3]) {
    println!("{:?}", arr);
}

fn main() {
    let three = [1, 2, 3];
    let four  = [1, 2, 3, 4];

    takes_three(three); // ✅
    takes_three(four);  // ❌ ERROR: expected [i32; 3], found [i32; 4]
}
```

### 3.3 Accessing Array Elements

Use **square bracket indexing** with a `usize` index (zero-based):

```rust
fn main() {
    let days = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];

    println!("{}", days[0]); // Mon — first element
    println!("{}", days[4]); // Fri — fifth element
    println!("{}", days[6]); // Sun — last element

    // Index with a variable (must be usize or cast to usize)
    let i: usize = 2;
    println!("{}", days[i]); // Wed

    // Last element using len() - 1
    let last = days[days.len() - 1];
    println!("{}", last); // Sun
}
```

### 3.4 Mutable Arrays

With `mut`, you can change individual element values:

```rust
fn main() {
    let mut scores = [70, 80, 90, 85, 75];
    println!("Before: {:?}", scores); // [70, 80, 90, 85, 75]

    scores[0] = 95; // update first element
    scores[4] += 5; // increment last element

    println!("After:  {:?}", scores); // [95, 80, 90, 85, 80]
}
```

### 3.5 Array Initialization Shorthand

To create an array where **every element has the same value**, use the repeat syntax `[value; count]`:

```rust
fn main() {
    // [value; count] — repeat syntax
    let zeros    = [0; 10];       // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    let trues    = [true; 5];     // [true, true, true, true, true]
    let dashes   = ['-'; 20];     // 20 dash characters

    println!("{:?}", zeros);   // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    println!("{:?}", trues);   // [true, true, true, true, true]

    // A buffer of bytes — common in low-level code
    let buffer: [u8; 1024] = [0; 1024]; // 1KB of zeros
    println!("Buffer length: {}", buffer.len()); // 1024
}
```

### 3.6 Array Length

```rust
fn main() {
    let arr = [10, 20, 30, 40, 50];

    println!("Length: {}", arr.len()); // 5

    // Length is available as a constant
    let len = arr.len(); // returns usize

    // Common pattern — last valid index
    let last_index = arr.len() - 1; // 4
    println!("Last element: {}", arr[last_index]); // 50

    // Empty array
    let empty: [i32; 0] = [];
    println!("Empty length: {}", empty.len()); // 0
    println!("Empty is_empty: {}", empty.is_empty()); // true
}
```

### 3.7 Iterating Over Arrays

There are several ways to loop over an array:

```rust
fn main() {
    let nums = [10, 20, 30, 40, 50];

    // ── Method 1: for..in (iterates by value for Copy types) ──
    for n in nums {
        print!("{n} "); // 10 20 30 40 50
    }
    println!();

    // ── Method 2: .iter() — iterates by reference ──
    for n in nums.iter() {
        print!("{n} "); // 10 20 30 40 50
    }
    println!();

    // ── Method 3: enumerate — get index AND value ──
    for (i, n) in nums.iter().enumerate() {
        println!("nums[{i}] = {n}");
    }
    // nums[0] = 10
    // nums[1] = 20  ... etc.

    // ── Method 4: index range ──
    for i in 0..nums.len() {
        print!("{} ", nums[i]); // 10 20 30 40 50
    }
    println!();

    // ── Method 5: mut iter — modify elements while iterating ──
    let mut data = [1, 2, 3, 4, 5];
    for n in data.iter_mut() {
        *n *= 2; // dereference with * to modify
    }
    println!("{:?}", data); // [2, 4, 6, 8, 10]
}
```

> **Tip:** Prefer `for n in nums` or `for n in nums.iter()` over manual index loops — it's more idiomatic and safer.

### 3.8 Out-of-Bounds Access

Unlike C/C++, Rust **always checks array bounds at runtime** and panics rather than accessing invalid memory:

```rust
fn main() {
    let arr = [1, 2, 3];
    println!("{}", arr[5]); // ❌ RUNTIME PANIC
}
```

**Error:**
```
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 5'
```

This is one of Rust's safety guarantees — **no undefined behavior** from out-of-bounds memory access. In C, this would silently read garbage memory or crash unpredictably.

**Safe way to access — `.get()` returns an `Option`:**

```rust
fn main() {
    let arr = [10, 20, 30];

    // .get() returns Option<&T> — Some if in bounds, None if not
    match arr.get(1) {
        Some(val) => println!("Found: {val}"),  // Found: 20
        None      => println!("Out of bounds"),
    }

    match arr.get(99) {
        Some(val) => println!("Found: {val}"),
        None      => println!("Out of bounds"), // Out of bounds
    }

    // Shorthand with if let
    if let Some(val) = arr.get(2) {
        println!("Third: {val}"); // Third: 30
    }
}
```

### 3.9 Multidimensional Arrays

Arrays can be nested to create 2D (or higher-dimensional) structures:

```rust
fn main() {
    // 2D array: 3 rows, 4 columns
    let matrix: [[i32; 4]; 3] = [
        [1,  2,  3,  4],
        [5,  6,  7,  8],
        [9, 10, 11, 12],
    ];

    // Access element at row 1, column 2
    println!("{}", matrix[1][2]); // 7

    // Print entire matrix
    for row in matrix {
        for val in row {
            print!("{:3}", val);
        }
        println!();
    }
    // Output:
    //   1  2  3  4
    //   5  6  7  8
    //   9 10 11 12

    // Tic-tac-toe board example
    let board: [[char; 3]; 3] = [
        ['X', 'O', 'X'],
        ['O', 'X', 'O'],
        ['O', 'X', 'X'],
    ];

    println!("Center: {}", board[1][1]); // X
}
```

### 3.10 Array Slices

A **slice** is a reference to a **contiguous portion** of an array (or any contiguous sequence). Slices are one of Rust's most important concepts.

```rust
fn main() {
    let arr = [10, 20, 30, 40, 50];

    // A slice of the entire array
    let all: &[i32] = &arr;

    // A slice of elements 1 through 3 (indices 1, 2, 3)
    let mid: &[i32] = &arr[1..4]; // [20, 30, 40]

    // Slice from start
    let first_three = &arr[..3]; // [10, 20, 30]

    // Slice to end
    let last_two = &arr[3..];    // [40, 50]

    println!("{:?}", all);         // [10, 20, 30, 40, 50]
    println!("{:?}", mid);         // [20, 30, 40]
    println!("{:?}", first_three); // [10, 20, 30]
    println!("{:?}", last_two);    // [40, 50]

    println!("Slice length: {}", mid.len()); // 3
}
```

**Slice type is `&[T]`** — it's a *reference* to a sequence of `T` values. It stores:
1. A pointer to the first element
2. The length of the slice

```rust
fn sum_slice(slice: &[i32]) -> i32 {
    let mut total = 0;
    for &val in slice {
        total += val;
    }
    total
}

fn main() {
    let arr = [1, 2, 3, 4, 5];

    println!("{}", sum_slice(&arr));      // 15 — whole array
    println!("{}", sum_slice(&arr[1..4])); // 9  — just [2, 3, 4]
    println!("{}", sum_slice(&arr[..2]));  // 3  — just [1, 2]
}
```

> **Why slices?** Functions that accept `&[T]` can work with arrays of **any length**, vectors, and other sequence types — making them far more flexible than functions taking `[T; N]` (which only accepts one specific size).

---

## 4. Tuples vs Arrays — When to Use Which

This is a common question. Here's a clear breakdown:

| Feature | Tuple | Array |
|---------|-------|-------|
| Element types | Can be **different** | Must be **same** |
| Access | `.0`, `.1`, `.2` | `[0]`, `[1]`, `[2]` |
| Iteration | ❌ Cannot iterate | ✅ Can iterate |
| Indexing with variable | ❌ No (index must be literal) | ✅ Yes |
| Primary use | Group **related but different** data | Collection of **same-type** data |
| Typical size | Small (2–5 elements) | Can be larger |
| Named fields | ❌ (use struct instead) | ❌ (use struct/vec instead) |

### Use a **tuple** when:
- Returning multiple values of different types from a function
- Grouping two or three related values temporarily (e.g., `(x, y)` coordinates)
- The data has a clear "this is the first thing, this is the second thing" structure

### Use an **array** when:
- You have a fixed collection of the **same type**
- You need to iterate over all elements
- You need random access by index
- The number of elements is known and fixed (days of week, months, board game grid)

```rust
// ✅ Tuple: different types, small, no iteration needed
let user_info: (&str, u32, bool) = ("Alice", 30, true);

// ✅ Array: same type, iterate, fixed count
let weekday_names = ["Mon", "Tue", "Wed", "Thu", "Fri"];

// ✅ Tuple: return multiple values
fn bounding_box() -> (f64, f64, f64, f64) { (0.0, 0.0, 100.0, 50.0) }

// ✅ Array: collection of scores, grades, temperatures
let test_scores: [u32; 5] = [88, 92, 75, 100, 85];
```

---

## 5. Arrays vs Vectors — A Preview

You might wonder: *"What if I need a collection that can grow or shrink?"*

That's what `Vec<T>` (a **Vector**) is for. Here's a preview:

| Feature | Array `[T; N]` | Vector `Vec<T>` |
|---------|---------------|-----------------|
| Size | **Fixed** at compile time | **Dynamic** at runtime |
| Memory | **Stack** | **Heap** |
| Syntax | `[1, 2, 3]` | `vec![1, 2, 3]` |
| Add elements | ❌ No | ✅ `.push()` |
| Remove elements | ❌ No | ✅ `.pop()` |
| Performance | Slightly faster | Slightly slower (heap alloc) |
| Best when | Size is known & fixed | Size varies or unknown |

```rust
fn main() {
    // Array — fixed, stack
    let arr: [i32; 3] = [1, 2, 3];

    // Vector — dynamic, heap (preview — covered in a later lesson)
    let mut vec: Vec<i32> = vec![1, 2, 3];
    vec.push(4); // now [1, 2, 3, 4]
    vec.push(5); // now [1, 2, 3, 4, 5]
    println!("{:?}", vec);
}
```

**Rule of thumb:** Use arrays when the number of elements is **known at compile time and won't change**. Use vectors when the number of elements is **determined at runtime or can change**.

---

## 6. Common Patterns and Idioms

### Pattern 1 — Swap two values using a tuple

```rust
fn main() {
    let mut a = 10;
    let mut b = 20;

    println!("Before: a={a}, b={b}"); // a=10, b=20

    (a, b) = (b, a); // swap! elegant and idiomatic

    println!("After:  a={a}, b={b}"); // a=20, b=10
}
```

### Pattern 2 — Named tuple-like data with type alias

```rust
type Point2D = (f64, f64);
type RgbColor = (u8, u8, u8);

fn distance(p1: Point2D, p2: Point2D) -> f64 {
    let dx = p2.0 - p1.0;
    let dy = p2.1 - p1.1;
    (dx * dx + dy * dy).sqrt()
}

fn main() {
    let origin: Point2D = (0.0, 0.0);
    let point:  Point2D = (3.0, 4.0);

    println!("Distance: {}", distance(origin, point)); // 5.0

    let red:   RgbColor = (255, 0,   0);
    let green: RgbColor = (0,   255, 0);
    println!("Red:   {:?}", red);
    println!("Green: {:?}", green);
}
```

### Pattern 3 — Find with index using enumerate

```rust
fn main() {
    let names = ["Alice", "Bob", "Charlie", "Diana"];
    let target = "Charlie";

    let mut found_index: Option<usize> = None;

    for (i, &name) in names.iter().enumerate() {
        if name == target {
            found_index = Some(i);
            break;
        }
    }

    match found_index {
        Some(i) => println!("Found '{target}' at index {i}"), // index 2
        None    => println!("'{target}' not found"),
    }
}
```

### Pattern 4 — Array as a lookup table

```rust
fn main() {
    // Months lookup table — index 0 is unused (month 1 = January)
    const MONTH_NAMES: [&str; 13] = [
        "",          // index 0 — placeholder
        "January", "February", "March",
        "April",   "May",      "June",
        "July",    "August",   "September",
        "October", "November", "December",
    ];

    let month_num = 7;
    println!("Month {month_num} is {}", MONTH_NAMES[month_num]);
    // Month 7 is July
}
```

### Pattern 5 — Functional operations on arrays (with iterators)

```rust
fn main() {
    let scores = [85, 92, 78, 95, 88, 72, 90];

    // Sum all scores
    let total: i32 = scores.iter().sum();

    // Average
    let avg = total as f64 / scores.len() as f64;

    // Maximum
    let max = scores.iter().max().unwrap();

    // Minimum
    let min = scores.iter().min().unwrap();

    // Count passing scores (>= 80)
    let passing = scores.iter().filter(|&&s| s >= 80).count();

    println!("Total:   {total}");
    println!("Average: {avg:.1}");
    println!("Max:     {max}");
    println!("Min:     {min}");
    println!("Passing: {passing} out of {}", scores.len());
}
```

**Output:**
```
Total:   600
Average: 85.7
Max:     95
Min:     72
Passing: 5 out of 7
```

### Pattern 6 — Unzip a tuple array

```rust
fn main() {
    let pairs = [(1, 'a'), (2, 'b'), (3, 'c')];

    // Access first elements
    for (num, _) in pairs {
        print!("{num} "); // 1 2 3
    }
    println!();

    // Access second elements
    for (_, ch) in pairs {
        print!("{ch} "); // a b c
    }
    println!();
}
```

---

## 7. Summary Cheat Sheet

```rust
// ══════════════════════════════════════════════════
// TUPLES
// ══════════════════════════════════════════════════

// Declaration
let t: (i32, &str, bool) = (1, "hello", true);
let single: (i32,) = (42,);   // single-element needs trailing comma
let unit: () = ();             // unit type — zero elements

// Access by index
let x = t.0;  // 1
let y = t.1;  // "hello"
let z = t.2;  // true

// Destructuring
let (a, b, c) = t;
let (_, label, _) = t;  // ignore with _

// Mutable
let mut pos = (0, 0);
pos.0 = 10;
pos.1 = -5;

// Tuple as return value
fn swap(a: i32, b: i32) -> (i32, i32) { (b, a) }
let (x, y) = swap(1, 2); // x=2, y=1

// Swap idiom
(a, b) = (b, a);

// Debug print
println!("{:?}", t);   // (1, "hello", true)


// ══════════════════════════════════════════════════
// ARRAYS
// ══════════════════════════════════════════════════

// Declaration
let arr: [i32; 5] = [1, 2, 3, 4, 5];
let repeat = [0; 10];          // [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
let empty: [i32; 0] = [];

// Access
let first = arr[0];            // 1
let last  = arr[arr.len()-1];  // 5

// Mutable
let mut arr = [1, 2, 3];
arr[0] = 99;

// Length
let len = arr.len();           // usize
let is_empty = arr.is_empty(); // bool

// Safe access — returns Option
let val = arr.get(2);          // Some(&3)
let oob = arr.get(99);         // None

// Iteration
for x in arr { }               // by value (Copy types)
for x in arr.iter() { }        // by reference
for (i, x) in arr.iter().enumerate() { }  // index + value
for x in arr.iter_mut() { *x *= 2; }      // mutable

// Slices
let slice: &[i32] = &arr;      // whole array as slice
let mid  = &arr[1..3];         // elements at index 1, 2
let head = &arr[..2];          // first 2 elements
let tail = &arr[2..];          // from index 2 to end

// 2D array
let grid: [[i32; 3]; 2] = [[1,2,3],[4,5,6]];
let cell = grid[0][1];         // 2

// Iterator methods
let sum: i32 = arr.iter().sum();
let max = arr.iter().max().unwrap();
let min = arr.iter().min().unwrap();
let count = arr.iter().filter(|&&x| x > 2).count();
```

---

## 🔗 What's Next?

- **Lesson 04 — Functions**
- **Lesson 05 — Control Flow - If Else**
- **Lesson 06 — Control Flow - Loops**
- **Lesson 07 — Comments & Documentation**
- **Lesson 08 — Ownerships and Rules**

---

## 📚 Further Reading

- [The Rust Book — Chapter 3.2: Compound Types](https://doc.rust-lang.org/book/ch03-02-data-types.html#compound-types)
- [Rust by Example — Tuples](https://doc.rust-lang.org/rust-by-example/primitives/tuples.html)
- [Rust by Example — Arrays and Slices](https://doc.rust-lang.org/rust-by-example/primitives/array.html)
- [Rust Reference — Tuple Types](https://doc.rust-lang.org/reference/types/tuple.html)
- [Rust Reference — Array Types](https://doc.rust-lang.org/reference/types/array.html)

---

*You're building a strong foundation. Keep going, Rustacean! 🦀*
