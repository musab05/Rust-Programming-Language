# ✅ Lesson 41 — Answers: Generics in Structs & Enums (T4)

---

## Section A

### A1 — ❌ Won't compile
`Point<T>` has one type parameter — both `x` and `y` must be the same type. `5` is `i32` and `10.0` is `f64`. Fix: use `Point<T, U>` with two type parameters.

### A2 — ✅ Compiles
`Point<T, U>` has two type parameters, so `x: i32` and `y: f64` is fine. Output: `5, 10`.

### A3 — ✅ Compiles (with the commented line staying commented)
`is_positive` is only implemented for `Wrapper<i32>`. `w` is `Wrapper<i32>` so it works. `w2` is `Wrapper<&str>` — calling `is_positive()` on it would fail, but that line is commented out. Output: `false`.

### A4 — ✅ Compiles
Generic enum works with any type. Output: `Just(42) Nothing`.

---

## Section B

### A5
```rust
use std::fmt::Display;

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
}

fn main() {
    let p = Pair::new(10, 25);
    p.cmp_display();  // Larger: 25

    let p2 = Pair::new("zebra", "apple");
    p2.cmp_display();  // Larger: zebra
}
```

### A6
```rust
use std::fmt::Display;

#[derive(Debug)]
enum Response<T, E> {
    Success(T),
    Failure(E),
    Pending,
}

impl<T: Display, E: Display> Response<T, E> {
    fn describe(&self) {
        match self {
            Response::Success(v) => println!("✅ Success: {v}"),
            Response::Failure(e) => println!("❌ Failure: {e}"),
            Response::Pending => println!("⏳ Pending..."),
        }
    }
}

fn main() {
    let r1: Response<String, String> = Response::Success("Data loaded".into());
    let r2: Response<i32, String> = Response::Failure("timeout".into());
    let r3: Response<(), ()> = Response::Pending;

    r1.describe();
    r2.describe();
    r3.describe();
}
```

### A7
```rust
#[derive(Debug)]
struct Point<T, U> { x: T, y: U }

impl<T, U> Point<T, U> {
    fn new(x: T, y: U) -> Self { Point { x, y } }

    fn mixup<V, W>(self, other: Point<V, W>) -> Point<T, W> {
        Point { x: self.x, y: other.y }
    }
}

fn main() {
    let p1 = Point::new(5, "hello");       // Point<i32, &str>
    let p2 = Point::new(true, 3.14);       // Point<bool, f64>
    let p3 = p1.mixup(p2);                 // Point<i32, f64>
    println!("{:?}", p3);                  // Point { x: 5, y: 3.14 }
}
```

---

## Section C

### A8
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | One type parameter means both fields share the same type |
| 2 | **True** | `impl Foo<f64>` is a concrete impl, only for that specific type |
| 3 | **True** | Methods can declare their own type parameters with `fn method<V>` |
| 4 | **True** | Both are generic enums from the standard library |
| 5 | **True** | `struct Foo<T, E = String>` — you can write `Foo<i32>` and E defaults to String |

### A9
`impl<T>` declares T as a **type parameter** for the impl block. Without it, the compiler would look for a concrete type named `T` (which doesn't exist). The `<T>` after `impl` says "this is generic over some type T," and then `Point<T>` uses that same T. Compare:
- `impl<T> Point<T>` — generic impl for all T
- `impl Point<f64>` — concrete impl for only f64

---

## 🏆 Lesson 41 Complete!

✅ Generic structs with one and multiple type parameters  
✅ Methods on generic structs (all T, specific T, bounded T)  
✅ Generic enums and pattern matching  
✅ Method-level type parameters (mixup)  
✅ Default type parameters  
✅ Standard library generics  

**Next up:** [Lesson 42 — impl Trait & dyn Trait](../lesson_42_impl_dyn_trait/lesson_42_impl_dyn_trait.md) 🦀
