# 🧪 Lesson 33 — Questions: Iterators & Iterator Trait (C5)

> **Lesson:** [lesson_33_iterators.md](./lesson_33_iterators.md)  
> **Answers:** [lesson_33_answers.md](./lesson_33_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let v = vec![10, 20, 30];
    let mut iter = v.iter();
    println!("{:?}", iter.next());
    println!("{:?}", iter.next());
    println!("{:?}", iter.next());
    println!("{:?}", iter.next());
}
```

### Q2
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let sum: i32 = v.iter().sum();
    println!("{sum}");
}
```

### Q3
```rust
fn main() {
    let v = vec![1, 2, 3];
    let result: Vec<i32> = v.iter().map(|x| x * 10).collect();
    println!("{:?}", result);
}
```

### Q4
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let even_count = v.iter().filter(|&&x| x % 2 == 0).count();
    println!("{even_count}");
}
```

### Q5 — Compile or not?
```rust
fn main() {
    let v = vec![String::from("hello"), String::from("world")];
    for s in v.into_iter() {
        println!("{s}");
    }
    println!("{:?}", v);
}
```

---

## Section B — iter vs into_iter vs iter_mut

### Q6
What type does each variable `x` have?
```rust
let v = vec![1_i32, 2, 3];
for x in v.iter() { }       // x is ?
for x in v.iter_mut() { }   // x is ?
for x in v.into_iter() { }  // x is ?
```

### Q7
Fix this code:
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    for x in v.iter() {
        *x += 1;
    }
    println!("{:?}", v);
}
```

---

## Section C — Write It Yourself

### Q8 — Word processor (Roadmap Practice Task)
Given a word list, chain 5 iterator adaptors to:
1. Filter out words shorter than 4 characters
2. Convert to uppercase
3. Sort alphabetically (hint: collect to Vec first, then sort)
4. Remove duplicates
5. Number each word

Test with: `["the", "quick", "brown", "fox", "jumps", "over", "the", "lazy", "brown", "dog"]`

### Q9 — Custom iterator: Countdown
Create a `Countdown` struct that counts from `n` down to 1. Implement `Iterator` for it. Test by collecting into a Vec and computing the sum.

### Q10 — Sum of squares
Using only iterator methods (no explicit loops), compute the sum of squares of all even numbers from 1 to 100.

### Q11 — find and position
Given `vec!["rust", "python", "go", "java", "rust"]`:
1. Find the first language with more than 3 characters
2. Find the position of "go"
3. Check if any language starts with 'r'
4. Check if all languages are lowercase

---

## Section D — Deep Understanding

### Q12 — True or False?
1. Iterator adaptors like `map` and `filter` execute immediately.
2. `for x in &v` is equivalent to `for x in v.iter()`.
3. After `into_iter()`, the original collection can still be used.
4. `fold` can do everything `sum`, `count`, and `max` do.
5. Every `for` loop in Rust uses iterators under the hood.
6. You can only implement `Iterator` for structs, not enums.

### Q13 — Explain
Why does this code not print anything?
```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    v.iter().map(|x| {
        println!("{x}");
        x * 2
    });
}
```

### Q14 — Design
You're building a pagination system. You have a `Vec<Item>` with 1000 items and need to return items 20-39 (page 2, 20 items per page). Write this using only iterator methods.

---

*Iterators: the backbone of functional Rust! 🦀*
