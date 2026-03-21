# ✅ Lesson 04 — Answers: Functions

> Full solutions with explanations.
> Questions are in [`lesson_04_questions.md`](./lesson_04_questions.md)

---

## Section A — Predict the Output

---

### A1 — Basic Function Call

**Output:**

```
Hello, Rustacean!
Hello, World!
Hello, Alice!
```

**Explanation:** The function is called three times with three different string arguments. Each call produces one line of output using the `{name}` format placeholder. The function has no return value — its only job is the side effect of printing.

---

### A2 — Return Value

**Output:**

```
20
40
360
```

**Trace:**

- `x = multiply(4, 5)` → `4 × 5 = 20`
- `y = multiply(x, 2)` → `multiply(20, 2)` → `20 × 2 = 40`
- `z = multiply(y, multiply(3, 3))` → `multiply(40, 9)` → `40 × 9 = 360`

Notice how function calls can be **nested** — the inner call `multiply(3, 3)` is evaluated first and its result (`9`) is used as the argument to the outer call.

---

### A3 — Statements vs Expressions

**Output:**

```
20
10
4
```

**Trace for `compute(5)`:**

1. `let doubled = 5 * 2` → `doubled = 10`
2. The block `{ let extra = 10; doubled + extra }` evaluates to `10 + 10 = 20`
3. `let result = 20`
4. `result` is the last expression → returned as `20`

**Trace for `compute(0)`:**

1. `doubled = 0`
2. Block: `0 + 10 = 10`
3. Returns `10`

**Trace for `compute(-3)`:**

1. `doubled = -6`
2. Block: `-6 + 10 = 4`
3. Returns `4`

---

### A4 — Implicit vs Explicit Return

**Output:**

```
10
10
10
```

All three functions produce identical output — `9 + 1 = 10` — they are functionally equivalent. The difference is stylistic:

- `version_a` uses an **implicit return** (last expression, no semicolon) — idiomatic Rust
- `version_b` uses an **explicit `return`** keyword — valid but non-idiomatic at the end of a function
- `version_c` assigns to a variable first, then returns it — also valid but more verbose

**Takeaway:** Prefer `version_a` style in idiomatic Rust.

---

### A5 — Early Return

**Output:**

```
negative
zero
normal
large
```

**Trace:** Each call hits the first matching `return` and exits immediately:

- `-5 < 0` → returns `"negative"` on line 1
- `0 == 0` → passes the first guard, hits `return "zero"`
- `42` → not negative, not zero, not > 100 → falls through all guards → `"normal"`
- `200 > 100` → returns `"large"` on line 3

This **guard clause** pattern is idiomatic Rust — handle special cases first, leave the happy path at the end without indentation.

---

### A6 — Tuple Return

**Output:**

```
sum=20 min=3 max=10
350 50 200
```

**Trace:**

- `stats(10, 3, 7)`: sum=20, min=3, max=10 → destructured into `(s, lo, hi)`
- `stats(100, 200, 50)`: sum=350, min=50, max=200 → accessed as `.0`, `.1`, `.2`

---

### A7 — Function Pointers

**Output:**

```
14
6
40
6
```

**Trace:**

- `apply(add, 10, 4)` → `add(10, 4)` → `14`
- `apply(sub, 10, 4)` → `sub(10, 4)` → `6`
- `apply(mul, 10, 4)` → `mul(10, 4)` → `40`
- `let op = if true { add } else { sub }` → `op = add` → `op(3, 3)` → `3 + 3 = 6`

---

### A8 — Closures

**Output:**

```
150
50
42
130
```

**Trace (`base = 100`):**

- `add_base(50)` → `50 + 100 = 150`
- `sub_base(150)` → `150 - 100 = 50`
- `scale(7, 6)` → `7 * 6 = 42`
- `add_base(scale(3, 10))` → `add_base(30)` → `30 + 100 = 130`

Both `add_base` and `sub_base` **capture** `base = 100` from the enclosing scope.

---

### A9 — Closures with Iterators

**Output:**

```
36
[2, 4, 6, 8]
[1, 4, 9, 16, 25, 36, 49, 64]
[25, 36, 49, 64]
```

**Trace (`data = [1, 2, 3, 4, 5, 6, 7, 8]`):**

- `sum`: `1+2+3+4+5+6+7+8 = 36`
- `evens`: filter where `x % 2 == 0` → `[2, 4, 6, 8]`
- `squares`: map each `x` to `x*x` → `[1, 4, 9, 16, 25, 36, 49, 64]`
- `big_squares`: squares then filter `> 20` → `[25, 36, 49, 64]` (dropping 1, 4, 9, 16)

---

### A10 — Nested Functions

**Output:**

```
11
1
21
```

**Trace:**

- `outer(5)`: `double(5)=10`, `add_one(10)=11`
- `outer(0)`: `double(0)=0`, `add_one(0)=1`
- `outer(10)`: `double(10)=20`, `add_one(20)=21`

`double` and `add_one` are nested inside `outer` — they are invisible outside that function. Like regular functions (not closures), they do NOT capture the enclosing scope; they work only with their explicit parameters.

---

## Section B — Fix the Bugs

---

### A11 — Missing Return Type

**Bug:** The function computes `a + b` but there is no `-> i32` return type declared, so Rust treats the function as returning `()`. The expression `a + b` then becomes a type error because the function body produces `i32` but the implicit return type is `()`.

```rust
// ❌ Original
fn add(a: i32, b: i32) {  // missing -> i32
    a + b
}

// ✅ Fixed
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn main() {
    println!("{}", add(3, 4)); // 7
}
```

---

### A12 — Semicolon Kills the Return Value

**Bug:** `n * n;` ends with a semicolon, turning the expression into a statement — the computed value is **discarded**. The function body now returns `()`, but the declared return type is `i32`. This is a compile error.

```rust
// ❌ Original
fn square(n: i32) -> i32 {
    n * n;  // semicolon discards the value!
}

// ✅ Fixed — remove the semicolon
fn square(n: i32) -> i32 {
    n * n   // no semicolon = this is the return value
}

fn main() {
    println!("{}", square(6)); // 36
}
```

The compiler even hints at this: _"help: remove this semicolon to return this value"_.

---

### A13 — Wrong Parameter Type

**Bug:** The `for` loop range `0..times` requires `times` to be an integer type (`i32`, `u32`, `usize`, etc.). `f64` cannot be used in a range expression.

```rust
// ❌ Original
fn greet(times: f64) {
    for _ in 0..times { // error: `f64` is not an integer
        println!("Hello!");
    }
}

// ✅ Fixed — use an integer type
fn greet(times: u32) {
    for _ in 0..times {
        println!("Hello!");
    }
}

fn main() {
    greet(3); // Hello! x3
}
```

---

### A14 — Calling with Wrong Argument Count

**Two bugs:**

1. `describe("Alice")` — missing the required `age` argument
2. `describe("Bob", 25, "engineer")` — has an extra argument that the function doesn't accept

```rust
// ❌ Original
fn main() {
    describe("Alice");            // missing age
    describe("Bob", 25, "engineer"); // extra argument
}

// ✅ Fixed
fn describe(name: &str, age: u32) {
    println!("{name} is {age} years old");
}

fn main() {
    describe("Alice", 30); // provide both required arguments
    describe("Bob", 25);   // only two arguments
}
```

---

### A15 — Type Mismatch in Return

**Bug:** The function declares `-> bool` but the `else` branch returns `0` (an integer), not `false` (a boolean). Rust doesn't coerce between integer and bool.

```rust
// ❌ Original
fn is_even(n: i32) -> bool {
    if n % 2 == 0 {
        return true;
    } else {
        return 0; // error: expected `bool`, found integer
    }
}

// ✅ Fixed — use false
fn is_even(n: i32) -> bool {
    if n % 2 == 0 { true } else { false }
    // Or even more idiomatically:
    // n % 2 == 0
}

fn main() {
    println!("{}", is_even(4)); // true
    println!("{}", is_even(7)); // false
}
```

**Most idiomatic version:**

```rust
fn is_even(n: i32) -> bool {
    n % 2 == 0  // comparison already produces a bool
}
```

---

### A16 — Closure Type Inference Conflict

**Bug:** Once Rust infers the type of a closure's parameter from its first use (`convert(5)` → inferred as `i32`), it cannot be called with a different type (`3.14` as `f64`). A closure's type is fixed after its first use.

```rust
// ❌ Original — type conflict
let convert = |x| x * 2;
println!("{}", convert(5));    // infers i32
println!("{}", convert(3.14)); // error: expected i32, found f64

// ✅ Fix — use two separate closures, each with an explicit type
fn main() {
    let convert_int   = |x: i32| x * 2;
    let convert_float = |x: f64| x * 2.0;

    println!("{}", convert_int(5));    // 10
    println!("{}", convert_float(3.14)); // 6.28
}
```

---

### A17 — Capturing Mutable Variable

**Bug:** When `add_to_total` is a closure that mutably borrows `total`, Rust won't allow you to use `total` again while the closure exists (the closure holds a mutable borrow). Printing `total` at the end while the closure is still in scope violates the borrow rules.

```rust
// ❌ Original — borrow conflict
let mut total = 0;
let add_to_total = |n| total += n;
add_to_total(10);
add_to_total(20);
add_to_total(30);
println!("Total: {total}"); // total is still mutably borrowed by the closure

// ✅ Fix — drop the closure before using total again
fn main() {
    let mut total = 0;
    {
        let mut add_to_total = |n| total += n; // mut needed on closure too
        add_to_total(10);
        add_to_total(20);
        add_to_total(30);
    } // closure dropped here, releasing the mutable borrow

    println!("Total: {total}"); // 60 — now accessible
}
```

---

### A18 — Missing `mut` on Closure

**Bug:** A closure that mutates a captured variable (like `count += 1`) must itself be declared `mut`. This is because `FnMut` closures modify their captured environment and need to be mutable.

```rust
// ❌ Original
let increment = || { ... }; // missing mut

// ✅ Fixed
fn main() {
    let mut count = 0;
    let mut increment = || {  // add mut here
        count += 1;
        count
    };

    println!("{}", increment()); // 1
    println!("{}", increment()); // 2
}
```

---

## Section C — Write It Yourself

---

### A19 — Temperature Converter Functions

```rust
fn celsius_to_fahrenheit(c: f64) -> f64 {
    c * 9.0 / 5.0 + 32.0
}

fn fahrenheit_to_celsius(f: f64) -> f64 {
    (f - 32.0) * 5.0 / 9.0
}

fn celsius_to_kelvin(c: f64) -> f64 {
    c + 273.15
}

fn main() {
    println!("=== Celsius → Fahrenheit ===");
    println!("{:.2}°C = {:.2}°F", 0.0,   celsius_to_fahrenheit(0.0));
    println!("{:.2}°C = {:.2}°F", 100.0, celsius_to_fahrenheit(100.0));
    println!("{:.2}°C = {:.2}°F", -40.0, celsius_to_fahrenheit(-40.0));

    println!("\n=== Fahrenheit → Celsius ===");
    println!("{:.2}°F = {:.2}°C", 32.0,  fahrenheit_to_celsius(32.0));
    println!("{:.2}°F = {:.2}°C", 212.0, fahrenheit_to_celsius(212.0));

    println!("\n=== Celsius → Kelvin ===");
    println!("{:.2}°C = {:.2}K", 0.0,    celsius_to_kelvin(0.0));
    println!("{:.2}°C = {:.2}K", 100.0,  celsius_to_kelvin(100.0));
    println!("{:.2}°C = {:.2}K", -273.15, celsius_to_kelvin(-273.15));
}
```

**Output:**

```
=== Celsius → Fahrenheit ===
0.00°C = 32.00°F
100.00°C = 212.00°F
-40.00°C = -40.00°F

=== Fahrenheit → Celsius ===
32.00°F = 0.00°C
212.00°F = 100.00°C

=== Celsius → Kelvin ===
0.00°C = 273.15K
100.00°C = 373.15K
-273.15°C = 0.00K
```

---

### A20 — Factorial

```rust
fn factorial(n: u64) -> u64 {
    if n <= 1 {
        1
    } else {
        n * factorial(n - 1)
    }
}

fn main() {
    for i in 0..=10 {
        println!("{}! = {}", i, factorial(i));
    }
}
```

**Output:**

```
0! = 1
1! = 1
2! = 2
3! = 6
4! = 24
5! = 120
6! = 720
7! = 5040
8! = 40320
9! = 362880
10! = 3628800
```

**How recursion works for `factorial(5)`:**

```
factorial(5)
= 5 * factorial(4)
= 5 * (4 * factorial(3))
= 5 * (4 * (3 * factorial(2)))
= 5 * (4 * (3 * (2 * factorial(1))))
= 5 * (4 * (3 * (2 * 1)))
= 5 * (4 * (3 * 2))
= 5 * (4 * 6)
= 5 * 24
= 120
```

---

### A21 — Fibonacci Sequence

```rust
// Recursive version
fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

// Iterative version (bonus — much more efficient!)
fn fibonacci_iter(n: u32) -> u64 {
    if n == 0 { return 0; }
    if n == 1 { return 1; }

    let mut prev = 0u64;
    let mut curr = 1u64;

    for _ in 2..=n {
        let next = prev + curr;
        prev = curr;
        curr = next;
    }
    curr
}

fn main() {
    print!("Recursive: ");
    for i in 0..10 {
        print!("{} ", fibonacci(i));
    }
    println!();

    print!("Iterative: ");
    for i in 0..10 {
        print!("{} ", fibonacci_iter(i));
    }
    println!();
}
```

**Output:**

```
Recursive: 0 1 1 2 3 5 8 13 21 34
Iterative: 0 1 1 2 3 5 8 13 21 34
```

> **Performance note:** The recursive version has exponential time complexity O(2ⁿ) — it recomputes the same values many times. The iterative version is O(n). For large n, always use the iterative approach.

---

### A22 — Caesar Cipher

```rust
fn caesar_encrypt(text: &str, shift: u8) -> String {
    text.chars().map(|c| {
        if c.is_ascii_uppercase() {
            let shifted = ((c as u8 - b'A' + shift) % 26 + b'A') as char;
            shifted
        } else {
            c
        }
    }).collect()
}

fn caesar_decrypt(text: &str, shift: u8) -> String {
    // Decrypting is encrypting with the inverse shift
    let inverse_shift = 26 - (shift % 26);
    caesar_encrypt(text, inverse_shift)
}

fn main() {
    let original  = "HELLO RUST";
    let encrypted = caesar_encrypt(original, 3);
    let decrypted = caesar_decrypt(&encrypted, 3);

    println!("Original:  {original}");   // HELLO RUST
    println!("Encrypted: {encrypted}");  // KHOOR UXVW
    println!("Decrypted: {decrypted}");  // HELLO RUST

    // Verify round-trip
    assert_eq!(original, decrypted, "Round-trip failed!");
    println!("Round-trip verified ✓");
}
```

**Output:**

```
Original:  HELLO RUST
Encrypted: KHOOR UXVW
Decrypted: HELLO RUST
Round-trip verified ✓
```

**How the shift works for 'H' with shift 3:**

- `'H' as u8 = 72`
- `72 - 65 ('A') = 7` (0-based index of H)
- `7 + 3 = 10`
- `10 % 26 = 10` (K is index 10)
- `10 + 65 = 75` → `'K'`

---

### A23 — Array Statistics with Functions

```rust
fn mean(data: &[f64]) -> f64 {
    data.iter().sum::<f64>() / data.len() as f64
}

fn variance(data: &[f64]) -> f64 {
    let m = mean(data);
    data.iter().map(|&x| (x - m).powi(2)).sum::<f64>() / data.len() as f64
}

fn std_deviation(data: &[f64]) -> f64 {
    variance(data).sqrt()
}

fn median(data: &[f64]) -> f64 {
    let mut sorted = data.to_vec();
    sorted.sort_by(|a, b| a.partial_cmp(b).unwrap());
    let len = sorted.len();
    if len % 2 == 0 {
        (sorted[len / 2 - 1] + sorted[len / 2]) / 2.0
    } else {
        sorted[len / 2]
    }
}

fn main() {
    let data = [4.0, 7.0, 13.0, 2.0, 1.0, 9.0, 6.0, 5.0];

    println!("Data:          {:?}", data);
    println!("Mean:          {:.4}", mean(&data));
    println!("Variance:      {:.4}", variance(&data));
    println!("Std Deviation: {:.4}", std_deviation(&data));
    println!("Median:        {:.4}", median(&data));
}
```

**Output:**

```
Data:          [4.0, 7.0, 13.0, 2.0, 1.0, 9.0, 6.0, 5.0]
Mean:          5.8750
Variance:      13.1094
Std Deviation: 3.6207
Median:        5.5000
```

**Median trace:** Sorted: `[1, 2, 4, 5, 6, 7, 9, 13]`. Even length (8), so median = `(5 + 6) / 2 = 5.5`.

---

### A24 — Higher-Order Functions

```rust
fn transform_all(data: &[i32], f: impl Fn(i32) -> i32) -> Vec<i32> {
    data.iter().map(|&x| f(x)).collect()
}

fn negate(x: i32) -> i32 { -x }

fn main() {
    let data = [1, 2, 3, 4, 5];
    let offset = 100;

    let doubled  = transform_all(&data, |x| x * 2);
    let squared  = transform_all(&data, |x| x * x);
    let negated  = transform_all(&data, negate);
    let shifted  = transform_all(&data, |x| x + offset);

    println!("Original: {:?}", data);
    println!("Doubled:  {:?}", doubled);
    println!("Squared:  {:?}", squared);
    println!("Negated:  {:?}", negated);
    println!("Shifted:  {:?}", shifted);
}
```

**Output:**

```
Original: [1, 2, 3, 4, 5]
Doubled:  [2, 4, 6, 8, 10]
Squared:  [1, 4, 9, 16, 25]
Negated:  [-1, -2, -3, -4, -5]
Shifted:  [101, 102, 103, 104, 105]
```

---

### A25 — Closure Counter Factory

```rust
fn make_counter(start: i32, step: i32) -> impl FnMut() -> i32 {
    let mut current = start;
    move || {           // `move` takes ownership of `current` and `step`
        let value = current;
        current += step;
        value
    }
}

fn main() {
    let mut counter = make_counter(0, 1);
    println!("{}", counter()); // 0
    println!("{}", counter()); // 1
    println!("{}", counter()); // 2

    let mut by_fives = make_counter(10, 5);
    println!("{}", by_fives()); // 10
    println!("{}", by_fives()); // 15
    println!("{}", by_fives()); // 20
}
```

**Output:**

```
0
1
2
10
15
20
```

**Key concept:** `move` transfers ownership of captured variables into the closure. This means the closure owns its own copy of `current` and `step` — the two counters are completely independent. `impl FnMut()` signals that the returned closure modifies its captured state.

---

### A26 — FizzBuzz with Functions

```rust
fn is_fizz(n: u32) -> bool { n % 3 == 0 }
fn is_buzz(n: u32) -> bool { n % 5 == 0 }
fn is_fizzbuzz(n: u32) -> bool { n % 15 == 0 }

fn fizzbuzz_label(n: u32) -> &'static str {
    if is_fizzbuzz(n) { "FizzBuzz" }
    else if is_fizz(n) { "Fizz" }
    else if is_buzz(n) { "Buzz" }
    else { "Number" }
}

fn main() {
    for n in 1..=20 {
        let label = fizzbuzz_label(n);
        if label == "Number" {
            println!("{n}");
        } else {
            println!("{label}");
        }
    }
}
```

**Output:**

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
16
17
Fizz
19
Buzz
```

---

### A27 — Word Utilities

```rust
fn count_vowels(s: &str) -> usize {
    s.chars()
     .filter(|c| "aeiouAEIOU".contains(*c))
     .count()
}

fn reverse_words(s: &str) -> String {
    s.split_whitespace()
     .rev()
     .collect::<Vec<&str>>()
     .join(" ")
}

fn capitalize_words(s: &str) -> String {
    s.split_whitespace()
     .map(|word| {
         let mut chars = word.chars();
         match chars.next() {
             None    => String::new(),
             Some(c) => c.to_uppercase().to_string() + chars.as_str(),
         }
     })
     .collect::<Vec<String>>()
     .join(" ")
}

fn main() {
    let text = "the quick brown fox jumps over the lazy dog";

    println!("Original:         {text}");
    println!("Vowel count:      {}", count_vowels(text));
    println!("Reversed words:   {}", reverse_words(text));
    println!("Capitalized:      {}", capitalize_words(text));
}
```

**Output:**

```
Original:         the quick brown fox jumps over the lazy dog
Vowel count:      11
Reversed words:   dog lazy the over jumps fox brown quick the
Capitalized:      The Quick Brown Fox Jumps Over The Lazy Dog
```

---

### A28 — Grade Calculator

```rust
fn letter_grade(score: f64) -> char {
    if score >= 90.0 { 'A' }
    else if score >= 80.0 { 'B' }
    else if score >= 70.0 { 'C' }
    else if score >= 60.0 { 'D' }
    else { 'F' }
}

fn grade_points(letter: char) -> f64 {
    match letter {
        'A' => 4.0,
        'B' => 3.0,
        'C' => 2.0,
        'D' => 1.0,
        _   => 0.0,
    }
}

fn gpa(scores: &[f64]) -> f64 {
    let total: f64 = scores.iter()
        .map(|&s| grade_points(letter_grade(s)))
        .sum();
    total / scores.len() as f64
}

fn pass_fail(score: f64) -> &'static str {
    if score >= 60.0 { "Pass" } else { "Fail" }
}

fn main() {
    let scores = [88.5, 72.0, 95.5, 61.0, 54.0, 80.5];

    println!("{:<10} {:>6}  {:>5}  {:>6}", "Score", "Grade", "GPA", "Status");
    println!("{}", "-".repeat(35));

    for &score in &scores {
        let grade  = letter_grade(score);
        let points = grade_points(grade);
        let status = pass_fail(score);
        println!("{:<10.1} {:>6}  {:>5.1}  {:>6}", score, grade, points, status);
    }

    println!("{}", "-".repeat(35));
    println!("Final GPA: {:.2}", gpa(&scores));
}
```

**Output:**

```
Score      Grade    GPA  Status
-----------------------------------
88.5           B    3.0    Pass
72.0           C    2.0    Pass
95.5           A    4.0    Pass
61.0           D    1.0    Pass
54.0           F    0.0    Fail
80.5           B    3.0    Pass
-----------------------------------
Final GPA: 2.17
```

---

## Section D — Deep Understanding

---

### A29 — Predict the Complex Output

**Output:**

```
103
107
115
```

**Detailed trace for `mystery(1)`:**

1. `let a = { let b = 1 * 2; b + 3 }` → `b = 2`, block = `2 + 3 = 5`, `a = 5`
2. `let c = if 5 > 10 { 5*2 } else { 5+100 }` → `5 > 10` is false → `c = 5 + 100 = 105`
3. `return c - x` → `105 - 1 = 104`

Wait — let me re-trace carefully:

**`mystery(1)`:** `a = 1*2 + 3 = 5`. `5 > 10`? No → `c = 5 + 100 = 105`. Return `105 - 1 = 104`... hmm.

Actually let me retrace with the correct formula:

- `mystery(1)`: `b = 1*2 = 2`, `a = 2+3 = 5`. `a > 10`? No → `c = 5+100 = 105`. Return `105 - 1 = 104`.
- `mystery(5)`: `b = 5*2 = 10`, `a = 10+3 = 13`. `13 > 10`? Yes → `c = 13*2 = 26`. Return `26 - 5 = 21`.
- `mystery(10)`: `b = 10*2 = 20`, `a = 20+3 = 23`. `23 > 10`? Yes → `c = 23*2 = 46`. Return `46 - 10 = 36`.

**Corrected output:**

```
104
21
36
```

**Step-by-step table:**

| Call          | `x` | `b=x*2` | `a=b+3` | `a>10?` | `c`       | `c-x` |
| ------------- | --- | ------- | ------- | ------- | --------- | ----- |
| `mystery(1)`  | 1   | 2       | 5       | No      | 5+100=105 | 104   |
| `mystery(5)`  | 5   | 10      | 13      | Yes     | 13×2=26   | 21    |
| `mystery(10)` | 10  | 20      | 23      | Yes     | 23×2=46   | 36    |

---

### A30 — True or False?

| #   | Statement                                                     | Answer    | Explanation                                                                                                                                         |
| --- | ------------------------------------------------------------- | --------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Parameter types can be inferred                               | **False** | Function parameters **always** require explicit type annotations in Rust                                                                            |
| 2   | Can return multiple values via tuple                          | **True**  | A tuple is one value that bundles multiple values — this is the idiomatic pattern                                                                   |
| 3   | Semicolon on last expression returns `()`                     | **True**  | The semicolon discards the value, leaving the function to return `()`                                                                               |
| 4   | Closures cannot modify enclosing scope vars                   | **False** | `FnMut` closures can mutably capture and modify variables from their scope                                                                          |
| 5   | Function defined later cannot be called by main               | **False** | Rust doesn't require forward declarations — order of definition doesn't matter                                                                      |
| 6   | `return` at end of function is idiomatic                      | **False** | Idiomatic Rust uses implicit returns (last expression without semicolon); `return` is for early returns only                                        |
| 7   | Closure capturing immutably implements `Fn`                   | **True**  | `Fn` is for closures that only need immutable access to captured variables                                                                          |
| 8   | Nested functions access enclosing scope                       | **False** | Nested _functions_ do NOT capture the enclosing scope — only _closures_ do. Nested functions need explicit parameters                               |
| 9   | `!` return type must contain loop or panic                    | **False** | Any code path that never returns qualifies: `std::process::exit()`, infinite recursion, etc. The requirement is just "never returns"                |
| 10  | `impl Fn(i32)->i32` and `fn(i32)->i32` always interchangeable | **False** | `fn(i32)->i32` is a function _pointer_ (for named functions). `impl Fn` is a trait bound that also accepts closures. Closures are NOT `fn` pointers |

---

### A31 — Refactor the Monster Function

```rust
fn letter_grade(score: f64) -> &'static str {
    if score >= 90.0 { "A" }
    else if score >= 80.0 { "B" }
    else if score >= 70.0 { "C" }
    else if score >= 60.0 { "D" }
    else { "F" }
}

fn pass_fail_status(score: f64) -> &'static str {
    if score >= 60.0 { "PASS" } else { "FAIL" }
}

fn print_student_report(name: &str, score: f64) {
    println!("--- {} ---", name);
    println!("Score: {:.1}", score);
    println!("Grade: {} ({})", letter_grade(score), pass_fail_status(score));
}

fn find_top_student<'a>(students: &[(&'a str, f64)]) -> (&'a str, f64) {
    students.iter()
        .copied()
        .max_by(|a, b| a.1.partial_cmp(&b.1).unwrap())
        .unwrap()
}

fn class_average(students: &[(&str, f64)]) -> f64 {
    let total: f64 = students.iter().map(|&(_, s)| s).sum();
    total / students.len() as f64
}

fn print_summary(students: &[(&str, f64)]) {
    let (top_name, top_score) = find_top_student(students);
    println!("=== Summary ===");
    println!("Average: {:.2}", class_average(students));
    println!("Top student: {} ({:.1})", top_name, top_score);
}

fn process(students: &[(&str, f64)]) {
    for &(name, score) in students {
        print_student_report(name, score);
    }
    print_summary(students);
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

**Output:** (identical to original)

```
--- Alice ---
Score: 92.5
Grade: A (PASS)
--- Bob ---
Score: 74.0
Grade: C (PASS)
--- Charlie ---
Score: 58.5
Grade: F (FAIL)
--- Diana ---
Score: 88.0
Grade: B (PASS)
=== Summary ===
Average: 78.25
Top student: Alice (92.5)
```

**Why this refactor is better:**

- Each function has **one responsibility** that can be read, tested, and reasoned about alone
- `letter_grade` and `pass_fail_status` can be reused anywhere in a larger program
- `main` reads like plain English: process → students → done
- Each piece is individually testable with `assert_eq!`

---

### A32 — The Big Debug Challenge

**9 bugs identified:**

```
Bug 1:  greet(name: &str, times i32)       — missing `:` before i32
Bug 2:  sum * sum;                          — semicolon discards return value
Bug 3:  fn describe_number(n: i32) -> &str — missing lifetime: should be &'static str
Bug 4:  if/else arms return &str but no semicolons — actually fine as written,
         the real issue is the missing 'static lifetime annotation
Bug 5:  apply_twice(f: fn(i32)->i32, x i32) — missing `:` before i32
Bug 6:  println!("{}", apply_twice(double 5)) — missing comma between args: `double, 5`
Bug 7:  let square = |x: i32| -> i32 { x * x; }; — semicolon in closure discards value,
         returns () but declared -> i32
Bug 8:  (implicit) add_and_square returns i32 but the body has sum*sum; (stmt not expr)
Bug 9:  describe_number returns &str without a lifetime — in this context &'static str is needed
```

**Fully corrected program:**

```rust
fn greet(name: &str, times: i32) {         // fix 1: add `:` after times
    for _ in 0..times {
        println!("Hello, {name}");
    }
}

fn add_and_square(a: i32, b: i32) -> i32 {
    let sum = a + b;
    sum * sum                               // fix 2: remove semicolon
}

fn describe_number(n: i32) -> &'static str { // fix 3: add 'static lifetime
    if n > 0 { "positive" }
    else if n < 0 { "negative" }
    else { "zero" }
}

fn apply_twice(f: fn(i32) -> i32, x: i32) -> i32 { // fix 5: add `:` after x
    f(f(x))
}

fn double(x: i32) -> i32 { x * 2 }

fn main() {
    greet("Alice", 2);

    println!("{}", add_and_square(3, 4));    // 49

    println!("{}", describe_number(-5));     // negative
    println!("{}", describe_number(0));      // zero

    println!("{}", apply_twice(double, 5)); // fix 6: add comma → double, 5 → 20

    let square = |x: i32| -> i32 { x * x }; // fix 7: remove semicolon inside closure
    println!("{}", square(4));               // 16
}
```

**Output:**

```
Hello, Alice
Hello, Alice
49
negative
zero
20
16
```

---

## 🏆 Lesson Complete!

You've now mastered:

- ✅ Defining and calling functions with parameters and return types
- ✅ The difference between statements and expressions — and why semicolons matter
- ✅ Implicit returns (idiomatic) vs explicit `return` (for early exits)
- ✅ Returning multiple values via tuples
- ✅ Function pointers — storing and passing functions as values
- ✅ Closures — anonymous functions that capture their environment
- ✅ `Fn`, `FnMut`, and `FnOnce` trait bounds
- ✅ Closures with iterator methods (`map`, `filter`, `fold`)
- ✅ Nested functions and diverging functions (`!`)
- ✅ Documentation comments with `///`
- ✅ Refactoring large functions into clean, single-responsibility pieces

**Up next:** [Lesson 05 — Control Flow](../lesson_05_if_else/lesson_05_if_else.md) 🦀
