# ✅ Lesson 39 — Answers: Trait Bounds (T2)

---

## Section A

### A1 — ❌ Won't compile
`println!("{item}")` requires `T: Display`, but there's no bound on `T`. Error: `T doesn't implement Display`.

### A2 — ✅ Compiles
`T: Display` bound is specified, and `i32` implements `Display`. Output: `42`.

### A3 — ✅ Compiles
`T: Add<Output = T>` lets us use `+`. `i32` implements `Add`. Output: `15`.

---

## Section B

### A4 — largest()
```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut max = list[0];
    for &item in &list[1..] {
        if item > max {
            max = item;
        }
    }
    max
}

fn main() {
    let numbers = vec![34, 50, 25, 100, 65];
    println!("Largest: {}", largest(&numbers));  // 100

    let chars = vec!['y', 'm', 'a', 'q'];
    println!("Largest: {}", largest(&chars));  // y
}
```
Needs `PartialOrd` for `>` comparison and `Copy` to copy values out of the slice.

### A5 — print_if_long
```rust
use std::fmt::Display;

fn print_if_long<T: Display>(items: &[T], min_len: usize) {
    for item in items {
        if item.to_string().len() >= min_len {
            println!("{item}");
        }
    }
}

fn main() {
    let words = vec!["hi", "hello", "hey", "greetings", "yo"];
    print_if_long(&words, 4);
    // hello
    // greetings
}
```

### A6 — where clause
```rust
use std::fmt::{Debug, Display};
use std::hash::Hash;

fn process<T, U>(a: T, b: U) -> String
where
    T: Display + Clone + Debug,
    U: PartialOrd + Hash,
{
    format!("{a}")
}
```

### A7 — Conditional methods
```rust
use std::fmt::Display;

struct Container<T> {
    items: Vec<T>,
}

impl<T> Container<T> {
    fn new() -> Self {
        Container { items: Vec::new() }
    }

    fn push(&mut self, item: T) {
        self.items.push(item);
    }
}

impl<T: Display> Container<T> {
    fn print_all(&self) {
        for item in &self.items {
            println!("  {item}");
        }
    }
}

impl<T: Ord> Container<T> {
    fn sort_contents(&mut self) {
        self.items.sort();
    }
}

fn main() {
    let mut c = Container::new();
    c.push(3);
    c.push(1);
    c.push(2);
    c.sort_contents();   // ✅ i32: Ord
    c.print_all();       // ✅ i32: Display
    // 1
    // 2
    // 3
}
```

---

## Section C

### A8
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `PartialOrd` provides `<`, `>`, `<=`, `>=` |
| 2 | **False** | `where` is just alternate syntax — same behavior |
| 3 | **False** | Blanket impls require a bound (e.g., `impl<T: Display> Trait for T`) |
| 4 | **False** | `f64` only implements `PartialOrd` because `NaN != NaN` breaks total ordering |
| 5 | **True** | `fn f(x: &impl Trait)` desugars to `fn f<T: Trait>(x: &T)` |

### A9
When `largest` returns `T` (not `&T`), it needs to **move or copy** a value out of the slice. Without `Copy`, `list[0]` would try to move out of the borrowed slice, which isn't allowed. `Copy` lets us copy the value. Alternatively, `Clone` with `.clone()` works for non-Copy types. The simplest fix is returning `&T` — then no Copy/Clone needed at all.

---

## 🏆 Lesson 39 Complete!

✅ Trait bound syntax: `T: Trait` and `impl Trait`  
✅ Multiple bounds with `+`  
✅ `where` clauses for readability  
✅ Conditional method implementations  
✅ Blanket implementations  
✅ Common standard library bounds  

**Next up:** [Lesson 40 — Generics in Functions](../lesson_40_generics/lesson_40_generics.md) 🦀
