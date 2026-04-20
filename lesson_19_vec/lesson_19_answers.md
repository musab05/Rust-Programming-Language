# ✅ Lesson 19 — Answers: Vec\<T\> (C2)

---

## Section A

### A1

```
[3, 1, 4, 1, 5, 9, 2]
7
Some(2)
6
```

### A2

```
Some(30)
None
[30, 40, 50]
```

`retain` keeps elements > 25: 10 and 20 are removed.

### A3

```
[2, 4, 6, 8, 10]
30
```

Each element ×2. Sum = 2+4+6+8+10 = 30.

### A4

```
[1, 2, 3, 4, 5, 6, 9]
Ok(3)
Err(6)
```

After sort+dedup: [1,2,3,4,5,6,9]. `binary_search(&4)` → Ok(3) (index 3). `binary_search(&7)` → Err(6) (would insert at index 6).

---

## Section B

### A5 — ✅ Compiles, ❌ Runtime panic

`v[5]` on a 3-element Vec panics: `index out of bounds: the len is 3 but the index is 5`. Fix: use `v.get(5)`.

### A6 — ❌ No

Cannot move `String` out of a Vec by indexing. **Error:** cannot move out of index of `Vec<String>`. Fix: use `v[0].clone()` or `v.remove(0)`.

### A7 — ✅ Yes

`&arr` coerces `&[i32;5]→&[i32]`, `&v` coerces `&Vec<i32>→&[i32]`, `&v[1..]` is already `&[i32]`. Output: `15\n60\n50`.

### A8 — ❌ No

`first = &v[0]` immutably borrows `v`. Then `v.push(4)` requires a mutable borrow — conflict. **Error:** cannot borrow `v` as mutable because it is also borrowed as immutable.

---

## Section C

### A9 — Safe indexing

```rust
fn get_third(v: &[i32]) -> Option<i32> {   // fix 1: &[i32]; fix 2: Option
    v.get(2).copied()
}
fn main() {
    let v = vec![10, 20, 30];
    let empty: Vec<i32> = vec![];
    println!("{:?}", get_third(&v));      // Some(30)
    println!("{:?}", get_third(&empty));  // None — no panic
}
```

### A10 — Sort then dedup

```rust
fn main() {
    let mut v = vec![3, 1, 2, 1, 3, 2];
    v.sort();    // fix: sort FIRST — [1, 1, 2, 2, 3, 3]
    v.dedup();   // then dedup removes consecutive dups → [1, 2, 3]
    println!("{:?}", v); // [1, 2, 3]
}
```

### A11 — Borrow conflict

```rust
fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    // Fix A: copy the value (i32 is Copy)
    let first = v[0];   // copy, not borrow
    v.push(6);
    println!("{first}"); // 1

    // Fix B (if type isn't Copy): use borrow then drop it before push
    let first_ref = &v[0];
    println!("{first_ref}"); // borrow ends here (NLL)
    v.push(7);               // now safe
}
```

---

## Section D

### A12 — Median and Mode (roadmap practice task)

```rust
use std::collections::HashMap;

fn median(v: &[i32]) -> f64 {
    if v.is_empty() { return 0.0; }
    let mut sorted = v.to_vec();
    sorted.sort();
    let mid = sorted.len() / 2;
    if sorted.len() % 2 == 0 {
        (sorted[mid - 1] + sorted[mid]) as f64 / 2.0
    } else {
        sorted[mid] as f64
    }
}

fn mode(v: &[i32]) -> i32 {
    let mut freq: HashMap<i32, usize> = HashMap::new();
    for &x in v { *freq.entry(x).or_insert(0) += 1; }
    *freq.iter().max_by_key(|(_, c)| *c).unwrap().0
}

fn main() {
    let data = [2, 7, 4, 3, 2, 7, 2, 9, 1, 2];
    println!("data:   {:?}", data);
    println!("median: {}", median(&data));  // 2.5 (sorted: [1,2,2,2,2,3,4,7,7,9] → (2+3)/2)
    println!("mode:   {}", mode(&data));    // 2 (appears 4 times)
}
```

### A13 — Vec statistics

```rust
fn stats(v: &[f64]) -> (f64, f64, f64, f64) {
    if v.is_empty() { return (0.0, 0.0, 0.0, 0.0); }
    let min  = v.iter().cloned().fold(f64::INFINITY, f64::min);
    let max  = v.iter().cloned().fold(f64::NEG_INFINITY, f64::max);
    let mean = v.iter().sum::<f64>() / v.len() as f64;
    let var  = v.iter().map(|x| (x - mean).powi(2)).sum::<f64>() / v.len() as f64;
    (min, max, mean, var.sqrt())
}

fn main() {
    let data = vec![2.0, 4.0, 4.0, 4.0, 5.0, 5.0, 7.0, 9.0];
    let (min, max, mean, std) = stats(&data);
    println!("min={min} max={max} mean={mean:.2} std={std:.2}");
}
```

### A14 — Rotate and filter

```rust
fn rotate_left(v: &mut Vec<i32>, n: usize) {
    if v.is_empty() { return; }
    let n = n % v.len();
    let tail: Vec<i32> = v.drain(..n).collect();
    v.extend(tail);
}

fn keep_unique(v: Vec<i32>) -> Vec<i32> {
    use std::collections::HashMap;
    let mut freq: HashMap<i32, usize> = HashMap::new();
    for &x in &v { *freq.entry(x).or_insert(0) += 1; }
    v.into_iter().filter(|x| freq[x] == 1).collect()
}

fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    rotate_left(&mut v, 2);
    println!("{:?}", v); // [3, 4, 5, 1, 2]

    let u = keep_unique(vec![1, 2, 3, 2, 1, 4]);
    println!("{:?}", u); // [3, 4]
}
```

### A15 — Matrix

```rust
fn zeros(rows: usize, cols: usize) -> Vec<Vec<f64>> {
    vec![vec![0.0; cols]; rows]
}

fn identity(n: usize) -> Vec<Vec<f64>> {
    let mut m = zeros(n, n);
    for i in 0..n { m[i][i] = 1.0; }
    m
}

fn transpose(m: &Vec<Vec<f64>>) -> Vec<Vec<f64>> {
    let rows = m.len();
    let cols = m[0].len();
    let mut t = zeros(cols, rows);
    for i in 0..rows { for j in 0..cols { t[j][i] = m[i][j]; } }
    t
}

fn multiply(a: &Vec<Vec<f64>>, b: &Vec<Vec<f64>>) -> Option<Vec<Vec<f64>>> {
    let (ra, ca) = (a.len(), a[0].len());
    let (rb, cb) = (b.len(), b[0].len());
    if ca != rb { return None; }
    let mut result = zeros(ra, cb);
    for i in 0..ra { for j in 0..cb { for k in 0..ca {
        result[i][j] += a[i][k] * b[k][j];
    }}}
    Some(result)
}

fn print_matrix(m: &Vec<Vec<f64>>) {
    for row in m { println!("{}", row.iter().map(|x| format!("{:6.2}", x)).collect::<Vec<_>>().join(" ")); }
}

fn main() {
    let a = vec![vec![1.0,2.0],[3.0,4.0]];
    let id = identity(2);
    println!("A × I =");
    if let Some(result) = multiply(&a, &id) { print_matrix(&result); }
    println!("Transpose of A:");
    print_matrix(&transpose(&a));
}
```

### A16 — Grade book

```rust
struct Student { name: String, grades: Vec<f64> }

impl Student {
    fn add_grade(&mut self, g: f64) { self.grades.push(g); }
    fn average(&self) -> f64 {
        if self.grades.is_empty() { return 0.0; }
        self.grades.iter().sum::<f64>() / self.grades.len() as f64
    }
    fn highest(&self) -> f64 { self.grades.iter().cloned().fold(f64::NEG_INFINITY, f64::max) }
    fn lowest (&self) -> f64 { self.grades.iter().cloned().fold(f64::INFINITY, f64::min) }
    fn letter (&self) -> char {
        match self.average() as u32 { 90..=100=>'A', 80..=89=>'B', 70..=79=>'C', 60..=69=>'D', _=>'F' }
    }
}

fn main() {
    let mut students = vec![
        Student { name: "Alice".into(), grades: vec![] },
        Student { name: "Bob".into(),   grades: vec![] },
        Student { name: "Cara".into(),  grades: vec![] },
        Student { name: "Dave".into(),  grades: vec![] },
    ];

    for (grades, s) in [
        vec![92.0,88.0,95.0], vec![75.0,82.0,68.0],
        vec![55.0,63.0,70.0], vec![98.0,95.0,99.0]
    ].iter().zip(students.iter_mut()) {
        for &g in grades { s.add_grade(g); }
    }

    students.sort_by(|a, b| b.average().partial_cmp(&a.average()).unwrap());

    println!("=== Grade Ranking ===");
    for (rank, s) in students.iter().enumerate() {
        println!("{}. {} — avg={:.1} ({}) hi={:.0} lo={:.0}",
            rank+1, s.name, s.average(), s.letter(), s.highest(), s.lowest());
    }
}
```

---

## Section E

### A17 — True or False?

| #   | Answer                                          | Explanation                                                                                  |
| --- | ----------------------------------------------- | -------------------------------------------------------------------------------------------- |
| 1   | **True**                                        | `vec![0; 5]` = `[0, 0, 0, 0, 0]`                                                             |
| 2   | **True**                                        | `pop()` returns `None` on empty Vec — never panics                                           |
| 3   | **True**                                        | `v[i]` panics at runtime when out of bounds                                                  |
| 4   | **False**                                       | `for x in &v` yields `&T` — borrows, does not move                                           |
| 5   | **True**                                        | `f64` does not implement `Ord` (NaN breaks total ordering); use `sort_by` with `partial_cmp` |
| 6   | **False**                                       | `dedup` only removes **consecutive** duplicates — sort first                                 |
| 7   | **True**                                        | `retain` modifies the Vec in place, removing non-matching elements                           |
| 8   | **True**                                        | Deref coercion: `&Vec<T>` → `&[T]` in function calls                                         |
| 9   | **True** if `into_iter` / **False** if `iter()` | `extend(other_vec.iter())` borrows — `other_vec` stays valid. `extend(other_vec)` moves it   |
| 10  | **False**                                       | `with_capacity(n)` creates an empty Vec with **capacity** n, but zero elements               |

### A18 — Trace ownership

After line A: `s = "b"` (moved out), `v = ["a", "c"]` (`"b"` removed, Vec shifts left).

Line B: prints `b`.

Line C: prints `["a", "c"]`.

Line D: `let t = v[0]` — `String` is not `Copy`. You cannot move out of a Vec by indexing. **Compile error:** "cannot move out of index of `Vec<String>`". Fix: `v[0].clone()` or `v.remove(0)`.

---

## 🏆 Lesson 19 Complete!

✅ Creating Vecs: `vec![]`, `Vec::new()`, `with_capacity`, `collect()`  
✅ push/pop, insert/remove, retain, clear, extend  
✅ Safe access with `.get()` vs panicking `v[i]`  
✅ Iterating: `&v`, `&mut v`, consuming `v`, `enumerate()`  
✅ Slicing: `&v[1..4]`, coercion to `&[T]`  
✅ Sorting: `sort()`, `sort_by()`, `sort_by_key()`  
✅ `dedup()`, `drain()`, `windows()`, `chunks()`  
✅ Vec of enums for mixed-type storage  
✅ Ownership: Vec owns elements, `remove()` moves them out  
✅ Median and mode — roadmap practice ✓

**Up next: [Lesson 20 — HashMap\<K,V\>](../lesson_20_hashmap/lesson_20_hashmap.md) 🦀**
