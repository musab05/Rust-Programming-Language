# ✅ Lesson 06 — Answers: Control Flow — Loops

> Full solutions with explanations.
> Questions are in [`lesson_06_questions.md`](./lesson_06_questions.md)

---

## Section A — Predict the Output

### A1 — loop with break
**Output:** `64`

**Trace:** x doubles each iteration: 1→2→4→8→16→32→64. The first value >50 is 64. `break 64` makes the loop expression evaluate to 64, assigned to `result`.

---

### A2 — while with counter
**Output:**
```
9
3
```
i starts at 10, decrements by 2: 10,8,6,4,2,0. The values divisible by 3 in that sequence: 9 (yes), 3 (yes). 10%3≠0, 8%3≠0, 6%3==0 — wait, 6 IS divisible by 3. Let me retrace: 10→no, 8→no, 6→yes(print 6), 4→no, 2→no, 0→no. But the loop condition is `i > 0`, so when i=0 the loop ends before printing. The values are 10,8,6,4,2. 6%3==0 so 6 prints. Also 9 — but 9 is odd and we go 10,8,6... we never reach 9 since we start at 10 and decrement by 2 (even numbers only). Actually: start=10, -2 gives 10,8,6,4,2. Only 6%3==0. Output is just `6`.

**Corrected Output:** `6`

**Trace:** i starts at 10, decrements by 2: 10, 8, 6, 4, 2. Loop stops when i≤0. 10%3≠0, 8%3≠0, 6%3=0 → print 6, 4%3≠0, 2%3≠0. Only `6` prints.

---

### A3 — for with range
**Output:**
```
n=1 sum=1
n=2 sum=3
n=3 sum=6
n=4 sum=10
n=5 sum=15
```
sum accumulates: 0+1=1, 1+2=3, 3+3=6, 6+4=10, 10+5=15.

---

### A4 — continue
**Output:** `1 3 5 `

**Trace:** i goes 0..8 (0,1,2,3,4,5,6,7).
- i=0: even → continue. Skip.
- i=1: odd, 1≤5 → print "1 "
- i=2: even → continue
- i=3: odd, 3≤5 → print "3 "
- i=4: even → continue
- i=5: odd, 5≤5 → print "5 "
- i=6: even → continue
- i=7: odd, 7>5 → break

Result: `1 3 5 ` then newline from println!().

---

### A5 — enumerate
**Output:**
```
0:zero
2:two
```
`enumerate()` gives (0,"zero"), (1,"one"), (2,"two"), (3,"three"). Condition `i % 2 == 0` filters even indices: 0 and 2.

---

### A6 — while let
**Output:**
```
total=30
total=50
total=60
```
`pop()` removes from the end: 30 first, then 20, then 10. Running totals: 30, 50, 60.

---

### A7 — nested loops with label
**Output:**
```
hit: i=0 j=4
done
```
Wait — j only goes 0..4 (0,1,2,3). Let me retrace. i+j==4: i=0,j=4 — but j only reaches 3. i=1,j=3 → 1+3=4 → hit. So:

**Corrected Output:**
```
hit: i=1 j=3
done
```
For i=0: j=0(0),j=1(1),j=2(2),j=3(3) — none equal 4. For i=1: j=0(1),j=1(2),j=2(3),j=3 → 1+3=4 → print "hit: i=1 j=3", break 'outer. Then "done".

---

### A8 — iterator chain
**Output:**
```
[1, 9, 25]
15
```
- Filter odd from [1,2,3,4,5,6]: [1,3,5]. Map square: [1,9,25].
- Filter x>3 from [1,2,3,4,5,6]: [4,5,6]. Sum: 15.

---

## Section B — Fix the Bugs

### A9 — break without value
**Bug:** `break;` with no value returns `()`, but `result` expects a value since we print it with `{}`.

```rust
// ✅ Fixed
fn main() {
    let result = loop {
        break 42;   // provide the value
    };
    println!("{result}"); // 42
}
```

---

### A10 — infinite while loop
**Bug:** `i` is never incremented inside the loop — it stays at 0 forever. Infinite loop.

```rust
// ✅ Fixed
fn main() {
    let mut i = 0;
    while i < 5 {
        println!("{i}");
        i += 1;   // must increment!
    }
}
```

---

### A11 — backwards range
**Bug:** `5..0` is an empty range in Rust — ranges go forward only. No iterations occur.

```rust
// ✅ Fixed — use .rev() on an inclusive range
fn main() {
    for i in (1..=5).rev() {
        println!("{i}");
    }
}
// Output: 5 4 3 2 1
```

---

### A12 — iter_mut then use after
**Bug:** `v.iter_mut()` borrows `v` mutably. But the code doesn't actually mutate anything — and then accesses `v` afterwards which is fine. Actually `iter_mut` on a vec that isn't `mut` would fail — `v` isn't declared `mut`.

```rust
// ✅ Fixed — if you only need to read, use .iter() not .iter_mut()
fn main() {
    let v = vec![1, 2, 3];
    for x in v.iter() {
        println!("{x}");
    }
    println!("{v:?}"); // [1, 2, 3]
}
```

---

### A13 — Logic analysis
**Answer:** The code IS correct. Count equals **40**.

Numbers 1..=100 that are odd (not divisible by 2): 50 numbers.
Of those 50 odd numbers, the ones divisible by 5 (odd multiples of 5: 5,15,25,35,45,55,65,75,85,95): 10 numbers.
50 - 10 = **40**.

The programmer's expectation of 40 was correct! The code gives 40.

---

### A14 — max returns Option
**Bug:** `.max()` returns `Option<&i32>`, not `i32`. You must unwrap it.

```rust
// ✅ Fixed
fn main() {
    let data = vec![3, 6, 2, 8, 1];
    let max: i32 = *data.iter().max().unwrap();  // deref & unwrap
    println!("{max}"); // 8
}
```

---

## Section C — Write It Yourself

### A15 — FizzBuzz with loop

```rust
fn main() {
    let mut n = 1;
    loop {
        if n % 15 == 0 {
            println!("FizzBuzz");
        } else if n % 3 == 0 {
            println!("Fizz");
        } else if n % 5 == 0 {
            println!("Buzz");
        } else {
            println!("{n}");
        }
        if n == 20 { break; }
        n += 1;
    }
}
```

---

### A16 — Collatz sequence

```rust
fn collatz_length(mut n: u64) -> u64 {
    let mut steps = 0;
    while n != 1 {
        n = if n % 2 == 0 { n / 2 } else { 3 * n + 1 };
        steps += 1;
    }
    steps
}

fn main() {
    let mut max_len = 0u64;
    let mut max_n = 1u64;

    for n in 1..=100 {
        let len = collatz_length(n);
        if len > max_len {
            max_len = len;
            max_n = n;
        }
    }

    println!("Longest Collatz in 1..=100: n={max_n} with {max_len} steps");
}
```
**Output:** `Longest Collatz in 1..=100: n=97 with 118 steps`

---

### A17 — Prime sieve

```rust
fn primes_up_to(limit: u64) -> Vec<u64> {
    let mut primes = Vec::new();
    for n in 2..=limit {
        let is_prime = (2..n).all(|i| n % i != 0);
        if is_prime {
            primes.push(n);
        }
    }
    primes
}

fn main() {
    let primes = primes_up_to(50);
    println!("Primes up to 50: {:?}", primes);
    println!("Count: {}", primes.len());
}
```
**Output:** `Primes up to 50: [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]`

---

### A18 — Running statistics

```rust
fn running_stats(data: &[f64]) -> Vec<(f64, f64, f64)> {
    let mut result = Vec::new();
    let mut running_sum = 0.0;

    for (i, &val) in data.iter().enumerate() {
        running_sum += val;
        let running_avg = running_sum / (i + 1) as f64;
        result.push((val, running_sum, running_avg));
    }
    result
}

fn main() {
    let data = [4.0, 7.0, 2.0, 9.0, 5.0];
    let stats = running_stats(&data);

    println!("{:<8} {:>10} {:>12}", "Value", "RunSum", "RunAvg");
    println!("{}", "-".repeat(32));
    for (val, sum, avg) in stats {
        println!("{:<8.1} {:>10.1} {:>12.2}", val, sum, avg);
    }
}
```
**Output:**
```
Value      RunSum       RunAvg
--------------------------------
4.0           4.0           4.00
7.0          11.0           5.50
2.0          13.0           4.33
9.0          22.0           5.50
5.0          27.0           5.40
```

---

### A19 — Iterator pipeline

```rust
fn main() {
    let data = [5, 12, 3, 18, 7, 22, 1, 15, 9, 30];

    let result: Vec<i32> = data.iter()
        .filter(|&&x| x > 10)
        .map(|&x| x * 3)
        .collect();

    println!("Filtered & tripled: {:?}", result);
    println!("Count: {}", result.len());
    println!("Sum:   {}", result.iter().sum::<i32>());
    println!("Max:   {}", result.iter().max().unwrap());
    println!("Min:   {}", result.iter().min().unwrap());
}
```
**Output:**
```
Filtered & tripled: [36, 54, 66, 45, 90]
Count: 5
Sum:   291
Max:   90
Min:   36
```

---

### A20 — Word frequency

```rust
fn main() {
    let sentence = "the cat sat on the mat the cat sat";
    let words: Vec<&str> = sentence.split_whitespace().collect();

    let mut counts: Vec<(String, usize)> = Vec::new();

    for word in &words {
        let word = word.to_string();
        if let Some(entry) = counts.iter_mut().find(|(w, _)| w == &word) {
            entry.1 += 1;
        } else {
            counts.push((word, 1));
        }
    }

    counts.sort_by(|a, b| a.0.cmp(&b.0));

    for (word, count) in &counts {
        println!("{word}: {count}");
    }
}
```
**Output:**
```
cat: 2
mat: 1
on: 1
sat: 2
the: 3
```

---

### A21 — Matrix spiral

```rust
fn main() {
    let matrix = [
        [ 1,  2,  3,  4],
        [ 5,  6,  7,  8],
        [ 9, 10, 11, 12],
        [13, 14, 15, 16],
    ];

    let mut spiral = Vec::new();
    let (mut top, mut bottom, mut left, mut right) = (0usize, 3usize, 0usize, 3usize);

    while top <= bottom && left <= right {
        for c in left..=right   { spiral.push(matrix[top][c]); }
        top += 1;
        for r in top..=bottom   { spiral.push(matrix[r][right]); }
        if right > 0 { right -= 1; }
        if top <= bottom {
            for c in (left..=right).rev() { spiral.push(matrix[bottom][c]); }
            if bottom > 0 { bottom -= 1; }
        }
        if left <= right {
            for r in (top..=bottom).rev() { spiral.push(matrix[r][left]); }
            left += 1;
        }
    }

    let spiral_str: Vec<String> = spiral.iter().map(|x| x.to_string()).collect();
    println!("{}", spiral_str.join(" "));
}
```
**Output:** `1 2 3 4 8 12 16 15 14 13 9 5 6 7 11 10`

---

### A22 — Retry loop

```rust
fn attempt_login(passwords: &[&str], correct: &str) -> (bool, usize) {
    let mut attempt = 0;
    loop {
        if attempt >= passwords.len() {
            return (false, attempt);
        }
        attempt += 1;
        if passwords[attempt - 1] == correct {
            return (true, attempt);
        }
    }
}

fn main() {
    let tries = ["wrong1", "wrong2", "secret", "extra"];
    let (success, count) = attempt_login(&tries, "secret");
    println!("Success: {success}, Attempts: {count}");

    let bad_tries = ["nope", "nah", "never"];
    let (success, count) = attempt_login(&bad_tries, "secret");
    println!("Success: {success}, Attempts: {count}");
}
```
**Output:**
```
Success: true, Attempts: 3
Success: false, Attempts: 3
```

---

## Section D — Deep Understanding

### A23 — Predict complex output

**Output:** `10`

**Trace:** `'a: for i in 0..5 { for j in 0..5 { if j >= i { continue 'a; } count += 1; } }`

For each i, we count j values where j < i before `continue 'a` fires:
- i=0: j=0 → 0 >= 0 → continue 'a immediately. count += 0.
- i=1: j=0 → 0 < 1 → count++(1). j=1 → 1 >= 1 → continue 'a. count += 1.
- i=2: j=0(+1), j=1(+1), j=2 → continue. count += 2.
- i=3: j=0(+1), j=1(+1), j=2(+1), j=3 → continue. count += 3.
- i=4: j=0(+1), j=1(+1), j=2(+1), j=3(+1), j=4 → continue. count += 4.

Total: 0+1+2+3+4 = **10**

---

### A24 — True or False?

| # | Statement | Answer | Explanation |
|---|---|---|---|
| 1 | `for` can return value with `break value` | **False** | Only `loop` can return a value via `break value`. `for` and `while` return `()`. |
| 2 | `while let` stops when pattern fails | **True** | That's its exact purpose. |
| 3 | `.map()` processes elements immediately | **False** | Iterators are lazy — processing only happens when consumed (e.g., `.collect()`). |
| 4 | `break 'outer` exits only inner loop | **False** | `break 'outer` exits the loop labeled `'outer`, which IS the outer loop. |
| 5 | `.into_iter()` on Vec consumes it | **True** | `into_iter()` takes ownership. The Vec cannot be used afterwards. |
| 6 | `for i in 5..0` iterates 5 down to 1 | **False** | `5..0` is an empty range. Nothing iterates. Use `(1..=5).rev()`. |
| 7 | `continue 'label` skips to next iteration of labeled loop | **True** | Exactly — it skips the rest of the current iteration of that specific loop. |
| 8 | `loop` is only loop that returns non-unit value | **True** | Only `loop` supports `break value`. `while` and `for` always return `()`. |

---

### A25 — Big Debug Challenge (8 bugs)

```
Bug 1: loop { println!("{n}"); n -= 1; } — no break condition → infinite loop
Bug 2: (1..20) should be (1..=20) to include 20
Bug 3: x ^ 2 is XOR, not squaring — should be x * x or x.pow(2)
Bug 4: .filter(|x| x % 2 == 0) — x is &&i32, need to deref: |&&x| x % 2 == 0
Bug 5: .map(|x| x ^ 2) — same deref issue: |&x| x * x
Bug 6: data.iter().find(|x| x > 100) — x is &&i32, need: |&&x| x > 100
Bug 7: if i % 3 = 0 — assignment not comparison, should be ==
Bug 8: The found result prints as Option, which is correct, but the deref in find needs fixing
```

**Fully corrected program:**
```rust
fn main() {
    // Count down from 10 to 1 — fix 1: add break condition
    let mut n = 10;
    loop {
        println!("{n}");
        n -= 1;
        if n == 0 { break; }
    }

    // Fix 2: ..=20, Fix 3: x*x, Fix 4&5: deref pattern
    let sum: i32 = (1..=20_i32)
        .filter(|&x| x % 2 == 0)
        .map(|x| x * x)
        .sum();
    println!("Sum of even squares: {sum}");

    // Fix 6: deref pattern in find
    let data = vec![10, 50, 80, 120, 200];
    let found = data.iter().find(|&&x| x > 100);
    println!("{found:?}");

    // Fix 7: == not =
    for (i, val) in data.iter().enumerate() {
        if i % 3 == 0 { println!("{val}"); }
    }
}
```

---

### A26 — Design: which loop?

1. **Read until "quit"** → `while let` or `loop` with break
```rust
let inputs = vec!["hello", "world", "quit", "ignored"];
let mut i = 0;
loop {
    let line = inputs[i];
    if line == "quit" { break; }
    println!("Got: {line}");
    i += 1;
}
```

2. **Process shopping cart Vec** → `for`
```rust
let cart = vec![12.99, 5.50, 8.75];
for &price in &cart { println!("Item: ${price:.2}"); }
```

3. **First Fibonacci > 1000** → `loop`
```rust
let (mut a, mut b) = (0u64, 1u64);
let result = loop {
    let next = a + b;
    a = b; b = next;
    if b > 1000 { break b; }
};
println!("First Fibonacci > 1000: {result}");
```

4. **Retry network up to 5 times** → `loop` with counter
```rust
let mut attempts = 0;
let success = loop {
    attempts += 1;
    if simulate_request() { break true; }
    if attempts >= 5 { break false; }
};
```

5. **Triangular asterisks** → nested `for`
```rust
for row in 1..=5 {
    for _ in 0..row { print!("* "); }
    println!();
}
```

---

## 🏆 Lesson Complete!

You've mastered:
- ✅ `loop` — infinite loops, `break` with values, loop labels
- ✅ `while` — condition-driven, `while let` for Option/pattern loops
- ✅ `for` — ranges (exclusive/inclusive), collections, `.iter()/.iter_mut()/.into_iter()`
- ✅ `break` and `continue` with and without labels
- ✅ Nested loops and labeled jumps
- ✅ Iterator adaptor pipeline: `map`, `filter`, `fold`, `any`, `all`, `find`, `enumerate`, `zip`, `rev`, `take`, `skip`
- ✅ Choosing the right loop for each situation

**Up next:** [Lesson 07 — Comments & Documentation](../lesson_07_comments/lesson_07_comments.md) 🦀
