# 📘 Lesson 04 — Functions in Rust

> **Series:** Learning Rust from Scratch
> **Difficulty:** Beginner → Intermediate
> **Prerequisite:** [Lesson 03 — Compound Types](../lesson_03/lesson_03_compound_types.md)
>
> **Files in this lesson:**
> - [`lesson_04_functions.md`](./lesson_04_functions.md) ← You are here
> - [`lesson_04_questions.md`](./lesson_04_questions.md) — Practice problems
> - [`lesson_04_answers.md`](./lesson_04_answers.md) — Solutions & explanations

---

## Table of Contents

1. [What is a Function?](#1-what-is-a-function)
2. [Defining and Calling Functions](#2-defining-and-calling-functions)
3. [Function Naming Conventions](#3-function-naming-conventions)
4. [Parameters and Arguments](#4-parameters-and-arguments)
   - 4.1 [Single Parameter](#41-single-parameter)
   - 4.2 [Multiple Parameters](#42-multiple-parameters)
   - 4.3 [Why Types Are Required on Parameters](#43-why-types-are-required-on-parameters)
5. [Statements vs Expressions](#5-statements-vs-expressions)
   - 5.1 [Statements](#51-statements)
   - 5.2 [Expressions](#52-expressions)
   - 5.3 [The Critical Difference — Semicolons](#53-the-critical-difference--semicolons)
   - 5.4 [Block Expressions](#54-block-expressions)
6. [Return Values](#6-return-values)
   - 6.1 [Declaring a Return Type](#61-declaring-a-return-type)
   - 6.2 [Implicit Return — The Last Expression](#62-implicit-return--the-last-expression)
   - 6.3 [Explicit Return with `return`](#63-explicit-return-with-return)
   - 6.4 [Early Returns](#64-early-returns)
   - 6.5 [Returning Nothing — The Unit Type](#65-returning-nothing--the-unit-type)
7. [Returning Multiple Values](#7-returning-multiple-values)
8. [Functions as Values](#8-functions-as-values)
   - 8.1 [Function Pointers](#81-function-pointers)
   - 8.2 [Passing Functions as Arguments](#82-passing-functions-as-arguments)
9. [Closures — Anonymous Functions](#9-closures--anonymous-functions)
   - 9.1 [Basic Closure Syntax](#91-basic-closure-syntax)
   - 9.2 [Closures Capture Their Environment](#92-closures-capture-their-environment)
   - 9.3 [Closures as Parameters](#93-closures-as-parameters)
   - 9.4 [Common Closure Uses with Iterators](#94-common-closure-uses-with-iterators)
10. [Nested Functions](#10-nested-functions)
11. [Diverging Functions — `!`](#11-diverging-functions--)
12. [Documentation Comments](#12-documentation-comments)
13. [Common Patterns and Best Practices](#13-common-patterns-and-best-practices)
14. [Summary Cheat Sheet](#14-summary-cheat-sheet)

---

## 1. What is a Function?

A **function** is a named, reusable block of code that performs a specific task. Functions are the primary way to **organize**, **reuse**, and **abstract** logic in Rust.

Every Rust program has at least one function: `main`.

```rust
fn main() {
    // every Rust program starts here
    println!("Hello, world!");
}
```

Functions allow you to:
- **Avoid repetition** — write logic once, call it many times
- **Abstract complexity** — hide implementation details behind a clear name
- **Test in isolation** — verify small units of logic independently
- **Compose behavior** — combine small functions to build complex systems

In Rust, functions are **first-class citizens** — they can be stored in variables, passed as arguments, and returned from other functions.

---

## 2. Defining and Calling Functions

A function is defined with the `fn` keyword:

```
fn  <name>  (  <parameters>  )  ->  <return_type>  {
    <body>
}
```

```rust
// Defining a function
fn greet() {
    println!("Hello from a function!");
}

// Calling a function
fn main() {
    greet(); // call it — note the ()
    greet(); // call it again — code is reused
    greet(); // and again
}
```

**Output:**
```
Hello from a function!
Hello from a function!
Hello from a function!
```

### Order doesn't matter

Unlike C/C++, Rust does **not** require functions to be declared before they are used. You can call a function defined later in the file:

```rust
fn main() {
    say_goodbye(); // called before it is defined — perfectly fine in Rust
}

fn say_goodbye() {
    println!("Goodbye!");
}
```

This works because the Rust compiler reads the entire file before processing anything, so all function definitions are known before checking any calls.

---

## 3. Function Naming Conventions

Rust uses `snake_case` for function names — all lowercase with underscores between words:

```rust
// ✅ Correct Rust naming
fn calculate_area()   { }
fn get_user_name()    { }
fn is_valid_email()   { }
fn parse_config_file(){ }

// ❌ Wrong — these compile but violate convention (compiler will warn)
fn CalculateArea()    { } // PascalCase — for types/structs
fn calculateArea()    { } // camelCase — not Rust style
fn CALCULATE_AREA()   { } // SCREAMING_SNAKE_CASE — for constants
```

Good function names are **verbs** or **verb phrases** that describe what the function *does*:

```rust
fn add(a: i32, b: i32) -> i32 { a + b }
fn is_even(n: i32) -> bool { n % 2 == 0 }
fn find_max(values: &[i32]) -> i32 { *values.iter().max().unwrap() }
fn convert_to_celsius(fahrenheit: f64) -> f64 { (fahrenheit - 32.0) * 5.0 / 9.0 }
```

---

## 4. Parameters and Arguments

**Parameters** are the variables listed in the function definition.
**Arguments** are the actual values passed when calling the function.

```rust
fn add(a: i32, b: i32) -> i32 { // a and b are PARAMETERS
    a + b
}

fn main() {
    let result = add(3, 7); // 3 and 7 are ARGUMENTS
    println!("{result}");   // 10
}
```

### 4.1 Single Parameter

```rust
fn square(n: i32) -> i32 {
    n * n
}

fn print_line(ch: char) {
    println!("{}", ch.to_string().repeat(20));
}

fn main() {
    println!("{}", square(5));  // 25
    println!("{}", square(12)); // 144

    print_line('='); // ====================
    print_line('-'); // --------------------
}
```

### 4.2 Multiple Parameters

```rust
fn describe_person(name: &str, age: u32, is_active: bool) {
    println!("Name: {name}, Age: {age}, Active: {is_active}");
}

fn power(base: f64, exponent: u32) -> f64 {
    base.powi(exponent as i32)
}

fn clamp_value(value: i32, min: i32, max: i32) -> i32 {
    if value < min { min }
    else if value > max { max }
    else { value }
}

fn main() {
    describe_person("Alice", 30, true);
    // Name: Alice, Age: 30, Active: true

    println!("{}", power(2.0, 10));  // 1024.0
    println!("{}", power(3.0, 3));   // 27.0

    println!("{}", clamp_value(150, 0, 100)); // 100
    println!("{}", clamp_value(-5, 0, 100));  // 0
    println!("{}", clamp_value(42, 0, 100));  // 42
}
```

### 4.3 Why Types Are Required on Parameters

Unlike local variables (where Rust can infer the type from usage), function parameters **always** require explicit type annotations:

```rust
// ❌ This does NOT compile
fn double(x) { // error: expected one of `:`, `@`, found `)`
    x * 2
}

// ✅ This is correct
fn double(x: i32) -> i32 {
    x * 2
}
```

**Why?** The Rust compiler uses parameter types to resolve function calls, perform type checking, and generate the correct machine code. Without explicit types, every call to the function would require the compiler to infer types from context — which can be ambiguous, slow, and error-prone. Explicit types make the function's contract crystal clear.

---

## 5. Statements vs Expressions

This is one of the most important — and most misunderstood — concepts in Rust. Understanding it is the key to writing idiomatic Rust code.

### 5.1 Statements

A **statement** performs an action and **does not produce a value**. Statements end with a semicolon `;`.

```rust
let x = 5;           // statement — declares a variable
let y = 6;           // statement
println!("{}", x);   // statement — performs I/O
x + y;               // statement — computes but throws away the result
```

Statements cannot be used where a value is expected:

```rust
let z = (let x = 5); // ❌ ERROR — a let statement has no value
```

### 5.2 Expressions

An **expression** evaluates to a value. Most things in Rust are expressions.

```rust
5           // expression — evaluates to 5
5 + 6       // expression — evaluates to 11
"hello"     // expression — evaluates to &str
true        // expression — evaluates to bool
square(3)   // expression — evaluates to whatever square returns
{           // block — also an expression!
    let x = 1;
    let y = 2;
    x + y   // the last line (no semicolon) is the block's value: 3
}
```

### 5.3 The Critical Difference — Semicolons

This is the single most important rule in Rust:

> **Adding a semicolon `;` to an expression turns it into a statement — it discards the value.**

```rust
fn add_expression(a: i32, b: i32) -> i32 {
    a + b   // ✅ No semicolon — this IS the return value
}

fn add_statement(a: i32, b: i32) -> i32 {
    a + b;  // ❌ Semicolon added — value is discarded!
            // Now this function returns () but says it returns i32
            // COMPILE ERROR
}
```

The compiler error for the second version:
```
error[E0308]: mismatched types
  |
  | fn add_statement(a: i32, b: i32) -> i32 {
  |                                     ^^^ expected `i32`, found `()`
  |     a + b;
  |          ^ help: remove this semicolon to return this value
```

Notice: Rust's error message even **tells you** to remove the semicolon!

### 5.4 Block Expressions

A **block** `{ ... }` is itself an expression in Rust. Its value is the last expression inside it (without a semicolon):

```rust
fn main() {
    // A block used as an expression
    let y = {
        let x = 3;
        x * x + 1  // no semicolon — this is the block's value
    };             // the block's value (10) is assigned to y
    println!("y = {y}"); // y = 10

    // Blocks can be nested
    let result = {
        let a = 5;
        let b = {
            let inner = 10;
            inner * 2   // inner block's value: 20
        };
        a + b           // outer block's value: 25
    };
    println!("result = {result}"); // result = 25

    // if is also an expression
    let grade = if result >= 90 { 'A' }
                else if result >= 80 { 'B' }
                else { 'C' };
    println!("grade = {grade}"); // grade = C (25 < 80)
}
```

---

## 6. Return Values

### 6.1 Declaring a Return Type

The return type is declared after `->` in the function signature:

```rust
fn add(a: i32, b: i32) -> i32 { ... }
//                         ^^^
//                         return type

fn get_greeting(name: &str) -> String { ... }
fn is_palindrome(s: &str) -> bool { ... }
fn get_origin() -> (f64, f64) { ... }
fn find_index(arr: &[i32], target: i32) -> Option<usize> { ... }
```

### 6.2 Implicit Return — The Last Expression

In Rust, a function **implicitly returns** the value of its last expression — no `return` keyword needed:

```rust
fn square(n: i32) -> i32 {
    n * n   // last expression, no semicolon → this is returned
}

fn max_of_three(a: i32, b: i32, c: i32) -> i32 {
    let ab = if a > b { a } else { b };
    if ab > c { ab } else { c } // last expression → returned
}

fn celsius_to_fahrenheit(c: f64) -> f64 {
    c * 9.0 / 5.0 + 32.0  // formula is the return value
}

fn main() {
    println!("{}", square(7));                   // 49
    println!("{}", max_of_three(3, 9, 5));       // 9
    println!("{}", celsius_to_fahrenheit(100.0)); // 212
}
```

This style is strongly preferred in Rust. The last expression *is* the return value — clean and concise.

### 6.3 Explicit Return with `return`

You can also use the `return` keyword for an explicit return:

```rust
fn square(n: i32) -> i32 {
    return n * n; // explicit return — works fine
}
```

Both styles compile to the same thing. However, using `return` at the end of a function (when it's not an early return) is considered **non-idiomatic** in Rust. Reserve `return` for early returns (see below).

### 6.4 Early Returns

`return` is valuable for **returning early** when a condition is met — before reaching the end of the function:

```rust
fn divide(a: f64, b: f64) -> f64 {
    if b == 0.0 {
        return 0.0; // early return — skip division by zero
    }
    a / b // reached only when b != 0.0
}

fn find_first_negative(numbers: &[i32]) -> Option<i32> {
    for &n in numbers {
        if n < 0 {
            return Some(n); // found one — return immediately
        }
    }
    None // no negative found — implicit return at the end
}

fn validate_age(age: i32) -> &'static str {
    if age < 0 {
        return "Age cannot be negative";
    }
    if age > 150 {
        return "Age is unrealistically high";
    }
    if age < 18 {
        return "Minor";
    }
    "Adult" // default — falls through all checks
}

fn main() {
    println!("{}", divide(10.0, 2.0));  // 5.0
    println!("{}", divide(10.0, 0.0));  // 0.0

    let nums = [3, 7, -2, 5, -8];
    println!("{:?}", find_first_negative(&nums)); // Some(-2)

    println!("{}", validate_age(-5));   // Age cannot be negative
    println!("{}", validate_age(25));   // Adult
    println!("{}", validate_age(15));   // Minor
}
```

Early returns make the **"happy path"** clear — the main logic at the bottom of the function is for the normal case, while edge cases are handled at the top with early returns.

### 6.5 Returning Nothing — The Unit Type

Functions that don't return a meaningful value implicitly return `()` (the unit type):

```rust
fn print_separator() {         // implicitly returns ()
    println!("{}", "=".repeat(40));
}

fn print_separator_explicit() -> () { // explicit — same thing
    println!("{}", "=".repeat(40));
}

fn main() {
    let result = print_separator(); // you can capture (), but why would you
    println!("{:?}", result);       // ()
}
```

You'll mainly encounter `()` in:
- Function signatures for side-effect-only functions
- `Result<(), Error>` — success with no value, failure with an error
- Closures that perform actions but return nothing

---

## 7. Returning Multiple Values

Rust functions can return only **one value**, but that value can be a **tuple** — which effectively lets you return multiple values:

```rust
fn min_max(arr: &[i32]) -> (i32, i32) {
    let mut min = arr[0];
    let mut max = arr[0];

    for &val in arr {
        if val < min { min = val; }
        if val > max { max = val; }
    }

    (min, max) // return a tuple of two values
}

fn euclidean_div(a: i32, b: i32) -> (i32, i32) {
    (a / b, a % b) // returns (quotient, remainder)
}

fn split_name(full_name: &str) -> (&str, &str) {
    // Find the space and split on it
    let space_pos = full_name.find(' ').unwrap_or(full_name.len());
    let first = &full_name[..space_pos];
    let last  = if space_pos < full_name.len() {
        &full_name[space_pos + 1..]
    } else {
        ""
    };
    (first, last)
}

fn main() {
    let scores = [42, 7, 98, 15, 63, 31];
    let (lo, hi) = min_max(&scores);
    println!("Min: {lo}, Max: {hi}"); // Min: 7, Max: 98

    let (q, r) = euclidean_div(17, 5);
    println!("17 ÷ 5 = {q} remainder {r}"); // 17 ÷ 5 = 3 remainder 2

    let (first, last) = split_name("Alice Wonderland");
    println!("First: {first}, Last: {last}"); // First: Alice, Last: Wonderland
}
```

---

## 8. Functions as Values

In Rust, functions are **first-class values** — they can be assigned to variables and passed as arguments.

### 8.1 Function Pointers

A function pointer type is written as `fn(param_types) -> return_type`:

```rust
fn add(a: i32, b: i32) -> i32 { a + b }
fn sub(a: i32, b: i32) -> i32 { a - b }
fn mul(a: i32, b: i32) -> i32 { a * b }

fn main() {
    // Assign a function to a variable
    let operation: fn(i32, i32) -> i32 = add;

    println!("{}", operation(10, 3)); // 13

    // Change which function the variable points to
    let operation = sub;
    println!("{}", operation(10, 3)); // 7

    // Store functions in an array
    let ops: [fn(i32, i32) -> i32; 3] = [add, sub, mul];
    for op in ops {
        println!("{}", op(10, 3)); // 13, 7, 30
    }
}
```

### 8.2 Passing Functions as Arguments

```rust
fn apply(f: fn(i32) -> i32, value: i32) -> i32 {
    f(value)
}

fn double(x: i32) -> i32  { x * 2 }
fn square(x: i32) -> i32  { x * x }
fn negate(x: i32) -> i32  { -x    }

fn apply_to_each(arr: &[i32], f: fn(i32) -> i32) -> Vec<i32> {
    let mut result = Vec::new();
    for &x in arr {
        result.push(f(x));
    }
    result
}

fn main() {
    println!("{}", apply(double, 5));  // 10
    println!("{}", apply(square, 5));  // 25
    println!("{}", apply(negate, 5));  // -5

    let nums = [1, 2, 3, 4, 5];
    let doubled = apply_to_each(&nums, double);
    let squared = apply_to_each(&nums, square);

    println!("{:?}", doubled); // [2, 4, 6, 8, 10]
    println!("{:?}", squared); // [1, 4, 9, 16, 25]
}
```

---

## 9. Closures — Anonymous Functions

A **closure** is an **anonymous function** (a function without a name) that can **capture variables from its surrounding scope**. Closures are one of Rust's most powerful and commonly used features.

### 9.1 Basic Closure Syntax

```rust
fn main() {
    // Named function
    fn add_ten(x: i32) -> i32 { x + 10 }

    // Equivalent closure — stored in a variable
    let add_ten_closure = |x: i32| -> i32 { x + 10 };

    // Closures with type inference (most common form)
    let add_ten_inferred = |x| x + 10; // types inferred from usage

    // Single expression — no braces needed
    let square = |x: i32| x * x;

    // Multi-line closure — use braces
    let describe = |n: i32| {
        let label = if n > 0 { "positive" }
                    else if n < 0 { "negative" }
                    else { "zero" };
        println!("{n} is {label}");
    };

    println!("{}", add_ten(5));           // 15
    println!("{}", add_ten_closure(5));   // 15
    println!("{}", add_ten_inferred(5));  // 15
    println!("{}", square(7));            // 49
    describe(-3);                         // -3 is negative
}
```

**Syntax comparison:**

```
Named function:   fn add(x: i32, y: i32) -> i32 { x + y }
Full closure:     |x: i32, y: i32| -> i32 { x + y }
Inferred closure: |x, y| { x + y }
Single-expr:      |x, y| x + y
```

The `||` are the **closure's parameter delimiters** (think of them as the closure's "ears"). Parameters go between the pipes.

### 9.2 Closures Capture Their Environment

This is what makes closures fundamentally different from regular functions — they can **capture and use variables from the scope they are defined in**:

```rust
fn main() {
    let threshold = 10; // variable in the outer scope

    // This closure CAPTURES threshold from the surrounding scope
    let is_above_threshold = |x| x > threshold;

    println!("{}", is_above_threshold(5));   // false
    println!("{}", is_above_threshold(15));  // true
    println!("{}", is_above_threshold(10));  // false

    // A regular function cannot do this:
    // fn check(x: i32) -> bool { x > threshold } // ❌ ERROR — threshold not in scope
}
```

```rust
fn main() {
    let prefix = "Hello";
    let suffix = "!";

    let make_greeting = |name: &str| -> String {
        format!("{prefix}, {name}{suffix}") // captures both prefix and suffix
    };

    println!("{}", make_greeting("Alice")); // Hello, Alice!
    println!("{}", make_greeting("Bob"));   // Hello, Bob!
}
```

```rust
fn main() {
    let mut count = 0;

    // Closure that mutably captures count
    let mut increment = || {
        count += 1;  // modifies the captured variable
        count
    };

    println!("{}", increment()); // 1
    println!("{}", increment()); // 2
    println!("{}", increment()); // 3
}
```

### 9.3 Closures as Parameters

When a function accepts a closure, you use the `Fn`, `FnMut`, or `FnOnce` trait bounds:

| Trait | Captures | Can be called |
|-------|----------|---------------|
| `Fn` | Immutably (or not at all) | Multiple times |
| `FnMut` | Mutably | Multiple times |
| `FnOnce` | By value (takes ownership) | Only once |

```rust
// Accepts any closure that takes an i32 and returns a bool
fn filter_positives(numbers: &[i32], predicate: impl Fn(i32) -> bool) -> Vec<i32> {
    let mut result = Vec::new();
    for &n in numbers {
        if predicate(n) {
            result.push(n);
        }
    }
    result
}

fn apply_and_print(value: i32, f: impl Fn(i32) -> i32) {
    println!("f({value}) = {}", f(value));
}

fn main() {
    let numbers = [-3, 0, 5, -1, 8, 2, -7, 4];

    // Pass a closure directly
    let positives = filter_positives(&numbers, |n| n > 0);
    println!("{:?}", positives); // [5, 8, 2, 4]

    // Capture from environment inside the closure
    let threshold = 3;
    let above_threshold = filter_positives(&numbers, |n| n > threshold);
    println!("{:?}", above_threshold); // [5, 8, 4]

    apply_and_print(5, |x| x * x);    // f(5) = 25
    apply_and_print(5, |x| x + 100);  // f(5) = 105
}
```

### 9.4 Common Closure Uses with Iterators

Closures shine when used with Rust's iterator methods:

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // .map() — transform each element
    let doubled: Vec<i32> = numbers.iter().map(|&x| x * 2).collect();
    println!("{:?}", doubled); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

    // .filter() — keep elements matching a condition
    let evens: Vec<&i32> = numbers.iter().filter(|&&x| x % 2 == 0).collect();
    println!("{:?}", evens); // [2, 4, 6, 8, 10]

    // .map + filter chained
    let even_squares: Vec<i32> = numbers.iter()
        .filter(|&&x| x % 2 == 0)
        .map(|&x| x * x)
        .collect();
    println!("{:?}", even_squares); // [4, 16, 36, 64, 100]

    // .fold() — accumulate a value (like reduce in other languages)
    let sum = numbers.iter().fold(0, |acc, &x| acc + x);
    println!("Sum: {sum}"); // 55

    // .any() — true if any element matches
    let has_negative = numbers.iter().any(|&x| x < 0);
    println!("{has_negative}"); // false

    // .all() — true if all elements match
    let all_positive = numbers.iter().all(|&x| x > 0);
    println!("{all_positive}"); // true

    // .find() — returns the first match as Option
    let first_over_five = numbers.iter().find(|&&x| x > 5);
    println!("{:?}", first_over_five); // Some(6)

    // .position() — returns the index of the first match
    let pos = numbers.iter().position(|&x| x == 7);
    println!("{:?}", pos); // Some(6)
}
```

---

## 10. Nested Functions

Rust allows defining functions **inside other functions**. These inner functions are only visible within the enclosing function:

```rust
fn compute_statistics(data: &[f64]) -> (f64, f64, f64) {
    // Helper functions defined inside — only accessible here
    fn sum(arr: &[f64]) -> f64 {
        arr.iter().sum()
    }

    fn mean(arr: &[f64]) -> f64 {
        sum(arr) / arr.len() as f64
    }

    fn variance(arr: &[f64]) -> f64 {
        let m = mean(arr);
        arr.iter().map(|&x| (x - m).powi(2)).sum::<f64>() / arr.len() as f64
    }

    let avg = mean(data);
    let var = variance(data);
    let std_dev = var.sqrt();

    (avg, var, std_dev)
}

fn main() {
    let data = [2.0, 4.0, 4.0, 4.0, 5.0, 5.0, 7.0, 9.0];
    let (mean, variance, std_dev) = compute_statistics(&data);

    println!("Mean:     {mean:.2}");     // 5.00
    println!("Variance: {variance:.2}"); // 4.00
    println!("Std Dev:  {std_dev:.2}");  // 2.00

    // sum(), mean(), variance() are NOT accessible here — they're private to compute_statistics
}
```

**Note:** Unlike closures, nested functions do **not** capture the enclosing scope — they're just regular functions that happen to be defined inside another function. They must have all their needed values passed in as parameters.

---

## 11. Diverging Functions — `!`

Some functions **never return** — they either loop forever, panic, or terminate the process. These are called **diverging functions** and their return type is `!` (pronounced "never"):

```rust
// panic! — terminates with an error message
fn fatal_error(message: &str) -> ! {
    panic!("Fatal: {message}");
}

// infinite loop — never returns
fn event_loop() -> ! {
    loop {
        // process events forever
    }
}

// process::exit — terminates the entire process
fn quit(code: i32) -> ! {
    std::process::exit(code);
}
```

Why is `!` useful? Because it can be used anywhere a type is expected — the compiler knows the code after a diverging call is **unreachable**:

```rust
fn main() {
    let number: i32 = "42".parse().unwrap_or_else(|_| {
        panic!("Failed to parse!") // ! can coerce to any type
    });
    println!("{number}"); // 42

    // The unimplemented! macro also returns !
    fn not_yet_done() -> i32 {
        unimplemented!("this function isn't written yet")
    }

    // todo!() is similar — marks code to be done later
    fn placeholder() -> String {
        todo!("implement this later")
    }
}
```

Common diverging macros:
- `panic!("message")` — crash with an error
- `unimplemented!()` — marks a function that hasn't been written yet
- `todo!()` — marks code to be completed later
- `unreachable!()` — marks code the compiler thinks could run but logically cannot

---

## 12. Documentation Comments

Rust has a special comment style for **documentation** — `///` for documenting items and `//!` for module-level docs:

```rust
/// Calculates the area of a rectangle.
///
/// # Arguments
///
/// * `width` — the width of the rectangle (must be positive)
/// * `height` — the height of the rectangle (must be positive)
///
/// # Returns
///
/// The area as a `f64`.
///
/// # Examples
///
/// ```
/// let area = rectangle_area(5.0, 3.0);
/// assert_eq!(area, 15.0);
/// ```
///
/// # Panics
///
/// Panics if either `width` or `height` is negative.
fn rectangle_area(width: f64, height: f64) -> f64 {
    assert!(width >= 0.0 && height >= 0.0, "Dimensions must be non-negative");
    width * height
}

/// Checks whether a number is prime.
///
/// # Examples
///
/// ```
/// assert_eq!(is_prime(7), true);
/// assert_eq!(is_prime(4), false);
/// assert_eq!(is_prime(1), false);
/// ```
fn is_prime(n: u64) -> bool {
    if n < 2 { return false; }
    if n == 2 { return true; }
    if n % 2 == 0 { return false; }

    let mut i = 3;
    while i * i <= n {
        if n % i == 0 { return false; }
        i += 2;
    }
    true
}

fn main() {
    println!("{}", rectangle_area(5.0, 3.0)); // 15.0
    println!("{}", is_prime(17));             // true
    println!("{}", is_prime(18));             // false
}
```

> **`cargo doc --open`** generates beautiful HTML documentation from `///` comments — run it in any Rust project to see your docs rendered in a browser.

---

## 13. Common Patterns and Best Practices

### Pattern 1 — Guard clauses for early validation

```rust
fn process_score(score: i32) -> &'static str {
    // Handle invalid cases first — the "guard clause" pattern
    if score < 0   { return "Invalid: score cannot be negative"; }
    if score > 100 { return "Invalid: score cannot exceed 100"; }

    // Main logic — clean and focused
    match score {
        90..=100 => "A",
        80..=89  => "B",
        70..=79  => "C",
        60..=69  => "D",
        _        => "F",
    }
}
```

### Pattern 2 — Pure functions (no side effects)

```rust
// Pure function — same input always gives same output, no side effects
fn celsius_to_kelvin(celsius: f64) -> f64 {
    celsius + 273.15
}

// Impure — has a side effect (printing)
fn print_celsius_to_kelvin(celsius: f64) {
    println!("{:.2}K", celsius + 273.15);
}

// Best practice: separate computation from output
fn main() {
    let kelvin = celsius_to_kelvin(100.0); // compute
    println!("{kelvin}");                   // output
}
```

### Pattern 3 — Expressive, readable function chains

```rust
fn main() {
    let words = vec!["hello", "world", "rust", "is", "awesome"];

    // Chain of operations — each step transforms the data
    let result: Vec<String> = words
        .iter()
        .filter(|w| w.len() > 3)          // keep words longer than 3 chars
        .map(|w| w.to_uppercase())         // convert to uppercase
        .collect();                        // gather into a Vec

    println!("{:?}", result);
    // ["HELLO", "WORLD", "RUST", "AWESOME"]
}
```

### Pattern 4 — Functions that return Options

```rust
fn safe_divide(a: f64, b: f64) -> Option<f64> {
    if b == 0.0 {
        None        // signal failure without panicking
    } else {
        Some(a / b) // signal success with a value
    }
}

fn find_student(names: &[&str], target: &str) -> Option<usize> {
    names.iter().position(|&name| name == target)
}

fn main() {
    match safe_divide(10.0, 2.0) {
        Some(result) => println!("Result: {result}"), // Result: 5
        None         => println!("Division by zero"),
    }

    match safe_divide(10.0, 0.0) {
        Some(result) => println!("Result: {result}"),
        None         => println!("Division by zero"),  // this runs
    }

    let names = ["Alice", "Bob", "Charlie"];
    match find_student(&names, "Bob") {
        Some(i) => println!("Found at index {i}"), // Found at index 1
        None    => println!("Not found"),
    }
}
```

### Pattern 5 — Small, focused functions (Single Responsibility)

```rust
// ❌ Too much in one function — hard to test, understand, and reuse
fn process_student_data(name: &str, score: f64) {
    let grade = if score >= 90.0 { "A" }
        else if score >= 80.0 { "B" }
        else if score >= 70.0 { "C" }
        else { "F" };
    println!("Student: {name}, Score: {score:.1}, Grade: {grade}");
    // ... and 50 more lines
}

// ✅ Each function has one job — easy to test and compose
fn letter_grade(score: f64) -> &'static str {
    if score >= 90.0 { "A" }
    else if score >= 80.0 { "B" }
    else if score >= 70.0 { "C" }
    else { "F" }
}

fn format_report(name: &str, score: f64) -> String {
    let grade = letter_grade(score);
    format!("Student: {name}, Score: {score:.1}, Grade: {grade}")
}

fn main() {
    println!("{}", format_report("Alice", 92.5));
    println!("{}", format_report("Bob", 74.0));
}
```

---

## 14. Summary Cheat Sheet

```rust
// ══════════════════════════════════════════════════
// FUNCTION BASICS
// ══════════════════════════════════════════════════

// No parameters, no return value
fn greet() {
    println!("Hello!");
}

// Parameters (types required)
fn add(a: i32, b: i32) -> i32 {
    a + b          // implicit return — no semicolon
}

// Explicit return
fn absolute(n: i32) -> i32 {
    if n < 0 { return -n; } // early return
    n                        // implicit return for normal case
}

// Multiple return values via tuple
fn divmod(a: i32, b: i32) -> (i32, i32) {
    (a / b, a % b)
}
let (q, r) = divmod(17, 5); // destructure the result

// ══════════════════════════════════════════════════
// STATEMENTS VS EXPRESSIONS
// ══════════════════════════════════════════════════

let x = 5;              // statement (no value)
let y = x + 1;          // expression assigned to y (value: 6)

let z = {               // block expression
    let a = 3;
    a * a               // no semicolon → this is the block's value (9)
};                      // z = 9

// ══════════════════════════════════════════════════
// CLOSURES
// ══════════════════════════════════════════════════

let double = |x: i32| x * 2;           // single expression
let add    = |a, b| a + b;             // type inferred
let greet  = |name: &str| {            // multi-line
    println!("Hello, {name}!");
};

// Capturing environment
let factor = 5;
let multiply = |x| x * factor;         // captures factor
println!("{}", multiply(3));            // 15

// Closures with iterators
let nums = vec![1, 2, 3, 4, 5];
let doubled: Vec<i32> = nums.iter().map(|&x| x * 2).collect();
let evens:   Vec<&i32> = nums.iter().filter(|&&x| x % 2 == 0).collect();
let sum: i32 = nums.iter().sum();

// ══════════════════════════════════════════════════
// FUNCTION POINTERS
// ══════════════════════════════════════════════════

fn square(x: i32) -> i32 { x * x }

let f: fn(i32) -> i32 = square;        // function pointer
println!("{}", f(5));                   // 25

// ══════════════════════════════════════════════════
// DIVERGING FUNCTIONS
// ══════════════════════════════════════════════════

fn crash(msg: &str) -> ! {
    panic!("{msg}");
}

// Useful placeholders
fn not_done() -> i32 { todo!() }
fn never_called() -> i32 { unreachable!() }

// ══════════════════════════════════════════════════
// DOCUMENTATION
// ══════════════════════════════════════════════════

/// One-line summary of the function.
///
/// More detailed explanation here.
///
/// # Examples
/// ```
/// let result = my_fn(5);
/// assert_eq!(result, 10);
/// ```
fn my_fn(x: i32) -> i32 { x * 2 }
```

---

## 🔗 What's Next?

- **Lesson 05 — Control Flow** (`if`/`else`, `loop`, `while`, `for`, `match`)
- **Lesson 06 — Ownership** (how Rust manages memory — moves, borrows, lifetimes)
- **Lesson 07 — Structs** (custom types with named fields and methods)
- **Lesson 08 — Enums and Pattern Matching** (`Option`, `Result`, `match`)

---

## 📚 Further Reading

- [The Rust Book — Chapter 3.3: Functions](https://doc.rust-lang.org/book/ch03-03-how-functions-work.html)
- [The Rust Book — Chapter 13: Closures](https://doc.rust-lang.org/book/ch13-01-closures.html)
- [Rust by Example — Functions](https://doc.rust-lang.org/rust-by-example/fn.html)
- [Rust by Example — Closures](https://doc.rust-lang.org/rust-by-example/fn/closures.html)
- [Rust Reference — Functions](https://doc.rust-lang.org/reference/items/functions.html)

---

*Functions are the vocabulary of your program. Name them well. Keep them small. Make them do one thing brilliantly. 🦀*
