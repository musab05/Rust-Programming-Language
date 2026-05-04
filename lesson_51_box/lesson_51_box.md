# 📘 Lesson 51 — Smart Pointers: Box (SP1)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** SP1 · Category: 📌 Smart Pointers  
> **Previous:** [Lesson 50 — Higher-Order Functions](../lesson_50_higher_order/lesson_50_higher_order.md)  
> **Next:** [Lesson 52 — Rc & Arc](../lesson_52_rc_arc/lesson_52_rc_arc.md)  
> **Practice:** [Questions](./lesson_51_questions.md) · [Answers](./lesson_51_answers.md)  
> **Practice Task:** Build a recursive binary tree using Box

---

## Table of Contents

1. [What Are Smart Pointers?](#1-what-are-smart-pointers)
2. [Box\<T\> — Heap Allocation](#2-boxt--heap-allocation)
3. [When to Use Box](#3-when-to-use-box)
4. [Recursive Types with Box](#4-recursive-types-with-box)
5. [Box and Trait Objects](#5-box-and-trait-objects)
6. [Deref and DerefMut](#6-deref-and-derefmut)
7. [Drop — Cleanup on Scope Exit](#7-drop--cleanup-on-scope-exit)
8. [Box vs Stack Allocation](#8-box-vs-stack-allocation)
9. [Real-World Example: Binary Tree](#9-real-world-example-binary-tree)
10. [Summary Cheat Sheet](#10-summary-cheat-sheet)

---

## 1. What Are Smart Pointers?

Smart pointers are structs that act like pointers but carry additional metadata and capabilities. They implement `Deref` and `Drop` traits.

| Smart Pointer | Purpose |
|---|---|
| `Box<T>` | Heap-allocated value, single owner |
| `Rc<T>` | Reference-counted shared ownership (single-threaded) |
| `Arc<T>` | Atomic reference-counted (multi-threaded) |
| `RefCell<T>` | Interior mutability (runtime borrow checking) |
| `Cow<T>` | Clone-on-write |

Regular references (`&T`) borrow data. Smart pointers **own** data.

---

## 2. Box\<T\> — Heap Allocation

`Box<T>` allocates data on the heap instead of the stack:

```rust
fn main() {
    // Stack allocation (default)
    let x = 5;  // lives on the stack

    // Heap allocation via Box
    let y = Box::new(5);  // 5 is on the heap, y is a pointer on the stack

    // Use it like a regular value (via Deref)
    println!("y = {y}");           // 5
    println!("y + 1 = {}", *y + 1); // 6

    // y is dropped here → heap memory freed
}
```

### Memory layout:

```
Stack                    Heap
┌──────────┐            ┌──────┐
│ x: 5     │            │  5   │ ← Box allocates here
│ y: ptr ──┼───────────→│      │
└──────────┘            └──────┘
```

---

## 3. When to Use Box

### 1. Types with unknown size at compile time:

```rust
// Trait objects — size unknown at compile time
fn make_greeting(formal: bool) -> Box<dyn std::fmt::Display> {
    if formal {
        Box::new(String::from("Good evening"))
    } else {
        Box::new("Hey!")  // &str
    }
}

fn main() {
    println!("{}", make_greeting(true));
    println!("{}", make_greeting(false));
}
```

### 2. Large data you don't want to copy:

```rust
fn main() {
    // Without Box — copies 1000 elements on assignment
    let big_array = [0u8; 1_000_000];  // 1MB on stack (might overflow!)

    // With Box — only copies the pointer (8 bytes)
    let big_boxed = Box::new([0u8; 1_000_000]);  // 1MB on heap, safe
    println!("Size on stack: {} bytes", std::mem::size_of_val(&big_boxed)); // 8
}
```

### 3. Transferring ownership without copying:

```rust
fn process(data: Box<[u8; 1_000_000]>) {
    println!("Processing {} bytes", data.len());
    // Only the 8-byte pointer was moved, not 1MB of data
}
```

---

## 4. Recursive Types with Box

Without `Box`, recursive types have infinite size:

```rust
// ❌ Won't compile — infinite size
// enum List {
//     Cons(i32, List),  // List contains List contains List...
//     Nil,
// }

// ✅ Box breaks the infinite recursion
enum List {
    Cons(i32, Box<List>),  // fixed size: i32 + pointer
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));

    // Print the list
    fn print_list(list: &List) {
        match list {
            Cons(val, next) => {
                print!("{val} → ");
                print_list(next);
            }
            Nil => println!("∅"),
        }
    }

    print_list(&list);  // 1 → 2 → 3 → ∅
}
```

### Why Box fixes this:

```
Without Box:
List = i32 + List = i32 + i32 + List = ...  (infinite!)

With Box:
List = i32 + Box<List> = i32 + pointer  (fixed size!)
```

---

## 5. Box and Trait Objects

`Box<dyn Trait>` is the most common way to store trait objects:

```rust
trait Animal {
    fn speak(&self) -> &str;
    fn name(&self) -> &str;
}

struct Dog { name: String }
struct Cat { name: String }

impl Animal for Dog {
    fn speak(&self) -> &str { "Woof!" }
    fn name(&self) -> &str { &self.name }
}

impl Animal for Cat {
    fn speak(&self) -> &str { "Meow!" }
    fn name(&self) -> &str { &self.name }
}

fn main() {
    let animals: Vec<Box<dyn Animal>> = vec![
        Box::new(Dog { name: "Rex".into() }),
        Box::new(Cat { name: "Whiskers".into() }),
        Box::new(Dog { name: "Buddy".into() }),
    ];

    for animal in &animals {
        println!("{}: {}", animal.name(), animal.speak());
    }
}
```

---

## 6. Deref and DerefMut

`Box<T>` implements `Deref<Target = T>`, so you can use it like a `&T`:

```rust
fn main() {
    let boxed = Box::new(String::from("hello"));

    // Automatic dereferencing (Deref coercion)
    let len = boxed.len();  // calls String::len via Deref
    println!("Length: {len}");

    // Explicit dereference
    let s: &str = &*boxed;
    println!("{s}");

    // Works with function parameters too
    fn print_str(s: &str) {
        println!("{s}");
    }
    print_str(&boxed);  // Box<String> → &String → &str via Deref chain
}
```

### Custom Deref:

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> { MyBox(x) }
}

impl<T> Deref for MyBox<T> {
    type Target = T;
    fn deref(&self) -> &T { &self.0 }
}

fn main() {
    let x = MyBox::new(42);
    println!("{}", *x);  // 42 — uses our Deref implementation
}
```

---

## 7. Drop — Cleanup on Scope Exit

`Box<T>` implements `Drop` — memory is freed when it goes out of scope:

```rust
struct Noisy {
    name: String,
}

impl Drop for Noisy {
    fn drop(&mut self) {
        println!("  Dropping '{}'", self.name);
    }
}

fn main() {
    println!("Creating boxes...");
    let a = Box::new(Noisy { name: "Alpha".into() });
    let b = Box::new(Noisy { name: "Beta".into() });

    println!("Using them...");
    println!("  a = {}", a.name);
    println!("  b = {}", b.name);

    println!("About to drop early...");
    drop(a);  // explicitly drop a

    println!("End of main...");
    // b is automatically dropped here
}
// Output:
// Creating boxes...
// Using them...
//   a = Alpha
//   b = Beta
// About to drop early...
//   Dropping 'Alpha'
// End of main...
//   Dropping 'Beta'
```

---

## 8. Box vs Stack Allocation

```rust
fn main() {
    // Stack: fast allocation, limited size, copied on move (if Copy)
    let stack_val = 42;

    // Box (heap): slower allocation, unlimited size, pointer moved
    let heap_val = Box::new(42);

    // Both work the same way in use
    println!("{stack_val} == {heap_val}: {}", stack_val == *heap_val);
}
```

| | Stack | Box (Heap) |
|---|---|---|
| Speed | Fastest | Slight overhead |
| Size limit | ~1-8 MB (thread stack) | Available RAM |
| Ownership | Moved or copied | Pointer moved (8 bytes) |
| Use case | Small, short-lived | Large, recursive, trait objects |

---

## 9. Real-World Example: Binary Tree

The roadmap practice task:

```rust
use std::fmt;

#[derive(Debug)]
enum Tree<T> {
    Leaf(T),
    Node {
        value: T,
        left: Box<Tree<T>>,
        right: Box<Tree<T>>,
    },
    Empty,
}

impl<T: fmt::Display + PartialOrd> Tree<T> {
    fn new() -> Self { Tree::Empty }

    fn leaf(value: T) -> Self { Tree::Leaf(value) }

    fn node(value: T, left: Tree<T>, right: Tree<T>) -> Self {
        Tree::Node {
            value,
            left: Box::new(left),
            right: Box::new(right),
        }
    }

    fn insert(self, new_val: T) -> Self {
        match self {
            Tree::Empty => Tree::Leaf(new_val),
            Tree::Leaf(val) => {
                if new_val < val {
                    Tree::Node {
                        value: val,
                        left: Box::new(Tree::Leaf(new_val)),
                        right: Box::new(Tree::Empty),
                    }
                } else {
                    Tree::Node {
                        value: val,
                        left: Box::new(Tree::Empty),
                        right: Box::new(Tree::Leaf(new_val)),
                    }
                }
            }
            Tree::Node { value, left, right } => {
                if new_val < value {
                    Tree::Node {
                        value,
                        left: Box::new(left.insert(new_val)),
                        right,
                    }
                } else {
                    Tree::Node {
                        value,
                        left,
                        right: Box::new(right.insert(new_val)),
                    }
                }
            }
        }
    }

    fn contains(&self, target: &T) -> bool {
        match self {
            Tree::Empty => false,
            Tree::Leaf(v) => v == target,
            Tree::Node { value, left, right } => {
                if target == value { true }
                else if target < value { left.contains(target) }
                else { right.contains(target) }
            }
        }
    }

    fn print_inorder(&self) {
        match self {
            Tree::Empty => {}
            Tree::Leaf(v) => print!("{v} "),
            Tree::Node { value, left, right } => {
                left.print_inorder();
                print!("{value} ");
                right.print_inorder();
            }
        }
    }
}

fn main() {
    let tree = Tree::new()
        .insert(5)
        .insert(3)
        .insert(7)
        .insert(1)
        .insert(4)
        .insert(6)
        .insert(8);

    print!("In-order: ");
    tree.print_inorder();  // 1 3 4 5 6 7 8
    println!();

    println!("Contains 4: {}", tree.contains(&4));  // true
    println!("Contains 9: {}", tree.contains(&9));  // false
}
```

---

## 10. Summary Cheat Sheet

```
BOX<T> BASICS
────────────────────────────────────────────────────────────
Box::new(value)          allocate on heap
*boxed                   dereference to access value
drop(boxed)              free heap memory early

WHEN TO USE
────────────────────────────────────────────────────────────
Recursive types          enum List { Cons(T, Box<List>), Nil }
Trait objects             Box<dyn Trait>
Large data               avoid stack overflow
Ownership transfer       move pointer, not data

DEREF COERCION
────────────────────────────────────────────────────────────
Box<String> → &String → &str    automatic chain
box.method()                    calls T::method via Deref

DROP
────────────────────────────────────────────────────────────
Automatic when Box goes out of scope
drop(box) for early cleanup

MEMORY
────────────────────────────────────────────────────────────
Stack: Box pointer (8 bytes)
Heap:  the actual T data
```

---

## What's Next?

**Lesson 52 — Rc & Arc** — Shared ownership with reference counting. Learn when multiple owners need to share data, and how `Arc` extends this to threads.

## Further Reading
- [The Rust Book — Ch 15.1: Box](https://doc.rust-lang.org/book/ch15-01-box.html)
- [std::boxed::Box](https://doc.rust-lang.org/std/boxed/struct.Box.html)

---

*Box: putting things on the heap, one allocation at a time! 🦀*
