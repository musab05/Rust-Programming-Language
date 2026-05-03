# 📘 Lesson 41 — Generics in Structs & Enums (T4)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T4 · Category: 🔷 Traits  
> **Previous:** [Lesson 40 — Generics in Functions](../lesson_40_generics/lesson_40_generics.md)  
> **Next:** [Lesson 42 — impl Trait & dyn Trait](../lesson_42_impl_dyn_trait/lesson_42_impl_dyn_trait.md)  
> **Practice:** [Questions](./lesson_41_questions.md) · [Answers](./lesson_41_answers.md)  
> **Practice Task:** Generic Pair\<T\> with cmp_display() using Display + PartialOrd

---

## Table of Contents

1. [Generic Structs](#1-generic-structs)
2. [Multiple Type Parameters in Structs](#2-multiple-type-parameters-in-structs)
3. [Implementing Methods on Generic Structs](#3-implementing-methods-on-generic-structs)
4. [Generic Enums](#4-generic-enums)
5. [Option\<T\> and Result\<T,E\> — Generics in the Standard Library](#5-optiont-and-resultte--generics-in-the-standard-library)
6. [Methods That Introduce New Type Parameters](#6-methods-that-introduce-new-type-parameters)
7. [Default Type Parameters](#7-default-type-parameters)
8. [Real-World Patterns](#8-real-world-patterns)
9. [Summary Cheat Sheet](#9-summary-cheat-sheet)

---

## 1. Generic Structs

A struct can hold any type via a type parameter:

```rust
#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

fn main() {
    let integer_point = Point { x: 5, y: 10 };
    let float_point = Point { x: 1.0, y: 4.5 };
    let char_point = Point { x: 'a', y: 'z' };

    println!("{:?}", integer_point);  // Point { x: 5, y: 10 }
    println!("{:?}", float_point);    // Point { x: 1.0, y: 4.5 }
    println!("{:?}", char_point);     // Point { x: 'a', y: 'z' }

    // ❌ Won't compile — x and y must be the same type T
    // let bad = Point { x: 5, y: 4.0 };
}
```

---

## 2. Multiple Type Parameters in Structs

Use multiple type parameters when fields can differ:

```rust
#[derive(Debug)]
struct Point<T, U> {
    x: T,
    y: U,
}

#[derive(Debug)]
struct KeyValue<K, V> {
    key: K,
    value: V,
}

fn main() {
    let mixed = Point { x: 5, y: 4.0 };  // Point<i32, f64>
    println!("{:?}", mixed);

    let entry = KeyValue {
        key: "name".to_string(),
        value: 42,
    };
    println!("{:?}", entry);

    // Both same type still works
    let same = Point { x: 1, y: 2 };  // Point<i32, i32>
    println!("{:?}", same);
}
```

---

## 3. Implementing Methods on Generic Structs

```rust
#[derive(Debug)]
struct Point<T> {
    x: T,
    y: T,
}

// Methods for ALL Point<T>
impl<T> Point<T> {
    fn new(x: T, y: T) -> Self {
        Point { x, y }
    }

    fn x(&self) -> &T {
        &self.x
    }

    fn y(&self) -> &T {
        &self.y
    }

    fn into_tuple(self) -> (T, T) {
        (self.x, self.y)
    }
}

// Methods ONLY for Point<f64>
impl Point<f64> {
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }

    fn distance_to(&self, other: &Point<f64>) -> f64 {
        let dx = self.x - other.x;
        let dy = self.y - other.y;
        (dx.powi(2) + dy.powi(2)).sqrt()
    }
}

// Methods when T has specific bounds
impl<T: std::fmt::Display> Point<T> {
    fn display(&self) {
        println!("({}, {})", self.x, self.y);
    }
}

impl<T: PartialOrd + std::fmt::Display> Point<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("x={} is >= y={}", self.x, self.y);
        } else {
            println!("y={} is > x={}", self.y, self.x);
        }
    }
}

fn main() {
    let p1 = Point::new(3.0, 4.0);
    println!("Distance from origin: {}", p1.distance_from_origin());  // 5.0

    let p2 = Point::new(0.0, 0.0);
    println!("Distance p1→p2: {}", p1.distance_to(&p2));  // 5.0

    let p3 = Point::new(10, 20);
    p3.display();        // (10, 20)
    p3.cmp_display();    // y=20 is > x=10

    // p3.distance_from_origin();  // ❌ only available on Point<f64>

    let (x, y) = p3.into_tuple();
    println!("Tuple: ({x}, {y})");
}
```

---

## 4. Generic Enums

```rust
// You've already used these!
// enum Option<T> {
//     Some(T),
//     None,
// }

// enum Result<T, E> {
//     Ok(T),
//     Err(E),
// }

// Custom generic enum
#[derive(Debug)]
enum Container<T> {
    Empty,
    Single(T),
    Pair(T, T),
}

impl<T: std::fmt::Display> Container<T> {
    fn describe(&self) {
        match self {
            Container::Empty => println!("Empty container"),
            Container::Single(v) => println!("Contains: {v}"),
            Container::Pair(a, b) => println!("Contains pair: {a}, {b}"),
        }
    }
}

impl<T: Clone> Container<T> {
    fn to_vec(&self) -> Vec<T> {
        match self {
            Container::Empty => vec![],
            Container::Single(v) => vec![v.clone()],
            Container::Pair(a, b) => vec![a.clone(), b.clone()],
        }
    }
}

fn main() {
    let c1: Container<i32> = Container::Empty;
    let c2 = Container::Single("hello");
    let c3 = Container::Pair(3.14, 2.71);

    c1.describe();  // Empty container
    c2.describe();  // Contains: hello
    c3.describe();  // Contains pair: 3.14, 2.71

    println!("{:?}", c3.to_vec());  // [3.14, 2.71]
}
```

### Generic enum for a linked list:

```rust
#[derive(Debug)]
enum List<T> {
    Cons(T, Box<List<T>>),
    Nil,
}

impl<T> List<T> {
    fn new() -> Self {
        List::Nil
    }

    fn prepend(self, value: T) -> Self {
        List::Cons(value, Box::new(self))
    }
}

impl<T: std::fmt::Display> List<T> {
    fn print(&self) {
        match self {
            List::Cons(val, next) => {
                print!("{val} → ");
                next.print();
            }
            List::Nil => println!("∅"),
        }
    }
}

fn main() {
    let list = List::new()
        .prepend(3)
        .prepend(2)
        .prepend(1);

    list.print();  // 1 → 2 → 3 → ∅
}
```

---

## 5. Option\<T\> and Result\<T,E\> — Generics in the Standard Library

These are the most important generic enums in Rust:

```rust
fn main() {
    // Option<T> — presence or absence
    let some_num: Option<i32> = Some(42);
    let no_num: Option<i32> = None;

    println!("Double: {:?}", some_num.map(|x| x * 2));  // Some(84)
    println!("Or: {}", no_num.unwrap_or(0));              // 0

    // Result<T, E> — success or failure
    let ok: Result<i32, String> = Ok(42);
    let err: Result<i32, String> = Err("failed".into());

    println!("Ok map: {:?}", ok.map(|x| x + 1));  // Ok(43)
    println!("Err or: {}", err.unwrap_or(0));       // 0

    // Vec<T> — generic collection
    let ints: Vec<i32> = vec![1, 2, 3];
    let strings: Vec<String> = vec!["a".into(), "b".into()];
    println!("{:?}, {:?}", ints, strings);

    // HashMap<K, V> — generic key-value
    use std::collections::HashMap;
    let mut scores: HashMap<String, f64> = HashMap::new();
    scores.insert("Alice".into(), 95.5);
}
```

---

## 6. Methods That Introduce New Type Parameters

A method can have its own type parameters, separate from the struct's:

```rust
#[derive(Debug)]
struct Point<T, U> {
    x: T,
    y: U,
}

impl<T, U> Point<T, U> {
    fn new(x: T, y: U) -> Self {
        Point { x, y }
    }

    // V and W are NEW type parameters, only for this method
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point {
            x: self.x,    // from self (type T)
            y: other.y,   // from other (type W)
        }
    }
}

fn main() {
    let p1 = Point::new(5, 10.4);        // Point<i32, f64>
    let p2 = Point::new("Hello", 'c');    // Point<&str, char>

    let p3 = p1.mixup(p2);               // Point<i32, char>
    println!("{:?}", p3);                 // Point { x: 5, y: 'c' }
}
```

---

## 7. Default Type Parameters

Type parameters can have defaults:

```rust
use std::marker::PhantomData;

// Default type parameter: E defaults to String
struct MyResult<T, E = String> {
    value: Result<T, E>,
}

impl<T: std::fmt::Debug, E: std::fmt::Debug> MyResult<T, E> {
    fn new_ok(val: T) -> Self {
        MyResult { value: Ok(val) }
    }

    fn new_err(err: E) -> Self {
        MyResult { value: Err(err) }
    }

    fn show(&self) {
        println!("{:?}", self.value);
    }
}

fn main() {
    // Uses default E = String
    let r1: MyResult<i32> = MyResult::new_ok(42);
    r1.show();  // Ok(42)

    let r2: MyResult<i32> = MyResult::new_err("something broke".into());
    r2.show();  // Err("something broke")

    // Override the default
    let r3: MyResult<i32, std::io::Error> = MyResult::new_ok(100);
    r3.show();  // Ok(100)
}
```

You've seen this with `HashMap`:
```rust
// HashMap<K, V, S = RandomState>
// The third parameter S (hasher) defaults to RandomState
```

---

## 8. Real-World Patterns

### Generic wrapper (newtype):

```rust
#[derive(Debug, Clone, PartialEq)]
struct Validated<T> {
    value: T,
}

impl<T> Validated<T> {
    fn into_inner(self) -> T {
        self.value
    }

    fn as_ref(&self) -> &T {
        &self.value
    }
}

// Specific validation for strings
impl Validated<String> {
    fn new_email(email: String) -> Result<Self, String> {
        if email.contains('@') && email.contains('.') {
            Ok(Validated { value: email })
        } else {
            Err(format!("Invalid email: {email}"))
        }
    }
}

// Specific validation for numbers
impl Validated<u16> {
    fn new_port(port: u16) -> Result<Self, String> {
        if port >= 1024 {
            Ok(Validated { value: port })
        } else {
            Err(format!("Port {port} is reserved (must be >= 1024)"))
        }
    }
}

fn main() {
    let email = Validated::new_email("user@example.com".into()).unwrap();
    let port = Validated::new_port(8080).unwrap();

    println!("Email: {:?}", email.as_ref());
    println!("Port: {}", port.into_inner());
}
```

### Generic Pair with comparison:

```rust
use std::fmt::Display;

#[derive(Debug)]
struct Pair<T> {
    first: T,
    second: T,
}

impl<T> Pair<T> {
    fn new(first: T, second: T) -> Self {
        Pair { first, second }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.first >= self.second {
            println!("Larger: {}", self.first);
        } else {
            println!("Larger: {}", self.second);
        }
    }

    fn larger(&self) -> &T {
        if self.first >= self.second { &self.first } else { &self.second }
    }

    fn smaller(&self) -> &T {
        if self.first <= self.second { &self.first } else { &self.second }
    }
}

fn main() {
    let p = Pair::new(42, 17);
    p.cmp_display();                           // Larger: 42
    println!("Larger: {}", p.larger());        // 42
    println!("Smaller: {}", p.smaller());      // 17

    let words = Pair::new("banana", "apple");
    words.cmp_display();                       // Larger: banana
}
```

---

## 9. Summary Cheat Sheet

```
GENERIC STRUCTS
────────────────────────────────────────────────────────────
struct Point<T> { x: T, y: T }            one type parameter
struct Pair<T, U> { first: T, second: U }  two type parameters

GENERIC ENUMS
────────────────────────────────────────────────────────────
enum Option<T> { Some(T), None }
enum Result<T, E> { Ok(T), Err(E) }
enum List<T> { Cons(T, Box<List<T>>), Nil }

IMPLEMENTING METHODS
────────────────────────────────────────────────────────────
impl<T> Point<T> { ... }              for all T
impl Point<f64> { ... }               only for f64
impl<T: Display> Point<T> { ... }     when T: Display

METHOD'S OWN TYPE PARAMETERS
────────────────────────────────────────────────────────────
impl<T, U> Point<T, U> {
    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W>
}

DEFAULT TYPE PARAMETERS
────────────────────────────────────────────────────────────
struct Foo<T, E = String> { ... }      E defaults to String

STANDARD LIBRARY GENERICS
────────────────────────────────────────────────────────────
Option<T>, Result<T, E>, Vec<T>
HashMap<K, V>, HashSet<T>
Box<T>, Rc<T>, Arc<T>
```

---

## What's Next?

**Lesson 42 — impl Trait & dyn Trait** — Static vs dynamic dispatch. Learn when to use `impl Trait` (monomorphization) vs `dyn Trait` (vtable dispatch) and build polymorphic collections.

## Further Reading
- [The Rust Book — Ch 10.1: Generic Data Types](https://doc.rust-lang.org/book/ch10-01-syntax.html)
- [Rust by Example — Generics](https://doc.rust-lang.org/rust-by-example/generics.html)

---

*Generic structs and enums: data structures that work with any type! 🦀*
