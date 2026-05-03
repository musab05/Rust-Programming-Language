# 📘 Lesson 45 — Associated Types (T8)

> **Series:** Rust From Zero · Intermediate Level  
> **Roadmap ID:** T8 · Category: 🔷 Traits  
> **Previous:** [Lesson 44 — Operator Overloading](../lesson_44_operator_overloading/lesson_44_operator_overloading.md)  
> **Next:** [Lesson 46 — Workspaces](../lesson_46_workspaces/lesson_46_workspaces.md)  
> **Practice:** [Questions](./lesson_45_questions.md) · [Answers](./lesson_45_answers.md)  
> **Practice Task:** Build a Graph trait with associated Node and Edge types

---

## Table of Contents

1. [What Are Associated Types?](#1-what-are-associated-types)
2. [Associated Types vs Generics](#2-associated-types-vs-generics)
3. [Iterator — The Classic Example](#3-iterator--the-classic-example)
4. [Defining Traits with Associated Types](#4-defining-traits-with-associated-types)
5. [Associated Constants](#5-associated-constants)
6. [Fully Qualified Syntax](#6-fully-qualified-syntax)
7. [Real-World Example: Graph Trait](#7-real-world-example-graph-trait)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. What Are Associated Types?

An **associated type** is a type placeholder defined inside a trait. The implementor chooses the concrete type:

```rust
trait Iterator {
    type Item;  // associated type — implementor decides what this is
    fn next(&mut self) -> Option<Self::Item>;
}
```

When you implement the trait, you specify what `Item` is:

```rust
struct Counter { current: u32, max: u32 }

impl Iterator for Counter {
    type Item = u32;  // Item is u32 for Counter

    fn next(&mut self) -> Option<u32> {
        if self.current < self.max {
            self.current += 1;
            Some(self.current)
        } else {
            None
        }
    }
}
```

---

## 2. Associated Types vs Generics

### With generics — multiple implementations possible:

```rust
// A type could implement this for many different T values
trait ConvertTo<T> {
    fn convert(&self) -> T;
}

struct Num(i32);

impl ConvertTo<f64> for Num {
    fn convert(&self) -> f64 { self.0 as f64 }
}

impl ConvertTo<String> for Num {
    fn convert(&self) -> String { self.0.to_string() }
}

fn main() {
    let n = Num(42);
    let f: f64 = n.convert();
    let s: String = n.convert();
    println!("{f}, {s}");
}
```

### With associated types — ONE implementation per type:

```rust
trait Iterator {
    type Item;  // each type has exactly ONE Item type
    fn next(&mut self) -> Option<Self::Item>;
}

// Counter can only iterate over ONE type (u32)
// You can't implement Iterator twice for Counter with different Items
```

### When to use which?

| Use | When |
|---|---|
| **Associated types** | Each type has ONE logical mapping (Iterator → one Item type) |
| **Generics** | A type can have MULTIPLE implementations (ConvertTo → many target types) |

---

## 3. Iterator — The Classic Example

```rust
struct Fibonacci { a: u64, b: u64 }

impl Fibonacci {
    fn new() -> Self { Fibonacci { a: 0, b: 1 } }
}

impl Iterator for Fibonacci {
    type Item = u64;  // this Iterator produces u64 values

    fn next(&mut self) -> Option<u64> {
        let result = self.a;
        let next = self.a + self.b;
        self.a = self.b;
        self.b = next;
        Some(result)
    }
}

fn main() {
    let fibs: Vec<u64> = Fibonacci::new().take(8).collect();
    println!("{:?}", fibs);  // [0, 1, 1, 2, 3, 5, 8, 13]
}
```

Other standard library traits with associated types:
- `Iterator` → `type Item`
- `Add` → `type Output`
- `IntoIterator` → `type Item` + `type IntoIter`
- `FromStr` → `type Err`
- `Deref` → `type Target`

---

## 4. Defining Traits with Associated Types

```rust
trait Container {
    type Item;

    fn items(&self) -> &[Self::Item];
    fn len(&self) -> usize { self.items().len() }
    fn is_empty(&self) -> bool { self.len() == 0 }
    fn first(&self) -> Option<&Self::Item> { self.items().first() }
}

struct IntBag {
    data: Vec<i32>,
}

impl Container for IntBag {
    type Item = i32;

    fn items(&self) -> &[i32] {
        &self.data
    }
}

struct WordBag {
    data: Vec<String>,
}

impl Container for WordBag {
    type Item = String;

    fn items(&self) -> &[String] {
        &self.data
    }
}

fn print_first<C: Container>(container: &C)
where
    C::Item: std::fmt::Display,
{
    match container.first() {
        Some(item) => println!("First: {item}"),
        None => println!("Empty!"),
    }
}

fn main() {
    let nums = IntBag { data: vec![10, 20, 30] };
    let words = WordBag { data: vec!["hello".into(), "world".into()] };

    println!("Nums len: {}", nums.len());
    print_first(&nums);   // First: 10
    print_first(&words);  // First: hello
}
```

---

## 5. Associated Constants

Traits can also have associated constants:

```rust
trait Shape {
    const DIMENSIONS: u32;
    type Area;

    fn area(&self) -> Self::Area;
    fn name(&self) -> &str;
}

struct Circle { radius: f64 }
struct Sphere { radius: f64 }

impl Shape for Circle {
    const DIMENSIONS: u32 = 2;
    type Area = f64;

    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }
    fn name(&self) -> &str { "Circle" }
}

impl Shape for Sphere {
    const DIMENSIONS: u32 = 3;
    type Area = f64;

    fn area(&self) -> f64 {
        4.0 * std::f64::consts::PI * self.radius * self.radius
    }
    fn name(&self) -> &str { "Sphere" }
}

fn describe<S: Shape>(shape: &S) where S::Area: std::fmt::Display {
    println!("{}: {}D, area = {}", shape.name(), S::DIMENSIONS, shape.area());
}

fn main() {
    describe(&Circle { radius: 5.0 });  // Circle: 2D, area = 78.54
    describe(&Sphere { radius: 5.0 });  // Sphere: 3D, area = 314.16
}
```

---

## 6. Fully Qualified Syntax

When a type implements multiple traits with the same method name:

```rust
trait Pilot { fn fly(&self); }
trait Wizard { fn fly(&self); }

struct Human;

impl Pilot for Human {
    fn fly(&self) { println!("Captain speaking..."); }
}

impl Wizard for Human {
    fn fly(&self) { println!("Up! *waves wand*"); }
}

impl Human {
    fn fly(&self) { println!("*flaps arms*"); }
}

fn main() {
    let h = Human;

    h.fly();              // calls Human::fly — *flaps arms*
    Pilot::fly(&h);       // calls Pilot::fly — Captain speaking...
    Wizard::fly(&h);      // calls Wizard::fly — Up! *waves wand*

    // Fully qualified syntax (most explicit):
    <Human as Pilot>::fly(&h);
    <Human as Wizard>::fly(&h);
}
```

---

## 7. Real-World Example: Graph Trait

The roadmap practice task:

```rust
trait Graph {
    type Node: std::fmt::Display;
    type Edge;

    fn nodes(&self) -> &[Self::Node];
    fn edges(&self) -> &[(usize, usize, Self::Edge)];  // (from, to, weight)

    fn node_count(&self) -> usize { self.nodes().len() }
    fn edge_count(&self) -> usize { self.edges().len() }

    fn neighbors(&self, node_idx: usize) -> Vec<usize> {
        self.edges().iter()
            .filter(|(from, _, _)| *from == node_idx)
            .map(|(_, to, _)| *to)
            .collect()
    }
}

struct CityMap {
    cities: Vec<String>,
    roads: Vec<(usize, usize, f64)>,  // (from, to, distance_km)
}

impl Graph for CityMap {
    type Node = String;
    type Edge = f64;

    fn nodes(&self) -> &[String] { &self.cities }
    fn edges(&self) -> &[(usize, usize, f64)] { &self.roads }
}

fn print_graph<G: Graph>(graph: &G) where G::Edge: std::fmt::Display {
    println!("Graph: {} nodes, {} edges", graph.node_count(), graph.edge_count());
    for (i, node) in graph.nodes().iter().enumerate() {
        let neighbors = graph.neighbors(i);
        println!("  {node} → {:?}", neighbors);
    }
}

fn main() {
    let map = CityMap {
        cities: vec!["London".into(), "Paris".into(), "Berlin".into()],
        roads: vec![
            (0, 1, 340.0),  // London → Paris
            (1, 2, 878.0),  // Paris → Berlin
            (0, 2, 932.0),  // London → Berlin
        ],
    };

    print_graph(&map);
}
```

---

## 8. Summary Cheat Sheet

```
ASSOCIATED TYPES
────────────────────────────────────────────────────────────
trait MyTrait {
    type Item;                     declare
    fn method(&self) -> Self::Item; use
}

impl MyTrait for Foo {
    type Item = String;            specify concrete type
    fn method(&self) -> String { ... }
}

ASSOCIATED CONSTANTS
────────────────────────────────────────────────────────────
trait Sized {
    const DIMS: u32;
}
impl Sized for Circle { const DIMS: u32 = 2; }

ASSOCIATED TYPES vs GENERICS
────────────────────────────────────────────────────────────
Associated type  → ONE implementation per type
Generic param    → MANY implementations per type

FULLY QUALIFIED SYNTAX
────────────────────────────────────────────────────────────
<Type as Trait>::method(&value)

STANDARD LIBRARY EXAMPLES
────────────────────────────────────────────────────────────
Iterator::Item, Add::Output, Deref::Target
FromStr::Err, IntoIterator::IntoIter
```

---

## What's Next?

**Lesson 46 — Workspaces** — Organize large Rust projects with Cargo workspaces. Manage multiple crates, shared dependencies, and build configurations.

## Further Reading
- [The Rust Book — Ch 19.2: Associated Types](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html#specifying-placeholder-types-in-trait-definitions-with-associated-types)
- [Rust by Example — Associated Types](https://doc.rust-lang.org/rust-by-example/generics/assoc_items/types.html)

---

*Associated types: one type, one implementation, zero ambiguity! 🦀*
