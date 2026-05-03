# ✅ Lesson 50 — Answers: Higher-Order Functions (CL3)

---

## Section A

### A1
```
26
```
`5 * 5 + 1 = 26`.

### A2
```
15 30
```
`make_adder(10)` returns `|x| x + 10`. So `f(5) = 15`, `f(20) = 30`.

### A3
```
16
```
`twice(f, 10)` computes `f(f(10))` = `f(13)` = `16`.

---

## Section B

### A4
```rust
fn compose<A, B, C, F, G>(f: F, g: G) -> impl Fn(A) -> C
where
    F: Fn(A) -> B,
    G: Fn(B) -> C,
{
    move |x| g(f(x))
}

fn main() {
    let double = |x: i32| x * 2;
    let to_string = |x: i32| format!("Result: {x}");

    let pipeline = compose(double, to_string);
    println!("{}", pipeline(21));  // Result: 42
}
```

### A5
```rust
struct Pipeline<T> { value: T }

impl<T> Pipeline<T> {
    fn new(value: T) -> Self { Pipeline { value } }

    fn pipe<U, F: FnOnce(T) -> U>(self, f: F) -> Pipeline<U> {
        Pipeline { value: f(self.value) }
    }

    fn finish(self) -> T { self.value }
}

fn main() {
    let result = Pipeline::new(vec![5, 1, 8, 3, 2, 9, 4, 7, 6])
        .pipe(|mut v| { v.sort(); v })
        .pipe(|v| v.into_iter().filter(|&x| x > 3).collect::<Vec<_>>())
        .pipe(|v| v.iter().map(|x| x * x).sum::<i32>())
        .finish();

    println!("Result: {result}");  // 16+25+36+49+64+81 = 271
}
```

### A6
```rust
use std::collections::HashMap;

fn main() {
    let mut fns: HashMap<String, Box<dyn Fn(f64) -> f64>> = HashMap::new();
    fns.insert("sqrt".into(), Box::new(|x| x.sqrt()));
    fns.insert("abs".into(), Box::new(|x| x.abs()));
    fns.insert("negate".into(), Box::new(|x| -x));

    let tests = vec![("sqrt", 16.0), ("abs", -42.0), ("negate", 7.0)];
    for (name, val) in tests {
        if let Some(f) = fns.get(name) {
            println!("{name}({val}) = {}", f(val));
        }
    }
    // sqrt(16) = 4
    // abs(-42) = 42
    // negate(7) = -7
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | They all take closures as arguments — classic HOFs |
| 2 | **False** | Rust doesn't auto-curry; you simulate it with nested closures |
| 3 | **True** | Captured variables must be moved into the returned closure |
| 4 | **True** | Both produce identical results; readability depends on context |
| 5 | **True** | A function returning a function is a higher-order function by definition |

### A8
Both produce the same result. **Version B** (functional) is more idiomatic in Rust for data transformation pipelines:
- More declarative — describes WHAT, not HOW
- No mutable state
- Composable — easy to add more steps
- The iterator chain may be optimized by the compiler

Version A (imperative) is fine for simple cases and may be clearer for developers new to functional style.

---

## 🏆 Lesson 50 Complete — Congratulations! 🎉

✅ Higher-order functions  
✅ Function composition  
✅ Currying and partial application  
✅ Functional pipelines  
✅ Iterator HOFs  
✅ Function registries  

**You've completed 50 Rust lessons!** 🦀  
Next: Smart Pointers, Concurrency, and Async Rust await!
