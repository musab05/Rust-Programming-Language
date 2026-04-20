# 🧪 Lesson 19 — Questions: Vec\<T\> (C2)

> **Lesson:** [lesson_19_vec.md](./lesson_19_vec.md)  
> **Answers:** [lesson_19_answers.md](./lesson_19_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let mut v = vec![3, 1, 4, 1, 5];
    v.push(9);
    v.push(2);
    println!("{:?}", v);
    println!("{}", v.len());
    println!("{:?}", v.pop());
    println!("{}", v.len());
}
```

### Q2
```rust
fn main() {
    let mut v = vec![10, 20, 30, 40, 50];
    println!("{:?}", v.get(2));
    println!("{:?}", v.get(10));
    v.retain(|&x| x > 25);
    println!("{:?}", v);
}
```

### Q3
```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    for x in &mut v { *x *= 2; }
    println!("{:?}", v);
    let sum: i32 = v.iter().sum();
    println!("{sum}");
}
```

### Q4
```rust
fn main() {
    let mut v = vec![3, 1, 4, 1, 5, 9, 2, 6];
    v.sort();
    v.dedup();
    println!("{:?}", v);
    println!("{:?}", v.binary_search(&4));
    println!("{:?}", v.binary_search(&7));
}
```

---

## Section B — Will It Compile?

### Q5
```rust
fn main() {
    let v = vec![1, 2, 3];
    println!("{}", v[5]);
}
```

### Q6
```rust
fn main() {
    let v = vec![String::from("hello"), String::from("world")];
    let s = v[0];   // move out of Vec
    println!("{s}");
}
```

### Q7
```rust
fn sum(data: &[i32]) -> i32 { data.iter().sum() }

fn main() {
    let arr = [1, 2, 3, 4, 5];
    let v   = vec![10, 20, 30];
    println!("{}", sum(&arr));
    println!("{}", sum(&v));
    println!("{}", sum(&v[1..]));
}
```

### Q8
```rust
fn main() {
    let v = vec![1, 2, 3];
    let first = &v[0];
    v.push(4);           // mutable borrow while immutable exists
    println!("{first}");
}
```

---

## Section C — Fix the Bugs

### Q9 — Safe indexing
```rust
fn get_third(v: &Vec<i32>) -> i32 {  // BUG 1: wrong param type
    v[2]                               // BUG 2: panics if len < 3
}
fn main() {
    let v = vec![10, 20, 30];
    let empty: Vec<i32> = vec![];
    println!("{}", get_third(&v));
    println!("{}", get_third(&empty)); // panic!
}
```

### Q10 — Sort then dedup
```rust
fn main() {
    let mut v = vec![3, 1, 2, 1, 3, 2];
    v.dedup();   // BUG: dedup only removes CONSECUTIVE duplicates
    println!("{:?}", v);  // won't give [1, 2, 3]
}
```

### Q11 — Borrow conflict
```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    let first = &v[0];      // immutable borrow
    v.push(6);              // BUG: mutable borrow while immutable exists
    println!("{first}");
}
```

---

## Section D — Write It Yourself

### Q12 — Roadmap Practice Task: Median and Mode
This is the exact roadmap practice task. Write:
- `fn median(v: &[i32]) -> f64` — sort a copy, return middle value (or average of two middles if even length)
- `fn mode(v: &[i32]) -> i32` — return the most frequent value (any if tie)
- In `main`: compute both on `[2, 7, 4, 3, 2, 7, 2, 9, 1, 2]`

### Q13 — Vec statistics
Write `fn stats(v: &[f64]) -> (f64, f64, f64, f64)` returning (min, max, mean, std_dev). Handle empty slices with `(0.0, 0.0, 0.0, 0.0)`.

### Q14 — Rotate and filter
Write:
- `fn rotate_left(v: &mut Vec<i32>, n: usize)` — rotate elements left by n using drain
- `fn keep_unique(v: Vec<i32>) -> Vec<i32>` — return only elements that appear exactly once

### Q15 — Matrix as Vec of Vecs
Represent a 2D matrix as `Vec<Vec<f64>>`:
- `fn zeros(rows: usize, cols: usize) -> Vec<Vec<f64>>`
- `fn identity(n: usize) -> Vec<Vec<f64>>`
- `fn transpose(m: &Vec<Vec<f64>>) -> Vec<Vec<f64>>`
- `fn multiply(a: &Vec<Vec<f64>>, b: &Vec<Vec<f64>>) -> Option<Vec<Vec<f64>>>` — returns None if dimensions don't match
- Print matrices nicely

### Q16 — Full program: grade book
```rust
struct Student { name: String, grades: Vec<f64> }
```
Implement:
- `fn add_grade(&mut self, g: f64)`
- `fn average(&self) -> f64`
- `fn highest(&self) -> f64`
- `fn lowest(&self) -> f64`
- `fn letter(&self) -> char` — A≥90 B≥80 C≥70 D≥60 F otherwise

Create 4 students, add grades, sort by average descending, print the ranking.

---

## Section E — Deep Understanding

### Q17 — True or False?
1. `vec![0; 5]` creates a Vec with 5 zeros.
2. `v.pop()` returns `Option<T>` — it never panics.
3. `v[i]` panics at runtime if `i >= v.len()`.
4. Iterating with `for x in &v` moves elements out of the Vec.
5. `v.sort()` requires `T: Ord`, which `f64` does NOT implement.
6. `v.dedup()` removes all duplicates regardless of position.
7. `v.retain(|x| ...)` modifies the Vec in place.
8. `&v` in a function call coerces `&Vec<T>` to `&[T]`.
9. `v.extend(other_vec)` moves `other_vec` — it cannot be used afterward.
10. `Vec::with_capacity(n)` creates a Vec with `n` elements, all zero.

### Q18 — Trace ownership
```rust
fn main() {
    let mut v = vec![
        String::from("a"),
        String::from("b"),
        String::from("c"),
    ];
    let s = v.remove(1);   // line A
    println!("{s}");        // line B
    println!("{:?}", v);    // line C
    // let t = v[0];        // line D — what would happen?
}
```
What is the value of `s`, `v` after line A? What would happen at line D?
