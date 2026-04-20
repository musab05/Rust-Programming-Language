# 🧪 Lesson 11 — Questions: Slices (O4)

> **Lesson:** [lesson_11_slices.md](./lesson_11_slices.md)  
> **Answers:** [lesson_11_answers.md](./lesson_11_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
fn main() {
    let s = String::from("hello world");
    let a = &s[..5];
    let b = &s[6..];
    let c = &s[..];
    println!("{a}");
    println!("{b}");
    println!("{}", c.len());
}
```

### Q2
```rust
fn main() {
    let arr = [10, 20, 30, 40, 50];
    let mid = &arr[1..4];
    println!("{:?}", mid);
    println!("{}", arr.len());
    println!("{}", mid.len());
}
```

### Q3
```rust
fn sum(data: &[i32]) -> i32 { data.iter().sum() }

fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let a = [10, 20, 30];
    println!("{}", sum(&v));
    println!("{}", sum(&a));
    println!("{}", sum(&v[2..]));
}
```

### Q4
```rust
fn main() {
    let data = [1, 2, 3, 4, 5, 6];
    for w in data.windows(3) { print!("{w:?} "); }
    println!();
    for c in data.chunks(2) { print!("{c:?} "); }
}
```

### Q5
```rust
fn first_word(s: &str) -> &str {
    match s.find(' ') {
        Some(i) => &s[..i],
        None    => s,
    }
}
fn main() {
    let s = String::from("rust programming");
    let w = first_word(&s);
    println!("{w}");
    println!("{s}");
}
```

---

## Section B — Will It Compile? (Yes / No + Why)

### Q6
```rust
fn main() {
    let s = String::from("hello");
    let r = &s[..];
    println!("{r}");
    println!("{s}");
}
```

### Q7
```rust
fn main() {
    let s = String::from("hello");
    let r = &s[..];
    s.push_str(" world"); // mutate while borrowed
    println!("{r}");
}
```

### Q8
```rust
fn main() {
    let emoji = "😀hi";
    let bad = &emoji[0..2]; // 😀 is 4 bytes
    println!("{bad}");
}
```

### Q9
```rust
fn takes_str(s: &str) -> usize { s.len() }
fn main() {
    let owned = String::from("hello");
    let result = takes_str(&owned);
    println!("{result}");
    println!("{owned}");
}
```

### Q10
```rust
fn main() {
    let mut arr = [1, 2, 3, 4, 5];
    let a = &mut arr[..2];
    let b = &mut arr[3..]; // simultaneous mutable borrows
    a[0] = 100;
    b[0] = 400;
    println!("{:?}", arr);
}
```

---

## Section C — Fix the Bugs

### Q11 — Inflexible parameter
```rust
fn word_count(s: &String) -> usize {   // BUG: wrong param type
    s.split_whitespace().count()
}
fn main() {
    let owned = String::from("one two three");
    let literal = "four five";
    println!("{}", word_count(&owned));
    println!("{}", word_count(literal)); // ERROR
}
```

### Q12 — Unicode panic
```rust
fn main() {
    let s = "Héllo";  // 'é' is 2 bytes
    let part = &s[0..2]; // BUG: slices inside 'é'
    println!("{part}");
}
```
Fix it to safely print the first **3 characters**.

### Q13 — Wrong return from first_word
```rust
fn first_word(s: &str) -> &str {
    let words: Vec<&str> = s.split_whitespace().collect();
    words[0]  // BUG: returns reference into local Vec
}
fn main() {
    let s = String::from("hello world");
    println!("{}", first_word(&s));
}
```

### Q14 — Out-of-bounds access
```rust
fn third_element(data: &[i32]) -> i32 {
    data[2]  // BUG: panics if len < 3
}
fn main() {
    let a = vec![10, 20, 30];
    let b: Vec<i32> = vec![];
    println!("{}", third_element(&a));
    println!("{}", third_element(&b)); // panic!
}
```
Fix `third_element` to return `Option<i32>`.

---

## Section D — Write It Yourself

### Q15 — Practice: `first_word`
The roadmap practice task. Write `fn first_word(s: &str) -> &str` that:
- Returns the first word (up to the first space)
- Returns the whole string if there is no space
- Does **not** allocate — returns a slice of the input

Test it on: `"rust programming language"`, `"single"`, `""`.

### Q16 — `last_word`
Write `fn last_word(s: &str) -> &str` — returns the last whitespace-delimited word, or the whole string if none. Return a slice of the input, no allocation.

### Q17 — Array statistics
Write `fn stats(data: &[f64]) -> (f64, f64, f64)` returning (min, max, mean). Handle empty slices by returning `(0.0, 0.0, 0.0)`. Call it on:
- The full Vec
- The first half
- The last half
Show the original Vec is unchanged after all calls.

### Q18 — Count vowels
Write `fn count_vowels(s: &str) -> usize` that counts ASCII vowels (a, e, i, o, u — upper and lower). Use a slice/iterator approach, no indexing.

### Q19 — Sliding window max
Write `fn window_max(data: &[i32], window: usize) -> Vec<i32>` that returns the maximum of each window of size `window` using `.windows()`. If `window > data.len()` return an empty Vec.

### Q20 — `split_at_mut` swap
Write `fn rotate_left(data: &mut [i32], n: usize)` that rotates the slice left by `n` positions **in place**, using `split_at_mut` and `.reverse()`. Example: `[1,2,3,4,5]` rotated left 2 → `[3,4,5,1,2]`. Hint: reverse left half, reverse right half, reverse whole.

### Q21 — Full program: word analysis
Build a program that takes a multi-word string and prints:
- Total characters
- Total words
- Longest word (as a slice, no clone)
- Whether it's a palindrome (whole string, ignoring spaces)
- The words sorted alphabetically (collect into a `Vec<&str>` from slices)

---

## Section E — Deep Understanding

### Q22 — True or False?
1. A `&str` always points to data in the program binary.
2. `data.get(5)` panics if index 5 is out of bounds.
3. `&Vec<i32>` automatically coerces to `&[i32]` in function calls.
4. `.windows(n)` and `.chunks(n)` both produce non-overlapping sub-slices.
5. A slice `&[T]` is a fat pointer: a pointer plus a length.
6. `"café"[0..3]` is always safe because ASCII chars are 1 byte.
7. `.sort()` on a `&mut [T]` requires `T: Ord`.
8. You can have two simultaneous `&mut [T]` slices from the same Vec using `split_at_mut`.
9. `&s[..]` on a `String` gives a `&str` of the whole string.
10. Returning a `&str` from a function that was sliced from a local `String` is safe as long as you annotate the lifetime.

### Q23 — Debug challenge (5 bugs)
Find and fix every bug:
```rust
fn longest_word(sentence: &String) -> &String {   // Bug 1
    sentence
        .split_whitespace()
        .max_by_key(|w| w.len())
        .unwrap_or("")                             // Bug 2: type mismatch
}

fn double_slice(data: &[i32]) -> Vec<i32> {
    let mut out = Vec::new();
    for i in 0..data.len() + 1 {                  // Bug 3: off by one
        out.push(data[i] * 2);
    }
    out
}

fn main() {
    let s = String::from("the quick brown fox");
    println!("{}", longest_word(&s));

    let v = vec![1, 2, 3];
    let emoji_s = "😀bye";
    let part = &emoji_s[0..2];                    // Bug 4: inside emoji
    println!("{part}");

    let mut arr = [5, 3, 1, 4, 2];
    let (left, right) = arr.split_at_mut(2);
    left.sort();
    right.sort();
    // Bug 5: arr is partially sorted, not fully sorted
    // Add one more step to fully sort arr
    println!("{:?}", arr);
}
```
