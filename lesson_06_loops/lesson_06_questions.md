# 🧪 Lesson 06 — Questions: Loops

> Answers in [`lesson_06_answers.md`](./lesson_06_answers.md)

---

## Section A — Predict the Output

### Q1 — loop with break
```rust
fn main() {
    let mut x = 1;
    let result = loop {
        x *= 2;
        if x > 50 { break x; }
    };
    println!("{result}");
}
```

### Q2 — while with counter
```rust
fn main() {
    let mut i = 10;
    while i > 0 {
        if i % 3 == 0 { println!("{i}"); }
        i -= 2;
    }
}
```

### Q3 — for with range
```rust
fn main() {
    let mut sum = 0;
    for n in 1..=5 {
        sum += n;
        println!("n={n} sum={sum}");
    }
}
```

### Q4 — continue
```rust
fn main() {
    for i in 0..8 {
        if i % 2 == 0 { continue; }
        if i > 5      { break; }
        print!("{i} ");
    }
    println!();
}
```

### Q5 — enumerate
```rust
fn main() {
    let words = ["zero", "one", "two", "three"];
    for (i, w) in words.iter().enumerate() {
        if i % 2 == 0 { println!("{i}:{w}"); }
    }
}
```

### Q6 — while let
```rust
fn main() {
    let mut v = vec![10, 20, 30];
    let mut total = 0;
    while let Some(n) = v.pop() {
        total += n;
        println!("total={total}");
    }
}
```

### Q7 — nested loops with label
```rust
fn main() {
    'outer: for i in 0..4 {
        for j in 0..4 {
            if i + j == 4 {
                println!("hit: i={i} j={j}");
                break 'outer;
            }
        }
    }
    println!("done");
}
```

### Q8 — iterator chain
```rust
fn main() {
    let data = vec![1, 2, 3, 4, 5, 6];
    let result: Vec<i32> = data.iter()
        .filter(|&&x| x % 2 != 0)
        .map(|&x| x * x)
        .collect();
    println!("{result:?}");

    let s: i32 = data.iter()
        .filter(|&&x| x > 3)
        .sum();
    println!("{s}");
}
```

---

## Section B — Fix the Bugs

### Q9
```rust
fn main() {
    let result = loop {
        break;       // should return 42
    };
    println!("{result}");
}
```

### Q10
```rust
fn main() {
    let mut i = 0;
    while i < 5 {
        println!("{i}");
    }
}
```

### Q11
```rust
fn main() {
    for i in 5..0 {    // should count 5 down to 1
        println!("{i}");
    }
}
```

### Q12
```rust
fn main() {
    let v = vec![1, 2, 3];
    for x in v.iter_mut() {
        println!("{x}");
    }
    // must still be able to use v here
    println!("{v:?}");
}
```

### Q13 — Logic bug
```rust
fn main() {
    let mut count = 0;
    for i in 1..=100 {
        if i % 2 == 0 { continue; }
        if i % 5 == 0 { continue; }
        count += 1;
    }
    // How many odd non-multiples-of-5 are in 1..=100?
    // The programmer expected 40, but got a different number.
    // Is the code correct? What does count actually equal?
    println!("{count}");
}
```

### Q14
```rust
fn main() {
    let data = vec![3, 6, 2, 8, 1];
    let max: i32 = data.iter().max();
    println!("{max}");
}
```

---

## Section C — Write It Yourself

### Q15 — FizzBuzz with loop
Using only a `loop` (no `for` or `while`), print FizzBuzz for 1 through 20.

### Q16 — Collatz sequence
Write `fn collatz_length(n: u64) -> u64` that returns the number of steps in the Collatz sequence starting at `n` until reaching 1.
- If n is even: `n = n / 2`
- If n is odd: `n = 3*n + 1`

In `main`, find and print the number in the range `1..=100` with the **longest** Collatz sequence.

### Q17 — Prime sieve
Write a function `primes_up_to(limit: u64) -> Vec<u64>` that returns all prime numbers up to `limit` using a simple sieve or trial-division approach. Print all primes up to 50.

### Q18 — Running statistics
Write a function `running_stats(data: &[f64]) -> Vec<(f64, f64, f64)>` that returns a Vec of `(current_value, running_sum, running_average)` tuples for each element. In `main`, print a formatted table.

### Q19 — Iterator pipeline
Given `data = [5, 12, 3, 18, 7, 22, 1, 15, 9, 30]`, using **only iterator adaptors** (no explicit loops):
1. Filter values greater than 10
2. Multiply each by 3
3. Collect into a Vec and print
4. Also compute: count, sum, max, and min of the filtered-and-multiplied result

### Q20 — Word frequency (sorted)
Write a function that takes a sentence `&str`, splits it into words, and using loops prints each **unique word** alongside the number of times it appears, sorted alphabetically. Use a `Vec<(String, usize)>` to store counts.

Test with: `"the cat sat on the mat the cat sat"`

### Q21 — Matrix spiral
Write a program that, given a 4×4 matrix filled with 1–16, prints the elements in **spiral order** (outer ring clockwise, then inner):
```
 1  2  3  4
 5  6  7  8
 9 10 11 12
13 14 15 16
```
Expected spiral: `1 2 3 4 8 12 16 15 14 13 9 5 6 7 11 10`

### Q22 — Retry loop
Write a function `fn attempt_login(passwords: &[&str], correct: &str) -> (bool, usize)` that:
- Tries each password in order using a `loop`
- Stops and returns `(true, attempt_number)` on success
- Returns `(false, total_attempts)` if all fail

---

## Section D — Deep Understanding

### Q23 — Predict complex output
```rust
fn main() {
    let mut count = 0;
    'a: for i in 0..5 {
        for j in 0..5 {
            if j >= i { continue 'a; }
            count += 1;
        }
    }
    println!("{count}");
}
```
Trace step-by-step and explain why.

### Q24 — True or False?
1. `for` loops in Rust can return a value using `break value`.
2. `while let` stops when the pattern stops matching.
3. `.map()` immediately processes all elements when called.
4. `break 'outer` in an inner loop exits only the inner loop.
5. `.into_iter()` on a `Vec` consumes the `Vec`.
6. `for i in 5..0` iterates backwards from 5 to 1.
7. `continue 'label` skips to the next iteration of the labeled loop.
8. `loop` is the only Rust loop that can return a non-unit value.

### Q25 — Big Debug Challenge (8 bugs)
```rust
fn main() {
    // Bug hunt!

    // Count down from 10 to 1
    let mut n = 10;
    loop {
        println!("{n}");
        n -= 1;
    }

    // Sum of squares of even numbers 1..=20
    let sum: i32 = (1..20)
        .filter(|x| x % 2 == 0)
        .map(|x| x ^ 2)         // intend: square
        .sum();
    println!("Sum of even squares: {sum}");

    // Find first element > 100
    let data = vec![10, 50, 80, 120, 200];
    let found = data.iter().find(|x| x > 100);
    println!("{found:?}");

    // Print every 3rd element
    for (i, val) in data.iter().enumerate() {
        if i % 3 = 0 { println!("{val}"); }
    }
}
```
Find all 8 bugs, explain each, and write the corrected program.

### Q26 — Design: which loop?
For each scenario explain which loop construct is best and why, then write a small example:
1. Read lines from input until the user types "quit" (simulate with a Vec of strings)
2. Process all items in a shopping cart Vec<f64>
3. Find the first Fibonacci number greater than 1,000
4. Retry a network request up to 5 times, returning the attempt count on success
5. Print a triangular pattern of asterisks for n rows
