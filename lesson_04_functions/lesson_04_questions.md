# 🧪 Lesson 04 — Practice Questions: Functions

> **Instructions:** Attempt every question before checking answers.
> Answers are in [`lesson_04_answers.md`](./lesson_04_answers.md)

---

## Section A — Predict the Output

Read carefully and write the exact output before running the code.

---

### Q1 — Basic Function Call

```rust
fn say_hello(name: &str) {
    println!("Hello, {name}!");
}

fn main() {
    say_hello("Rustacean");
    say_hello("World");
    say_hello("Alice");
}
```

**What is the output?**

---

### Q2 — Return Value

```rust
fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

fn main() {
    let x = multiply(4, 5);
    let y = multiply(x, 2);
    let z = multiply(y, multiply(3, 3));
    println!("{x}");
    println!("{y}");
    println!("{z}");
}
```

**What are the three output lines?**

---

### Q3 — Statements vs Expressions

```rust
fn compute(n: i32) -> i32 {
    let doubled = n * 2;
    let result = {
        let extra = 10;
        doubled + extra
    };
    result
}

fn main() {
    println!("{}", compute(5));
    println!("{}", compute(0));
    println!("{}", compute(-3));
}
```

**What are the three output lines? Trace the block expression.**

---

### Q4 — Implicit vs Explicit Return

```rust
fn version_a(x: i32) -> i32 {
    x + 1
}

fn version_b(x: i32) -> i32 {
    return x + 1;
}

fn version_c(x: i32) -> i32 {
    let y = x + 1;
    y
}

fn main() {
    println!("{}", version_a(9));
    println!("{}", version_b(9));
    println!("{}", version_c(9));
}
```

**What is the output? What do all three functions have in common?**

---

### Q5 — Early Return

```rust
fn classify(n: i32) -> &'static str {
    if n < 0  { return "negative"; }
    if n == 0 { return "zero"; }
    if n > 100 { return "large"; }
    "normal"
}

fn main() {
    println!("{}", classify(-5));
    println!("{}", classify(0));
    println!("{}", classify(42));
    println!("{}", classify(200));
}
```

**What are the four output lines?**

---

### Q6 — Tuple Return

```rust
fn stats(a: i32, b: i32, c: i32) -> (i32, i32, i32) {
    let sum = a + b + c;
    let min = a.min(b).min(c);
    let max = a.max(b).max(c);
    (sum, min, max)
}

fn main() {
    let (s, lo, hi) = stats(10, 3, 7);
    println!("sum={s} min={lo} max={hi}");

    let result = stats(100, 200, 50);
    println!("{} {} {}", result.0, result.1, result.2);
}
```

**What are the two output lines?**

---

### Q7 — Function Pointers

```rust
fn add(a: i32, b: i32) -> i32 { a + b }
fn sub(a: i32, b: i32) -> i32 { a - b }
fn mul(a: i32, b: i32) -> i32 { a * b }

fn apply(f: fn(i32, i32) -> i32, x: i32, y: i32) -> i32 {
    f(x, y)
}

fn main() {
    println!("{}", apply(add, 10, 4));
    println!("{}", apply(sub, 10, 4));
    println!("{}", apply(mul, 10, 4));

    let op: fn(i32, i32) -> i32 = if true { add } else { sub };
    println!("{}", op(3, 3));
}
```

**What are the four output lines?**

---

### Q8 — Closures

```rust
fn main() {
    let base = 100;

    let add_base    = |x| x + base;
    let sub_base    = |x| x - base;
    let scale       = |x, factor| x * factor;

    println!("{}", add_base(50));
    println!("{}", sub_base(150));
    println!("{}", scale(7, 6));
    println!("{}", add_base(scale(3, 10)));
}
```

**What are the four output lines?**

---

### Q9 — Closures with Iterators

```rust
fn main() {
    let data = [1, 2, 3, 4, 5, 6, 7, 8];

    let sum: i32 = data.iter().sum();
    let evens: Vec<&i32> = data.iter().filter(|&&x| x % 2 == 0).collect();
    let squares: Vec<i32> = data.iter().map(|&x| x * x).collect();
    let big_squares: Vec<i32> = data.iter()
        .map(|&x| x * x)
        .filter(|&x| x > 20)
        .collect();

    println!("{sum}");
    println!("{:?}", evens);
    println!("{:?}", squares);
    println!("{:?}", big_squares);
}
```

**What are the four output lines?**

---

### Q10 — Nested Functions

```rust
fn outer(x: i32) -> i32 {
    fn double(n: i32) -> i32 {
        n * 2
    }

    fn add_one(n: i32) -> i32 {
        n + 1
    }

    add_one(double(x))
}

fn main() {
    println!("{}", outer(5));
    println!("{}", outer(0));
    println!("{}", outer(10));
}
```

**What are the three output lines?**

---

## Section B — Fix the Bugs

Each snippet has one or more bugs. Find every one, explain it, and write the corrected version.

---

### Q11 — Missing Return Type

```rust
fn add(a: i32, b: i32) {
    a + b
}

fn main() {
    println!("{}", add(3, 4));
}
```

---

### Q12 — Semicolon Kills the Return Value

```rust
fn square(n: i32) -> i32 {
    n * n;
}

fn main() {
    println!("{}", square(6));
}
```

---

### Q13 — Wrong Parameter Type

```rust
fn greet(times: f64) {
    for _ in 0..times {
        println!("Hello!");
    }
}

fn main() {
    greet(3.0);
}
```

---

### Q14 — Calling with Wrong Argument Count

```rust
fn describe(name: &str, age: u32) {
    println!("{name} is {age} years old");
}

fn main() {
    describe("Alice");
    describe("Bob", 25, "engineer");
}
```

---

### Q15 — Type Mismatch in Return

```rust
fn is_even(n: i32) -> bool {
    if n % 2 == 0 {
        return true;
    } else {
        return 0; // returning 0 instead of false
    }
}

fn main() {
    println!("{}", is_even(4));
    println!("{}", is_even(7));
}
```

---

### Q16 — Closure Type Inference Conflict

```rust
fn main() {
    let convert = |x| x * 2;

    println!("{}", convert(5));    // used as i32
    println!("{}", convert(3.14)); // used as f64 — conflict!
}
```

**Explain what happens and fix it properly (you may need two separate closures).**

---

### Q17 — Capturing Mutable Variable

```rust
fn main() {
    let mut total = 0;
    let add_to_total = |n| total += n; // closure captures total

    add_to_total(10);
    add_to_total(20);
    add_to_total(30);

    println!("Total: {total}");
}
```

**This code has a borrow issue. Explain what it is and fix it.**

---

### Q18 — Missing `mut` on Closure

```rust
fn main() {
    let mut count = 0;
    let increment = || {
        count += 1;
        count
    };

    println!("{}", increment());
    println!("{}", increment());
}
```

---

## Section C — Write It Yourself

---

### Q19 — Temperature Converter Functions

Write **three separate functions**:
- `celsius_to_fahrenheit(c: f64) -> f64`
- `fahrenheit_to_celsius(f: f64) -> f64`
- `celsius_to_kelvin(c: f64) -> f64`

Then in `main`:
- Test each with at least two values
- Print results with 2 decimal places and proper labels

---

### Q20 — Factorial

Write a function `factorial(n: u64) -> u64` that:
- Returns `1` if `n` is `0` or `1`
- Returns `n × factorial(n-1)` for any other `n`

Wait — Rust doesn't allow recursive calls without careful planning... actually it does! Write it recursively.

In `main`, compute and print `factorial` for `0` through `10`.

---

### Q21 — Fibonacci Sequence

Write a function `fibonacci(n: u32) -> u64` that returns the nth Fibonacci number (0-indexed):
- `fibonacci(0)` = 0
- `fibonacci(1)` = 1
- `fibonacci(n)` = `fibonacci(n-1)` + `fibonacci(n-2)`

In `main`, print the first 10 Fibonacci numbers (indices 0–9) on one line separated by spaces.

> **Bonus:** Also write an iterative version `fibonacci_iter(n: u32) -> u64` that uses a loop instead of recursion. Compare the outputs.

---

### Q22 — Caesar Cipher

Write a function `caesar_encrypt(text: &str, shift: u8) -> String` that:
- Shifts each ASCII uppercase letter by `shift` positions
- Wraps around: 'Z' shifted by 1 becomes 'A'
- Leaves non-uppercase characters unchanged

Write a companion `caesar_decrypt(text: &str, shift: u8) -> String` that reverses the process.

In `main`:
- Encrypt `"HELLO RUST"` with shift `3` → should give `"KHOOR UXVW"`
- Decrypt the result → should give back `"HELLO RUST"`

---

### Q23 — Array Statistics with Functions

Write these four functions, each taking `data: &[f64]`:
- `mean(data: &[f64]) -> f64`
- `variance(data: &[f64]) -> f64` — average of squared differences from mean
- `std_deviation(data: &[f64]) -> f64` — square root of variance
- `median(data: &[f64]) -> f64` — middle value when sorted (average middle two if even length)

In `main`, test with: `[4.0, 7.0, 13.0, 2.0, 1.0, 9.0, 6.0, 5.0]`

> **Hint for median:** You'll need to sort a copy of the slice. Use `let mut sorted = data.to_vec(); sorted.sort_by(|a, b| a.partial_cmp(b).unwrap());`

---

### Q24 — Higher-Order Functions

Write a function:
```rust
fn transform_all(data: &[i32], f: impl Fn(i32) -> i32) -> Vec<i32>
```
That applies `f` to every element of `data` and returns a new Vec.

Then in `main`, use it with:
1. A closure that doubles every element
2. A closure that squares every element
3. A named function that negates every element
4. A closure that adds a captured value `offset = 100` to every element

Print all four results.

---

### Q25 — Closure Counter Factory

Write a function `make_counter(start: i32, step: i32) -> impl FnMut() -> i32` that:
- Returns a closure
- Each time the closure is called, it returns the next value (starting at `start`, incrementing by `step`)

In `main`:
```rust
let mut counter = make_counter(0, 1);
println!("{}", counter()); // 0
println!("{}", counter()); // 1
println!("{}", counter()); // 2

let mut by_fives = make_counter(10, 5);
println!("{}", by_fives()); // 10
println!("{}", by_fives()); // 15
println!("{}", by_fives()); // 20
```

---

### Q26 — FizzBuzz with Functions

Refactor FizzBuzz into clean, testable functions:

- `fizzbuzz_label(n: u32) -> &'static str` — returns `"FizzBuzz"`, `"Fizz"`, `"Buzz"`, or... wait, you can't return the number as `&'static str`. Return `"Number"` for non-FizzBuzz numbers.
- `is_fizz(n: u32) -> bool`
- `is_buzz(n: u32) -> bool`
- `is_fizzbuzz(n: u32) -> bool`

In `main`:
- Print the FizzBuzz label for numbers 1–20
- But for "Number" cases, actually print the number itself

Expected output for the first 15:
```
1
2
Fizz
4
Buzz
Fizz
7
8
Fizz
Buzz
11
Fizz
13
14
FizzBuzz
```

---

### Q27 — Word Utilities

Write these three string utility functions using closures and iterators:

```rust
fn count_vowels(s: &str) -> usize
fn reverse_words(s: &str) -> String
fn capitalize_words(s: &str) -> String
```

- `count_vowels`: counts 'a', 'e', 'i', 'o', 'u' (case-insensitive)
- `reverse_words`: reverses the order of words, e.g., `"hello world"` → `"world hello"`
- `capitalize_words`: capitalizes first letter of each word, e.g., `"hello world"` → `"Hello World"`

In `main`, test with: `"the quick brown fox jumps over the lazy dog"`

---

### Q28 — Grade Calculator

Write a complete grade calculation system using multiple functions:

```rust
fn letter_grade(score: f64) -> char
fn grade_points(letter: char) -> f64
fn gpa(scores: &[f64]) -> f64
fn pass_fail(score: f64) -> &'static str
```

- `letter_grade`: `90+` → `'A'`, `80+` → `'B'`, `70+` → `'C'`, `60+` → `'D'`, else `'F'`
- `grade_points`: `'A'`→4.0, `'B'`→3.0, `'C'`→2.0, `'D'`→1.0, `'F'`→0.0
- `gpa`: average grade points for all scores
- `pass_fail`: `60+` → `"Pass"`, else `"Fail"`

In `main`, use these scores: `[88.5, 72.0, 95.5, 61.0, 54.0, 80.5]`

Print: each score with its letter grade, pass/fail, and the final GPA.

---

## Section D — Deep Understanding

---

### Q29 — Predict the Complex Output

Trace this carefully step by step before running it.

```rust
fn mystery(x: i32) -> i32 {
    let a = {
        let b = x * 2;
        b + 3        // block expression value
    };

    let c = if a > 10 { a * 2 } else { a + 100 };

    c - x
}

fn main() {
    println!("{}", mystery(1));
    println!("{}", mystery(5));
    println!("{}", mystery(10));
}
```

**Trace each call showing every intermediate value.**

---

### Q30 — True or False?

For each statement, say True or False and briefly justify.

1. Function parameter types can be inferred by the compiler.
2. A function can return multiple values using a tuple.
3. Adding a semicolon after the last expression in a function returns `()`.
4. Closures cannot modify variables from the enclosing scope.
5. A function defined later in the file cannot be called by `main`.
6. `return` at the end of a function (not an early return) is idiomatic Rust.
7. A closure that captures a variable by immutable reference implements `Fn`.
8. Nested functions can access variables from their enclosing function's scope.
9. A diverging function with return type `!` must contain a `loop` or `panic!`.
10. `impl Fn(i32) -> i32` and `fn(i32) -> i32` are interchangeable in all contexts.

---

### Q31 — Refactor the Monster Function

The function below does too many things at once. Refactor it into at least **four separate, well-named functions** while keeping the exact same behaviour.

```rust
fn process(students: &[(&str, f64)]) {
    let mut total = 0.0;
    let mut count = 0;
    let mut highest = f64::NEG_INFINITY;
    let mut highest_name = "";

    for &(name, score) in students {
        println!("--- {} ---", name);

        let grade = if score >= 90.0 { "A" }
            else if score >= 80.0 { "B" }
            else if score >= 70.0 { "C" }
            else if score >= 60.0 { "D" }
            else { "F" };

        let status = if score >= 60.0 { "PASS" } else { "FAIL" };

        println!("Score: {:.1}", score);
        println!("Grade: {} ({})", grade, status);

        total += score;
        count += 1;

        if score > highest {
            highest = score;
            highest_name = name;
        }
    }

    let average = total / count as f64;
    println!("=== Summary ===");
    println!("Average: {:.2}", average);
    println!("Top student: {} ({:.1})", highest_name, highest);
}

fn main() {
    let students = [
        ("Alice", 92.5),
        ("Bob", 74.0),
        ("Charlie", 58.5),
        ("Diana", 88.0),
    ];
    process(&students);
}
```

**Requirements for your refactor:**
- At least 4 separate functions
- `main` should read clearly like plain English
- Keep the exact same output

---

### Q32 — The Big Debug Challenge

The program below has **9 bugs**. Find every one, explain it, and provide the fully corrected program.

```rust
fn greet(name: &str, times i32) {
    for _ in 0..times {
        println!("Hello, {name}");
    }
}

fn add_and_square(a: i32, b: i32) -> i32 {
    let sum = a + b;
    sum * sum;
}

fn describe_number(n: i32) -> &str {
    if n > 0 { "positive" }
    else if n < 0 { "negative" }
    else { "zero" }
}

fn apply_twice(f: fn(i32) -> i32, x i32) -> i32 {
    f(f(x))
}

fn double(x: i32) -> i32 { x * 2 }

fn main() {
    greet("Alice", 2);

    println!("{}", add_and_square(3, 4));

    println!("{}", describe_number(-5));
    println!("{}", describe_number(0));

    println!("{}", apply_twice(double 5));

    let square = |x: i32| -> i32 { x * x; };
    println!("{}", square(4));
}
```

**List every bug with a clear explanation, then show the corrected program.**

---

*"A well-named function is worth a thousand comments." 🦀*
