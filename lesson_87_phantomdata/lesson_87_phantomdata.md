# 📘 Lesson 87 — PhantomData (AT5)

> **Series:** Rust From Zero · Advanced Level (Gap Fill)  
> **Roadmap ID:** AT5 · Category: 🔷 Advanced Types  
> **Previous:** [Lesson 86 — Dynamically Sized Types](../lesson_86_dst/lesson_86_dst.md)  
> **Next:** [Lesson 88 — Benchmarking with Criterion](../lesson_88_benchmarking/lesson_88_benchmarking.md)  
> **Practice:** [Questions](./lesson_87_questions.md) · [Answers](./lesson_87_answers.md)  
> **Practice Task:** Type-safe units using PhantomData\<Unit\>

---

## Table of Contents

1. [What Is PhantomData?](#1-what-is-phantomdata)
2. [Unused Type Parameters](#2-unused-type-parameters)
3. [Type-Safe Units](#3-type-safe-units)
4. [Type-State Pattern](#4-type-state-pattern)
5. [PhantomData and Lifetimes](#5-phantomdata-and-lifetimes)
6. [PhantomData and Drop Check](#6-phantomdata-and-drop-check)
7. [Common Patterns](#7-common-patterns)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. What Is PhantomData?

`PhantomData<T>` is a **zero-sized** marker that tells the compiler "I logically contain a `T`" — even though there's no actual `T` stored:

```rust
use std::marker::PhantomData;

struct MyStruct<T> {
    id: u64,
    _marker: PhantomData<T>,  // zero bytes! just a marker
}

fn main() {
    println!("PhantomData size: {}", std::mem::size_of::<PhantomData<String>>());  // 0
    println!("MyStruct<i32>:    {}", std::mem::size_of::<MyStruct<i32>>());        // 8
    println!("MyStruct<String>: {}", std::mem::size_of::<MyStruct<String>>());     // 8
    // PhantomData adds ZERO bytes to the struct
}
```

---

## 2. Unused Type Parameters

Without PhantomData, unused type parameters cause errors:

```rust
use std::marker::PhantomData;

// ❌ Won't compile — T is unused
// struct Id<T> { value: u64 }

// ✅ PhantomData "uses" T
struct Id<T> {
    value: u64,
    _type: PhantomData<T>,
}

struct User;
struct Order;

impl<T> Id<T> {
    fn new(value: u64) -> Self {
        Id { value, _type: PhantomData }
    }
}

fn find_user(id: Id<User>) -> String {
    format!("User #{}", id.value)
}

fn find_order(id: Id<Order>) -> String {
    format!("Order #{}", id.value)
}

fn main() {
    let user_id: Id<User> = Id::new(42);
    let order_id: Id<Order> = Id::new(42);

    println!("{}", find_user(user_id));
    println!("{}", find_order(order_id));

    // find_user(order_id);  // ❌ compile error! Id<Order> ≠ Id<User>
}
```

---

## 3. Type-Safe Units

Prevent unit confusion at compile time:

```rust
use std::marker::PhantomData;
use std::ops::{Add, Mul};

struct Meters;
struct Seconds;
struct MetersPerSecond;

#[derive(Debug, Clone, Copy)]
struct Quantity<Unit> {
    value: f64,
    _unit: PhantomData<Unit>,
}

impl<U> Quantity<U> {
    fn new(value: f64) -> Self {
        Quantity { value, _unit: PhantomData }
    }
}

impl<U> Add for Quantity<U> {
    type Output = Self;
    fn add(self, rhs: Self) -> Self {
        Quantity::new(self.value + rhs.value)
    }
}

impl<U> std::fmt::Display for Quantity<U> {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{:.2}", self.value)
    }
}

// Speed = distance / time
fn speed(distance: Quantity<Meters>, time: Quantity<Seconds>) -> Quantity<MetersPerSecond> {
    Quantity::new(distance.value / time.value)
}

fn main() {
    let d1 = Quantity::<Meters>::new(100.0);
    let d2 = Quantity::<Meters>::new(50.0);
    let t = Quantity::<Seconds>::new(9.58);

    let total_distance = d1 + d2;  // ✅ same unit
    println!("Distance: {total_distance}m");

    let v = speed(d1, t);
    println!("Speed: {v} m/s");

    // let wrong = d1 + t;  // ❌ Quantity<Meters> + Quantity<Seconds> — type error!
}
```

---

## 4. Type-State Pattern

Use PhantomData to enforce state transitions at compile time:

```rust
use std::marker::PhantomData;

// States (zero-sized types)
struct Draft;
struct Review;
struct Published;

struct Article<State> {
    title: String,
    content: String,
    _state: PhantomData<State>,
}

impl Article<Draft> {
    fn new(title: &str) -> Self {
        Article { title: title.into(), content: String::new(), _state: PhantomData }
    }

    fn write(&mut self, text: &str) {
        self.content.push_str(text);
    }

    fn submit(self) -> Article<Review> {
        println!("📝 '{}' submitted for review", self.title);
        Article { title: self.title, content: self.content, _state: PhantomData }
    }
}

impl Article<Review> {
    fn approve(self) -> Article<Published> {
        println!("✅ '{}' approved!", self.title);
        Article { title: self.title, content: self.content, _state: PhantomData }
    }

    fn reject(self) -> Article<Draft> {
        println!("❌ '{}' rejected, back to draft", self.title);
        Article { title: self.title, content: self.content, _state: PhantomData }
    }
}

impl Article<Published> {
    fn read(&self) -> &str {
        &self.content
    }
}

fn main() {
    let mut article = Article::<Draft>::new("Rust PhantomData");
    article.write("PhantomData is a zero-sized marker...");

    // article.read();  // ❌ can't read a draft!

    let in_review = article.submit();
    // in_review.write("more");  // ❌ can't edit during review!

    let published = in_review.approve();
    println!("📖 Reading: {}", published.read());  // ✅

    // published.submit();  // ❌ can't submit a published article!
}
```

---

## 5. PhantomData and Lifetimes

PhantomData can bind a lifetime to a struct:

```rust
use std::marker::PhantomData;

struct Iter<'a, T> {
    ptr: *const T,
    end: *const T,
    _lifetime: PhantomData<&'a T>,  // tells compiler: we borrow 'a
}

impl<'a, T> Iter<'a, T> {
    fn new(slice: &'a [T]) -> Self {
        let ptr = slice.as_ptr();
        let end = unsafe { ptr.add(slice.len()) };
        Iter { ptr, end, _lifetime: PhantomData }
    }
}

impl<'a, T> Iterator for Iter<'a, T> {
    type Item = &'a T;
    fn next(&mut self) -> Option<Self::Item> {
        if self.ptr == self.end { return None; }
        let item = unsafe { &*self.ptr };
        self.ptr = unsafe { self.ptr.add(1) };
        Some(item)
    }
}

fn main() {
    let data = vec![10, 20, 30];
    let iter = Iter::new(&data);
    for val in iter {
        println!("{val}");
    }
}
```

---

## 6. PhantomData and Drop Check

`PhantomData<T>` tells the compiler to act as if the struct owns a `T` for drop-check purposes:

```rust
use std::marker::PhantomData;

struct MyBox<T> {
    ptr: *mut T,
    _owns: PhantomData<T>,  // "I own a T" — affects drop order
}

impl<T> MyBox<T> {
    fn new(val: T) -> Self {
        let ptr = Box::into_raw(Box::new(val));
        MyBox { ptr, _owns: PhantomData }
    }
}

impl<T> Drop for MyBox<T> {
    fn drop(&mut self) {
        unsafe { drop(Box::from_raw(self.ptr)); }
    }
}

fn main() {
    let b = MyBox::new(String::from("hello"));
    // Properly dropped — PhantomData ensures correct drop checking
}
```

---

## 7. Common Patterns

| Pattern | PhantomData Usage |
|---|---|
| Typed IDs | `struct Id<T> { val: u64, _: PhantomData<T> }` |
| Unit types | `struct Quantity<U> { val: f64, _: PhantomData<U> }` |
| Type-state | `struct Conn<State> { ..., _: PhantomData<State> }` |
| Lifetime binding | `struct Iter<'a> { ptr: *const u8, _: PhantomData<&'a u8> }` |
| Ownership | `struct MyBox<T> { ptr: *mut T, _: PhantomData<T> }` |

---

## 8. Summary Cheat Sheet

```
PHANTOMDATA
────────────────────────────────────────────────────────────
std::marker::PhantomData<T>     zero-sized marker

PURPOSE
────────────────────────────────────────────────────────────
1. Use unused type parameters     Id<T> { val: u64, _: PhantomData<T> }
2. Type-safe units                Quantity<Meters> vs Quantity<Seconds>
3. Type-state pattern             Connection<Connected> vs <Disconnected>
4. Bind lifetimes                 PhantomData<&'a T> → struct borrows 'a
5. Drop check                    PhantomData<T> → "I own a T"

KEY FACTS
────────────────────────────────────────────────────────────
Size: 0 bytes (always)
No runtime cost
Purely compile-time construct
Convention: field name _marker or _phantom
```

---

## What's Next?

**Lesson 88 — Benchmarking with Criterion** — Statistical benchmarks for comparing performance.

## Further Reading
- [std::marker::PhantomData](https://doc.rust-lang.org/std/marker/struct.PhantomData.html)
- [Nomicon — PhantomData](https://doc.rust-lang.org/nomicon/phantom-data.html)

---

*PhantomData: invisible to runtime, indispensable to the type system! 🦀*
