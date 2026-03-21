# 🧪 Lesson 03 — Practice Questions: Compound Types

> **Instructions:** Work through each question before checking answers.
> Answers are in [`lesson_03_answers.md`](./lesson_03_answers.md)

---

## Section A — Predict the Output

Read carefully and write down what you think prints. Then run it to check.

---

### Q1 — Tuple Access

```rust
fn main() {
    let data = (100, 3.14, 'R', false);

    println!("{}", data.0);
    println!("{}", data.1);
    println!("{}", data.2);
    println!("{}", data.3);
    println!("{:?}", data);
}
```

**What are the five output lines?**

---

### Q2 — Destructuring

```rust
fn main() {
    let point = (8, -3, 15);
    let (x, y, z) = point;

    println!("x={x}");
    println!("y={y}");
    println!("z={z}");
    println!("sum={}", x + y + z);
}
```

**What are the four output lines?**

---

### Q3 — Nested Tuple Access

```rust
fn main() {
    let nested = ((10, 20), (30, 40));

    println!("{}", nested.0.0);
    println!("{}", nested.0.1);
    println!("{}", nested.1.0);
    println!("{}", nested.1.1);

    let sum = nested.0.0 + nested.0.1 + nested.1.0 + nested.1.1;
    println!("{sum}");
}
```

**What are the five output lines?**

---

### Q4 — Array Basics

```rust
fn main() {
    let arr = [5, 10, 15, 20, 25];

    println!("{}", arr[0]);
    println!("{}", arr[4]);
    println!("{}", arr.len());
    println!("{:?}", arr);
}
```

**What are the four output lines?**

---

### Q5 — Array Repeat Syntax

```rust
fn main() {
    let a = [3; 5];
    let b = [true; 3];
    let c = [0u8; 4];

    println!("{:?}", a);
    println!("{:?}", b);
    println!("{:?}", c);
    println!("{}", a.len() + b.len() + c.len());
}
```

**What are the four output lines?**

---

### Q6 — Mutable Tuple and Array

```rust
fn main() {
    let mut t = (1, 2, 3);
    t.0 += 10;
    t.2 *= 5;
    println!("{:?}", t);

    let mut arr = [1, 2, 3, 4, 5];
    arr[0] = 99;
    arr[4] = 0;
    println!("{:?}", arr);
}
```

**What are the two output lines?**

---

### Q7 — Slices

```rust
fn main() {
    let arr = [10, 20, 30, 40, 50, 60];

    let s1 = &arr[..3];
    let s2 = &arr[3..];
    let s3 = &arr[1..4];
    let s4 = &arr[..];

    println!("{:?}", s1);
    println!("{:?}", s2);
    println!("{:?}", s3);
    println!("{}", s3.len());
    println!("{:?}", s4);
}
```

**What are the five output lines?**

---

### Q8 — Enumerate

```rust
fn main() {
    let fruits = ["apple", "banana", "cherry"];

    for (i, fruit) in fruits.iter().enumerate() {
        println!("{i}: {fruit}");
    }
}
```

**What is the output?**

---

### Q9 — Swap Idiom

```rust
fn main() {
    let mut x = 42;
    let mut y = 99;

    println!("Before: x={x}, y={y}");
    (x, y) = (y, x);
    println!("After:  x={x}, y={y}");
}
```

**What are the two output lines?**

---

### Q10 — 2D Array

```rust
fn main() {
    let grid = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];

    println!("{}", grid[0][0]);
    println!("{}", grid[1][1]);
    println!("{}", grid[2][2]);

    let diagonal_sum = grid[0][0] + grid[1][1] + grid[2][2];
    println!("Diagonal sum: {diagonal_sum}");
}
```

**What are the four output lines?**

---

## Section B — Fix the Bugs

Each snippet contains one or more errors. Identify every bug, explain it, and write a corrected version.

---

### Q11 — Wrong Tuple Index

```rust
fn main() {
    let t = (10, 20, 30);
    println!("{}", t[1]);
}
```

---

### Q12 — Missing Type on Const Array

```rust
fn main() {
    const DAYS_IN_MONTHS = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
    println!("{}", DAYS_IN_MONTHS[0]);
}
```

---

### Q13 — Array Length Mismatch

```rust
fn main() {
    let arr: [i32; 4] = [1, 2, 3];
    println!("{:?}", arr);
}
```

---

### Q14 — Out-of-Bounds Access

```rust
fn main() {
    let arr = [10, 20, 30];
    let last = arr[3]; // trying to get the last element
    println!("{last}");
}
```

**Fix it two ways: (a) correct the index, (b) use `.get()` with a match.**

---

### Q15 — Iterating and Mutating

```rust
fn main() {
    let arr = [1, 2, 3, 4, 5];
    for n in arr.iter() {
        *n *= 2;
    }
    println!("{:?}", arr);
}
```

**What are the two bugs here?**

---

### Q16 — Single-Element Tuple vs Expression

```rust
fn main() {
    let single: (i32,) = (42);
    println!("{:?}", single);
}
```

---

### Q17 — Mixed Types in Array

```rust
fn main() {
    let mixed = [1, 2.0, 3, 4, 5];
    println!("{:?}", mixed);
}
```

---

### Q18 — Function Accepting Wrong Array Size

```rust
fn first_three(arr: [i32; 3]) -> i32 {
    arr[0] + arr[1] + arr[2]
}

fn main() {
    let nums = [10, 20, 30, 40, 50];
    println!("{}", first_three(nums));
}
```

**Fix it two ways: (a) use a slice parameter, (b) pass only the first 3 elements.**

---

## Section C — Write It Yourself

---

### Q19 — RGB Color Mixer

Write a program that:
- Defines three `(u8, u8, u8)` tuples named `red`, `green`, `blue` as pure colors:
  - `red   = (255, 0, 0)`
  - `green = (0, 255, 0)`
  - `blue  = (0, 0, 255)`
- Creates a `mixed` color by averaging the R, G, B channels of all three colors
- Prints each original color and the mixed result with a label

Expected output format:
```
red:   (255, 0, 0)
green: (0, 255, 0)
blue:  (0, 0, 255)
mixed: (85, 85, 85)
```

---

### Q20 — Statistics Calculator

Write a program that:
- Defines an array of 8 `f64` values: `[72.5, 88.0, 91.5, 65.0, 78.5, 95.0, 83.5, 70.0]`
- Calculates and prints:
  - The **sum** of all values
  - The **average** (mean)
  - The **minimum** value
  - The **maximum** value
  - The **range** (max - min)
  - The **count** of values above the average

Print each result clearly labelled, floats with 2 decimal places.

---

### Q21 — Matrix Transpose

A matrix transpose flips rows and columns. Given a 2×3 matrix, the transpose is 3×2.

Write a program that:
- Defines a 2×3 `i32` matrix:
  ```
  [[1, 2, 3],
   [4, 5, 6]]
  ```
- Manually creates the 3×2 transposed matrix (no loops needed — you can hardcode the assignments)
- Prints the original and transposed matrix, row by row

Original:
```
1 2 3
4 5 6
```
Transposed:
```
1 4
2 5
3 6
```

---

### Q22 — FizzBuzz Array

Write a program that:
- Creates an `[&str; 20]` array (or builds the results some other way)
- Fills it with FizzBuzz values for numbers 1–20:
  - Divisible by both 3 and 5 → `"FizzBuzz"`
  - Divisible by 3 only → `"Fizz"`
  - Divisible by 5 only → `"Buzz"`
  - Otherwise → use a fixed placeholder (since array elements must be the same type, use the number... but `&str` complicates this)

> **Hint:** This is tricky with a fixed `&str` array since you can't store both `&str` and numbers. Instead, print each result directly inside a loop from `1..=20` without storing in an array. Focus on the FizzBuzz logic itself.

---

### Q23 — Coordinate Distance

Write a program that:
- Defines two 3D points as tuples: `p1: (f64, f64, f64)` and `p2: (f64, f64, f64)`
- Calculates the Euclidean distance between them:
  `distance = sqrt((x2-x1)² + (y2-y1)² + (z2-z1)²)`
- Destructures both tuples to get named components
- Prints both points and the distance with 4 decimal places

Test with: `p1 = (1.0, 2.0, 3.0)` and `p2 = (4.0, 6.0, 3.0)`
Expected distance: `5.0000`

---

### Q24 — Safe Lookup Table

Write a program that:
- Defines a `const` array `PLANET_NAMES: [&str; 8]` containing the 8 planets in order from the Sun
- Writes code to safely look up a planet by its number (1-indexed, so Mercury = 1)
- Uses `.get()` to handle the lookup safely
- Tests with planet numbers: `3`, `8`, `0`, `9`
- Prints: `"Planet 3 is Earth"` or `"Planet 0 does not exist"` etc.

---

### Q25 — Running Totals

Write a program that:
- Has a `[i32; 6]` array of sales figures: `[120, 340, 280, 510, 190, 430]`
- Creates a second array (or prints directly) of **running totals** — where each position holds the sum of all previous elements plus the current one
- Prints both the original array and the running totals

Expected output:
```
Sales:   [120, 340, 280, 510, 190, 430]
Running: [120, 460, 740, 1250, 1440, 1870]
```

---

### Q26 — Tuple Struct Simulation

Since you haven't learned structs yet, simulate a "student record" using a tuple `(u32, &str, f64, char)` for `(id, name, gpa, grade)`.

Write a program that:
- Creates an array of 4 student records (array of tuples)
- Iterates over all students
- For each student prints: `"ID: X | Name: Y | GPA: Z.ZZ | Grade: G"`
- After the loop, prints the **highest GPA** found among all students

Students:
```
(1001, "Alice",   3.92, 'A')
(1002, "Bob",     3.45, 'B')
(1003, "Charlie", 2.87, 'C')
(1004, "Diana",   3.98, 'A')
```

---

## Section D — Deep Understanding

---

### Q27 — Predict and Explain

What does this print? Trace each step. Pay attention to what's happening to each binding.

```rust
fn main() {
    let arr = [1, 2, 3, 4, 5];

    let sum: i32 = arr.iter().sum();
    let doubled_sum: i32 = arr.iter().map(|x| x * 2).sum();

    let evens: Vec<_> = arr.iter().filter(|&&x| x % 2 == 0).collect();
    let odds: Vec<_>  = arr.iter().filter(|&&x| x % 2 != 0).collect();

    println!("Sum:          {sum}");
    println!("Doubled sum:  {doubled_sum}");
    println!("Evens: {:?}", evens);
    println!("Odds:  {:?}", odds);
}
```

---

### Q28 — True or False?

For each statement, say True or False and explain why.

1. A tuple can hold elements of different types.
2. You can iterate over a tuple with a `for` loop.
3. `[i32; 3]` and `[i32; 4]` are the same type.
4. `(42)` is a single-element tuple.
5. Array indices in Rust start at 1.
6. `.get(i)` on an array returns `Option<&T>`.
7. You can resize an array after it is created.
8. `&arr[1..3]` gives a slice containing elements at indices 1 and 2.
9. A `&[T]` slice stores both a pointer and a length.
10. You can compare two tuples of the same type with `==`.
11. `let a: [i32; 0] = [];` is valid Rust.
12. Accessing `arr[99]` on a 3-element array causes undefined behavior.

---

### Q29 — Big Debug Challenge

The following program has **8 bugs**. Find and fix every one.

```rust
fn main() {
    // A student's exam data: (name, scores array, passed)
    let student = ("Morgan", [85, 92, 78, 95, 88], true)

    // Print name
    println!("Student: {}", student[0]);

    // Print scores
    println!("Scores: {:?}", student[1]);

    // Calculate average
    let scores = student.1;
    let total: i32 = scores.iter().sum;
    let avg = total / scores.len();

    println!("Average: {avg}");

    // Check highest score
    let best = scores.iter().max();
    println!("Best score: {best}");

    // Update a score (suppose the third exam was re-graded to 82)
    scores[2] = 82;
    println!("Updated scores: {:?}", scores);
}
```

**List every bug, explain it, and provide the corrected program.**

---

### Q30 — Design Challenge: Pick the Right Structure

For each scenario, decide: **tuple**, **array**, **both**, or **neither (use Vec)**? Justify each answer.

1. A 3D vector in a physics engine: x, y, z coordinates (all `f64`)
2. The 7 days of the week as strings
3. A user profile: id (`u32`), username (`&str`), age (`u8`), active (`bool`)
4. A list of student names where you add names as students enroll
5. The RGB value of a color: red, green, blue (all `u8`)
6. Chess board positions — 8×8 grid of squares
7. The result of an integer division: quotient and remainder
8. Pixel data for a 1920×1080 image
9. The top 10 high scores in a game (fixed, always 10)
10. Historical stock prices for the past N days (N unknown at compile time)

---

*"The compiler is not your enemy — it's a very strict but fair teacher." 🦀*
