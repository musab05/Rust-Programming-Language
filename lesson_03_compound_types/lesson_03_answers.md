# ✅ Lesson 03 — Answers: Compound Types

> Full solutions with explanations.
> Questions are in [`lesson_03_questions.md`](./lesson_03_questions.md)

---

## Section A — Predict the Output

---

### A1 — Tuple Access

**Output:**

```
100
3.14
R
false
(100, 3.14, 'R', false)
```

**Explanation:**

- `.0` through `.3` access tuple elements by zero-based positional index
- `{:?}` (Debug format) is required to print a tuple — `{}` (Display) does not work on tuples
- Each element prints according to its own type's formatting

---

### A2 — Destructuring

**Output:**

```
x=8
y=-3
z=15
sum=20
```

**Trace:** `let (x, y, z) = point` unpacks the three elements into separate bindings simultaneously. `8 + (-3) + 15 = 20`.

---

### A3 — Nested Tuple Access

**Output:**

```
10
20
30
40
100
```

**Trace:**

- `nested.0` is `(10, 20)` → `nested.0.0` is `10`, `nested.0.1` is `20`
- `nested.1` is `(30, 40)` → `nested.1.0` is `30`, `nested.1.1` is `40`
- Sum: `10 + 20 + 30 + 40 = 100`

---

### A4 — Array Basics

**Output:**

```
5
25
5
[5, 10, 15, 20, 25]
```

**Explanation:**

- `arr[0]` = first element = `5`
- `arr[4]` = last element = `25` (index 4 on a 5-element array)
- `.len()` = `5`
- `{:?}` prints the full array in debug format

---

### A5 — Array Repeat Syntax

**Output:**

```
[3, 3, 3, 3, 3]
[true, true, true]
[0, 0, 0, 0]
12
```

**Explanation:**

- `[3; 5]` creates an array of five `3`s
- `[true; 3]` creates an array of three `true`s
- `[0u8; 4]` creates an array of four `0u8` values
- `5 + 3 + 4 = 12`

---

### A6 — Mutable Tuple and Array

**Output:**

```
(11, 2, 15)
[99, 2, 3, 4, 0]
```

**Trace:**

- `t.0 += 10` → `1 + 10 = 11`
- `t.2 *= 5` → `3 * 5 = 15`
- `t.1` stays at `2`
- `arr[0] = 99`, `arr[4] = 0`, middle elements unchanged

---

### A7 — Slices

**Output:**

```
[10, 20, 30]
[40, 50, 60]
[20, 30, 40]
3
[10, 20, 30, 40, 50, 60]
```

**Explanation:**

- `&arr[..3]` → indices 0, 1, 2 → `[10, 20, 30]`
- `&arr[3..]` → indices 3, 4, 5 → `[40, 50, 60]`
- `&arr[1..4]` → indices 1, 2, 3 → `[20, 30, 40]`
- `s3.len()` = `3` (three elements: 20, 30, 40)
- `&arr[..]` → all elements → `[10, 20, 30, 40, 50, 60]`

---

### A8 — Enumerate

**Output:**

```
0: apple
1: banana
2: cherry
```

**Explanation:** `.enumerate()` yields `(index, &element)` pairs. `i` starts at 0.

---

### A9 — Swap Idiom

**Output:**

```
Before: x=42, y=99
After:  x=99, y=42
```

**Explanation:** `(x, y) = (y, x)` is a tuple assignment that evaluates the right side first (`(99, 42)`), then destructures it into `x` and `y` simultaneously. This cleanly swaps without a temp variable.

---

### A10 — 2D Array

**Output:**

```
1
5
9
Diagonal sum: 15
```

**Trace:**

- `grid[0][0]` = row 0, col 0 = `1`
- `grid[1][1]` = row 1, col 1 = `5`
- `grid[2][2]` = row 2, col 2 = `9`
- Diagonal sum: `1 + 5 + 9 = 15`

---

## Section B — Fix the Bugs

---

### A11 — Wrong Tuple Index

**Bug:** Tuples use **dot notation** (`.0`, `.1`, `.2`), not square bracket indexing (`[1]`). Square brackets are for arrays.

```rust
// ❌ Original
println!("{}", t[1]); // error: cannot index into a value of type `(i32, i32, i32)`

// ✅ Fixed
fn main() {
    let t = (10, 20, 30);
    println!("{}", t.1); // 20
}
```

---

### A12 — Missing Type on Const Array

**Bug:** `const` items **require** an explicit type annotation. The type cannot be inferred.

```rust
// ❌ Original
const DAYS_IN_MONTHS = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];

// ✅ Fixed
fn main() {
    const DAYS_IN_MONTHS: [u8; 12] = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];
    println!("{}", DAYS_IN_MONTHS[0]); // 31
}
```

---

### A13 — Array Length Mismatch

**Bug:** The type annotation says `[i32; 4]` (4 elements) but the initializer only provides 3. The length in the type and the actual element count must match exactly.

```rust
// ❌ Original
let arr: [i32; 4] = [1, 2, 3]; // error: expected 4 elements, found 3

// ✅ Fixed — Option 1: fix the type annotation
fn main() {
    let arr: [i32; 3] = [1, 2, 3];
    println!("{:?}", arr); // [1, 2, 3]
}

// ✅ Fixed — Option 2: add the missing element
fn main() {
    let arr: [i32; 4] = [1, 2, 3, 4];
    println!("{:?}", arr); // [1, 2, 3, 4]
}
```

---

### A14 — Out-of-Bounds Access

**Bug:** `arr[3]` on a 3-element array (`arr = [10, 20, 30]`). Valid indices are `0`, `1`, `2`. Index `3` is out of bounds and will panic at runtime.

```rust
// ❌ Original
let last = arr[3]; // panic: index out of bounds: the len is 3 but the index is 3

// ✅ Fix (a): correct the index to get the actual last element
fn main() {
    let arr = [10, 20, 30];
    let last = arr[2]; // index 2 = last element
    println!("{last}"); // 30

    // Or use len() - 1 for clarity:
    let last = arr[arr.len() - 1];
    println!("{last}"); // 30
}

// ✅ Fix (b): use .get() for safe access
fn main() {
    let arr = [10, 20, 30];
    match arr.get(2) {
        Some(val) => println!("{val}"), // 30
        None      => println!("Out of bounds"),
    }
}
```

---

### A15 — Iterating and Mutating

**Two bugs:**

1. `arr` is not declared with `mut` — you cannot mutate it
2. `.iter()` yields **immutable** references `&T` — you need `.iter_mut()` to get mutable references

```rust
// ❌ Original
let arr = [1, 2, 3, 4, 5];   // not mut
for n in arr.iter() {          // immutable iterator
    *n *= 2;                   // error: cannot assign to `*n`
}

// ✅ Fixed
fn main() {
    let mut arr = [1, 2, 3, 4, 5]; // fix 1: add mut
    for n in arr.iter_mut() {       // fix 2: use iter_mut()
        *n *= 2;
    }
    println!("{:?}", arr); // [2, 4, 6, 8, 10]
}
```

---

### A16 — Single-Element Tuple vs Expression

**Bug:** `(42)` is just a parenthesized integer expression — it evaluates to `i32`, not a tuple. A single-element tuple **requires a trailing comma**: `(42,)`.

```rust
// ❌ Original
let single: (i32,) = (42); // error: expected tuple, found integer

// ✅ Fixed
fn main() {
    let single: (i32,) = (42,); // trailing comma makes it a tuple
    println!("{:?}", single);   // (42,)
}
```

---

### A17 — Mixed Types in Array

**Bug:** Arrays require **all elements to have the same type**. `1` is an integer (`i32` by default) and `2.0` is a float (`f64`). These are incompatible in one array.

```rust
// ❌ Original
let mixed = [1, 2.0, 3, 4, 5]; // error: expected integer, found floating-point number

// ✅ Fix — make all values the same type (all float)
fn main() {
    let all_float = [1.0, 2.0, 3.0, 4.0, 5.0];
    println!("{:?}", all_float); // [1.0, 2.0, 3.0, 4.0, 5.0]
}

// OR: use a tuple for mixed types (at the cost of losing iteration)
fn main() {
    let mixed = (1, 2.0, 3, 4, 5);
    println!("{:?}", mixed); // (1, 2.0, 3, 4, 5)
}
```

---

### A18 — Function Accepting Wrong Array Size

**Bug:** `first_three` accepts `[i32; 3]` (exactly 3 elements) but `nums` has 5 elements (`[i32; 5]`). These are different types.

```rust
// ❌ Original
fn first_three(arr: [i32; 3]) -> i32 { arr[0] + arr[1] + arr[2] }
// error: expected `[i32; 3]`, found `[i32; 5]`

// ✅ Fix (a): Use a slice parameter — accepts any length
fn first_three(arr: &[i32]) -> i32 {
    arr[0] + arr[1] + arr[2]
}

fn main() {
    let nums = [10, 20, 30, 40, 50];
    println!("{}", first_three(&nums));       // 60 — works!
    println!("{}", first_three(&nums[..3]));  // 60 — also works
}

// ✅ Fix (b): Pass only the first 3 elements as a slice
fn first_three(arr: [i32; 3]) -> i32 {
    arr[0] + arr[1] + arr[2]
}

fn main() {
    let nums = [10, 20, 30, 40, 50];
    // Slice and convert — this doesn't directly work for [i32;3]
    // The cleanest fix is still the slice approach above.
    // If you truly need [i32;3], create it explicitly:
    let first: [i32; 3] = [nums[0], nums[1], nums[2]];
    println!("{}", first_three(first)); // 60
}
```

> **Best practice:** Use `&[T]` (slice) parameters instead of `[T; N]` whenever possible — it makes functions far more flexible and reusable.

---

## Section C — Write It Yourself

---

### A19 — RGB Color Mixer

```rust
fn main() {
    let red:   (u8, u8, u8) = (255, 0,   0);
    let green: (u8, u8, u8) = (0,   255, 0);
    let blue:  (u8, u8, u8) = (0,   0,   255);

    // Average each channel across all three colors
    let mixed_r = ((red.0 as u16 + green.0 as u16 + blue.0 as u16) / 3) as u8;
    let mixed_g = ((red.1 as u16 + green.1 as u16 + blue.1 as u16) / 3) as u8;
    let mixed_b = ((red.2 as u16 + green.2 as u16 + blue.2 as u16) / 3) as u8;
    let mixed: (u8, u8, u8) = (mixed_r, mixed_g, mixed_b);

    println!("red:   {:?}", red);
    println!("green: {:?}", green);
    println!("blue:  {:?}", blue);
    println!("mixed: {:?}", mixed);
}
```

**Output:**

```
red:   (255, 0, 0)
green: (0, 255, 0)
blue:  (0, 0, 255)
mixed: (85, 85, 85)
```

> **Note:** We cast to `u16` before adding to prevent `u8` overflow — `255 + 255 + 255 = 765` overflows `u8` (max 255) but fits in `u16`.

---

### A20 — Statistics Calculator

```rust
fn main() {
    let data: [f64; 8] = [72.5, 88.0, 91.5, 65.0, 78.5, 95.0, 83.5, 70.0];

    let sum: f64    = data.iter().sum();
    let avg: f64    = sum / data.len() as f64;
    let max: f64    = data.iter().cloned().fold(f64::NEG_INFINITY, f64::max);
    let min: f64    = data.iter().cloned().fold(f64::INFINITY, f64::min);
    let range: f64  = max - min;
    let above_avg   = data.iter().filter(|&&x| x > avg).count();

    println!("Count:       {}", data.len());
    println!("Sum:         {:.2}", sum);
    println!("Average:     {:.2}", avg);
    println!("Minimum:     {:.2}", min);
    println!("Maximum:     {:.2}", max);
    println!("Range:       {:.2}", range);
    println!("Above avg:   {}", above_avg);
}
```

**Output:**

```
Count:       8
Sum:         644.00
Average:     80.50
Minimum:     65.00
Maximum:     95.00
Range:       30.00
Above avg:   4
```

---

### A21 — Matrix Transpose

```rust
fn main() {
    let original: [[i32; 3]; 2] = [
        [1, 2, 3],
        [4, 5, 6],
    ];

    // Transpose: original[r][c] becomes transposed[c][r]
    let transposed: [[i32; 2]; 3] = [
        [original[0][0], original[1][0]],  // [1, 4]
        [original[0][1], original[1][1]],  // [2, 5]
        [original[0][2], original[1][2]],  // [3, 6]
    ];

    println!("Original (2×3):");
    for row in original {
        for val in row {
            print!("{:3}", val);
        }
        println!();
    }

    println!("\nTransposed (3×2):");
    for row in transposed {
        for val in row {
            print!("{:3}", val);
        }
        println!();
    }
}
```

**Output:**

```
Original (2×3):
  1  2  3
  4  5  6

Transposed (3×2):
  1  4
  2  5
  3  6
```

---

### A22 — FizzBuzz Array

```rust
fn main() {
    for n in 1..=20 {
        let result = if n % 15 == 0 {
            "FizzBuzz".to_string()
        } else if n % 3 == 0 {
            "Fizz".to_string()
        } else if n % 5 == 0 {
            "Buzz".to_string()
        } else {
            n.to_string()
        };
        println!("{n:2}: {result}");
    }
}
```

**Output:**

```
 1: 1
 2: 2
 3: Fizz
 4: 4
 5: Buzz
 6: Fizz
 7: 7
 8: 8
 9: Fizz
10: Buzz
11: 11
12: Fizz
13: 13
14: 14
15: FizzBuzz
16: 16
17: 17
18: Fizz
19: 19
20: Buzz
```

> **Why not a `[&str; 20]` array?** Because `&str` can only hold string literals known at compile time. The numbers (like "1", "2") would need to be computed at runtime, which produces `String`, not `&str`. This is a great lesson in Rust's ownership model — you'll learn about `String` vs `&str` in depth later.

---

### A23 — Coordinate Distance

```rust
fn main() {
    let p1: (f64, f64, f64) = (1.0, 2.0, 3.0);
    let p2: (f64, f64, f64) = (4.0, 6.0, 3.0);

    let (x1, y1, z1) = p1;
    let (x2, y2, z2) = p2;

    let dx = x2 - x1;
    let dy = y2 - y1;
    let dz = z2 - z1;

    let distance = (dx * dx + dy * dy + dz * dz).sqrt();

    println!("Point 1: ({x1}, {y1}, {z1})");
    println!("Point 2: ({x2}, {y2}, {z2})");
    println!("Distance: {:.4}", distance);
}
```

**Output:**

```
Point 1: (1, 2, 3)
Point 2: (4, 6, 3)
Distance: 5.0000
```

**Verification:** `sqrt((4-1)² + (6-2)² + (3-3)²)` = `sqrt(9 + 16 + 0)` = `sqrt(25)` = `5.0` ✅

---

### A24 — Safe Lookup Table

```rust
fn main() {
    const PLANET_NAMES: [&str; 8] = [
        "Mercury", "Venus", "Earth", "Mars",
        "Jupiter", "Saturn", "Uranus", "Neptune",
    ];

    let test_numbers: [usize; 4] = [3, 8, 0, 9];

    for &num in test_numbers.iter() {
        // Convert 1-indexed to 0-indexed, then use .get() for safety
        if num == 0 {
            println!("Planet {num} does not exist");
        } else {
            match PLANET_NAMES.get(num - 1) {
                Some(&name) => println!("Planet {num} is {name}"),
                None        => println!("Planet {num} does not exist"),
            }
        }
    }
}
```

**Output:**

```
Planet 3 is Earth
Planet 8 is Neptune
Planet 0 does not exist
Planet 9 does not exist
```

---

### A25 — Running Totals

```rust
fn main() {
    let sales: [i32; 6] = [120, 340, 280, 510, 190, 430];
    let mut running: [i32; 6] = [0; 6];

    let mut cumulative = 0;
    for (i, &sale) in sales.iter().enumerate() {
        cumulative += sale;
        running[i] = cumulative;
    }

    println!("Sales:   {:?}", sales);
    println!("Running: {:?}", running);
}
```

**Output:**

```
Sales:   [120, 340, 280, 510, 190, 430]
Running: [120, 460, 740, 1250, 1440, 1870]
```

**Trace:**

- `120` → 120
- `120 + 340` → 460
- `460 + 280` → 740
- `740 + 510` → 1250
- `1250 + 190` → 1440
- `1440 + 430` → 1870

---

### A26 — Tuple Struct Simulation

```rust
fn main() {
    // (id, name, gpa, grade)
    let students: [(u32, &str, f64, char); 4] = [
        (1001, "Alice",   3.92, 'A'),
        (1002, "Bob",     3.45, 'B'),
        (1003, "Charlie", 2.87, 'C'),
        (1004, "Diana",   3.98, 'A'),
    ];

    let mut highest_gpa: f64 = 0.0;

    for (id, name, gpa, grade) in students {
        println!("ID: {id} | Name: {:7} | GPA: {:.2} | Grade: {grade}", name, gpa);
        if gpa > highest_gpa {
            highest_gpa = gpa;
        }
    }

    println!("\nHighest GPA: {:.2}", highest_gpa);
}
```

**Output:**

```
ID: 1001 | Name: Alice   | GPA: 3.92 | Grade: A
ID: 1002 | Name: Bob     | GPA: 3.45 | Grade: B
ID: 1003 | Name: Charlie | GPA: 2.87 | Grade: C
ID: 1004 | Name: Diana   | GPA: 3.98 | Grade: A

Highest GPA: 3.98
```

---

## Section D — Deep Understanding

---

### A27 — Predict and Explain

**Output:**

```
Sum:          15
Doubled sum:  30
Evens: [2, 4]
Odds:  [1, 3, 5]
```

**Trace:**

- `arr = [1, 2, 3, 4, 5]`
- `sum`: `1+2+3+4+5 = 15`
- `doubled_sum`: `.map(|x| x * 2)` doubles each → `[2,4,6,8,10]`, then `.sum()` = `30`
- `evens`: `.filter(|&&x| x % 2 == 0)` → keeps `2` and `4` → `[2, 4]`
- `odds`: `.filter(|&&x| x % 2 != 0)` → keeps `1`, `3`, `5` → `[1, 3, 5]`

The `&&x` in the filter closure is because `.iter()` yields `&&i32` — the outer `&` is from the filter's closure argument, the inner `&` is from the iterator. Double-dereferencing with `&&x` gives us the raw `i32` value.

---

### A28 — True or False?

| #   | Statement                               | Answer    | Explanation                                                                        |
| --- | --------------------------------------- | --------- | ---------------------------------------------------------------------------------- |
| 1   | Tuple can hold different types          | **True**  | That's the defining feature of tuples                                              |
| 2   | Can iterate a tuple with `for`          | **False** | Elements may have different types; `for` can't be typed over them                  |
| 3   | `[i32; 3]` and `[i32; 4]` are same type | **False** | The length `N` is part of the type                                                 |
| 4   | `(42)` is a single-element tuple        | **False** | `(42)` is just parenthesized `i32`; need `(42,)` for a tuple                       |
| 5   | Array indices start at 1                | **False** | Zero-based indexing: first element is at index `0`                                 |
| 6   | `.get(i)` returns `Option<&T>`          | **True**  | Returns `Some(&elem)` if in bounds, `None` if not                                  |
| 7   | Can resize array after creation         | **False** | Arrays have a fixed size that's part of the type — use `Vec<T>` for dynamic sizing |
| 8   | `&arr[1..3]` gives indices 1 and 2      | **True**  | Rust ranges are exclusive on the right: `1..3` means indices 1 and 2               |
| 9   | `&[T]` stores pointer and length        | **True**  | A slice is a "fat pointer": memory address + element count                         |
| 10  | Can compare same-type tuples with `==`  | **True**  | If all element types implement `PartialEq`, the tuple does too                     |
| 11  | `let a: [i32; 0] = [];` is valid        | **True**  | Zero-length arrays are valid in Rust                                               |
| 12  | `arr[99]` causes undefined behavior     | **False** | Rust panics (safe crash) — no undefined behavior. This is a key safety guarantee   |

---

### A29 — Big Debug Challenge

**8 bugs identified and explained:**

```rust
fn main() {
    let student = ("Morgan", [85, 92, 78, 95, 88], true)
    //                                                   ^ BUG 1: missing semicolon

    println!("Student: {}", student[0]);
    //                              ^^^ BUG 2: tuples use dot notation, not [index]
    //                                  should be student.0

    println!("Scores: {:?}", student[1]);
    //                               ^^^ BUG 3: same issue — should be student.1

    let scores = student.1;
    let total: i32 = scores.iter().sum;
    //                             ^^^ BUG 4: sum is a METHOD call, needs ()
    //                                 should be .sum()

    let avg = total / scores.len();
    //                ^^^^^^^^^^^^^ BUG 5: type mismatch — total is i32, scores.len() is usize
    //                              must cast: total / scores.len() as i32

    println!("Average: {avg}");

    let best = scores.iter().max();
    println!("Best score: {best}");
    //                     ^^^^  BUG 6: .max() returns Option<&i32>, not i32
    //                           must unwrap: .max().unwrap() or use {:?}

    scores[2] = 82;
    //^^^^^^^^^^^^^BUG 7: scores is not declared mut
    //             BUG 8: scores is a COPY of student.1, not a reference to it
    //             Need: let mut scores = student.1;
    //             Also the tuple itself needs to be mutable for this to make sense
}
```

**Fully corrected program:**

```rust
fn main() {
    let student = ("Morgan", [85, 92, 78, 95, 88], true); // fix 1: semicolon

    println!("Student: {}", student.0);           // fix 2: dot notation
    println!("Scores: {:?}", student.1);          // fix 3: dot notation

    let mut scores = student.1;                   // fix 7+8: add mut
    let total: i32 = scores.iter().sum();         // fix 4: add () to sum
    let avg = total / scores.len() as i32;        // fix 5: cast len() to i32

    println!("Average: {avg}");

    let best = scores.iter().max().unwrap();       // fix 6: unwrap Option
    println!("Best score: {best}");

    scores[2] = 82;                               // now works: scores is mut
    println!("Updated scores: {:?}", scores);
}
```

**Output:**

```
Student: Morgan
Scores: [85, 92, 78, 95, 88]
Average: 87
Best score: 95
Updated scores: [85, 92, 82, 95, 88]
```

---

### A30 — Design Challenge: Pick the Right Structure

| #   | Scenario                             | Best Choice                          | Justification                                                                |
| --- | ------------------------------------ | ------------------------------------ | ---------------------------------------------------------------------------- |
| 1   | 3D vector (x, y, z as f64)           | **Array `[f64; 3]`**                 | Same type, fixed count, needs iteration for math operations                  |
| 2   | 7 days of the week                   | **Array `[&str; 7]`**                | Same type, fixed count, can be a `const`                                     |
| 3   | User profile (id, name, age, active) | **Tuple** (or struct)                | Mixed types, fixed fields — tuple works, struct is cleaner with named fields |
| 4   | List of student names (grows)        | **Vec\<&str\>**                      | Unknown/changing count — arrays can't grow                                   |
| 5   | RGB color (r, g, b as u8)            | **Tuple `(u8,u8,u8)`**               | Same type, could also be `[u8; 3]`; tuple is idiomatic for structured fields |
| 6   | Chess board 8×8                      | **2D Array `[[Square; 8]; 8]`**      | Fixed 8×8, same type, indexed by row/col                                     |
| 7   | Integer division result              | **Tuple `(i32, i32)`**               | Two related but conceptually different values (quotient vs remainder)        |
| 8   | Pixel data for 1920×1080             | **Vec\<u8\>** or **Vec\<RgbPixel\>** | ~6 million pixels — far too large for stack; must use heap (Vec)             |
| 9   | Top 10 high scores (always 10)       | **Array `[u32; 10]`**                | Fixed count, same type, stack allocation is fine                             |
| 10  | Stock prices for N days              | **Vec\<f64\>**                       | N is unknown at compile time — Vec grows dynamically                         |

---

## 🏆 Lesson Complete!

You've now mastered:

- ✅ Declaring, accessing, and destructuring tuples
- ✅ The unit type `()` and single-element tuples
- ✅ Mutable tuples and tuples as return values
- ✅ Arrays: declaration, access, mutation, initialization shorthand
- ✅ Iteration: `for..in`, `.iter()`, `.enumerate()`, `.iter_mut()`
- ✅ Safe vs unsafe array access (`.get()` vs `[i]`)
- ✅ Slices (`&[T]`) and how they enable flexible functions
- ✅ 2D arrays and multidimensional data
- ✅ Knowing when to use tuples vs arrays vs Vec

**Up next:** [Lesson 04 — Functions](../lesson_04_functions/lesson_04_functions.md) 🦀
