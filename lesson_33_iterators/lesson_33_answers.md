# ✅ Lesson 33 — Answers: Iterators & Iterator Trait (C5)

---

## Section A

### A1
```
Some(10)
Some(20)
Some(30)
None
```
After the iterator is exhausted, `next()` returns `None`.

### A2
```
15
```
`sum()` adds all elements: 1 + 2 + 3 + 4 + 5 = 15.

### A3
```
[10, 20, 30]
```
`map` multiplies each element by 10, `collect` gathers into a Vec.

### A4
```
2
```
`filter` keeps only even numbers (2, 4), `count` returns 2.

### A5 — ❌ Won't compile
`into_iter()` takes ownership of `v`. The final `println!("{:?}", v)` tries to use `v` after it's been moved. Error: `value used after move`.

---

## Section B

### A6
```rust
for x in v.iter() { }       // x is &i32
for x in v.iter_mut() { }   // x is &mut i32
for x in v.into_iter() { }  // x is i32
```

### A7
```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    for x in v.iter_mut() {  // Changed: iter() → iter_mut(), and v must be mut
        *x += 1;
    }
    println!("{:?}", v);  // [2, 3, 4, 5, 6]
}
```
`iter()` gives `&i32` (immutable). Need `iter_mut()` to get `&mut i32`.

---

## Section C

### A8 — Word processor
```rust
fn main() {
    let words = vec!["the", "quick", "brown", "fox", "jumps",
                     "over", "the", "lazy", "brown", "dog"];

    // Step 1: Filter out short words (< 4 chars)
    // Step 2: Convert to uppercase
    let mut processed: Vec<String> = words.iter()
        .filter(|w| w.len() >= 4)       // adaptor 1: filter
        .map(|w| w.to_uppercase())       // adaptor 2: map
        .collect();

    // Step 3: Sort alphabetically
    processed.sort();                    // adaptor 3: sort

    // Step 4: Deduplicate
    processed.dedup();                   // adaptor 4: dedup

    // Step 5: Number each word
    let numbered: Vec<String> = processed.iter()
        .enumerate()                     // adaptor 5: enumerate
        .map(|(i, w)| format!("{}. {}", i + 1, w))
        .collect();

    for line in &numbered {
        println!("{line}");
    }
    // 1. BROWN
    // 2. JUMPS
    // 3. LAZY
    // 4. OVER
    // 5. QUICK
}
```

### A9 — Countdown iterator
```rust
struct Countdown {
    current: u32,
}

impl Countdown {
    fn new(start: u32) -> Countdown {
        Countdown { current: start }
    }
}

impl Iterator for Countdown {
    type Item = u32;

    fn next(&mut self) -> Option<u32> {
        if self.current > 0 {
            let val = self.current;
            self.current -= 1;
            Some(val)
        } else {
            None
        }
    }
}

fn main() {
    let countdown: Vec<u32> = Countdown::new(5).collect();
    println!("Countdown: {:?}", countdown);  // [5, 4, 3, 2, 1]

    let sum: u32 = Countdown::new(10).sum();
    println!("Sum 10..1: {sum}");  // 55
}
```

### A10 — Sum of squares of evens
```rust
fn main() {
    let result: i64 = (1..=100)
        .filter(|x| x % 2 == 0)
        .map(|x: i64| x * x)
        .sum();
    println!("Sum of squares of evens 1-100: {result}");  // 171700
}
```

### A11 — find and position
```rust
fn main() {
    let langs = vec!["rust", "python", "go", "java", "rust"];

    // 1. First language with more than 3 characters
    let long = langs.iter().find(|l| l.len() > 3);
    println!("First > 3 chars: {:?}", long);  // Some("rust")

    // 2. Position of "go"
    let pos = langs.iter().position(|&l| l == "go");
    println!("Position of 'go': {:?}", pos);  // Some(2)

    // 3. Any language starts with 'r'?
    let has_r = langs.iter().any(|l| l.starts_with('r'));
    println!("Any starts with 'r': {has_r}");  // true

    // 4. All lowercase?
    let all_lower = langs.iter().all(|l| l.chars().all(|c| c.is_lowercase()));
    println!("All lowercase: {all_lower}");  // true
}
```

---

## Section D

### A12 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Iterator adaptors are lazy — they do nothing until consumed |
| 2 | **True** | `for x in &v` desugars to `for x in v.iter()` |
| 3 | **False** | `into_iter()` takes ownership — the collection is moved |
| 4 | **True** | `fold` is the most general consumer — all others can be implemented with it |
| 5 | **True** | `for` loops use the `IntoIterator` trait under the hood |
| 6 | **False** | You can implement `Iterator` for any type (struct, enum, etc.) |

### A13 — Explanation
The `map` adaptor is **lazy**. It creates a new iterator but doesn't execute until consumed. Since nothing consumes the result (no `collect()`, `for_each()`, `sum()`, etc.), the closure is never called. The compiler will warn: "unused `Map` that must be used".

Fix: add `.for_each(drop)` or `.collect::<Vec<_>>()` or use a `for` loop.

### A14 — Pagination
```rust
fn get_page<T: Clone>(items: &[T], page: usize, per_page: usize) -> Vec<T> {
    items.iter()
        .skip(page * per_page)    // skip previous pages
        .take(per_page)           // take this page's items
        .cloned()
        .collect()
}

fn main() {
    let items: Vec<i32> = (0..1000).collect();
    let page_2 = get_page(&items, 1, 20);  // page index 1 = items 20-39
    println!("Page 2: {:?}", page_2);
    // [20, 21, 22, ..., 39]
}
```

---

## 🏆 Lesson 33 Complete!

✅ Iterator trait and `next()`  
✅ `iter()`, `into_iter()`, `iter_mut()` differences  
✅ Lazy evaluation  
✅ Consuming adaptors: `sum`, `count`, `fold`, `collect`, `find`  
✅ Custom iterators  
✅ Range iteration  

**Next up:** [Lesson 34 — Iterator Adaptors](../lesson_34_iterator_adaptors/lesson_34_iterator_adaptors.md) 🦀
