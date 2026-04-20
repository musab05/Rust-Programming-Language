# 🧪 Lesson 20 — Questions: HashMap\<K,V\> (C3)

> **Lesson:** [lesson_20_hashmap.md](./lesson_20_hashmap.md)  
> **Answers:** [lesson_20_answers.md](./lesson_20_answers.md)

---

## Section A — Predict the Output

### Q1
```rust
use std::collections::HashMap;
fn main() {
    let mut map = HashMap::new();
    map.insert("a", 1);
    map.insert("b", 2);
    map.insert("a", 99);  // overwrite
    println!("{:?}", map.get("a"));
    println!("{:?}", map.get("c"));
    println!("{}", map.len());
}
```

### Q2
```rust
use std::collections::HashMap;
fn main() {
    let text = "hello world hello rust world hello";
    let mut freq: HashMap<&str, u32> = HashMap::new();
    for word in text.split_whitespace() {
        *freq.entry(word).or_insert(0) += 1;
    }
    println!("{}", freq["hello"]);
    println!("{}", freq["world"]);
    println!("{}", freq.get("python").unwrap_or(&0));
}
```

### Q3
```rust
use std::collections::HashMap;
fn main() {
    let mut map: HashMap<&str, Vec<i32>> = HashMap::new();
    map.entry("odds").or_insert_with(Vec::new).push(1);
    map.entry("odds").or_insert_with(Vec::new).push(3);
    map.entry("evens").or_insert_with(Vec::new).push(2);
    println!("{:?}", map["odds"]);
    println!("{:?}", map["evens"]);
    println!("{}", map.len());
}
```

### Q4
```rust
use std::collections::HashMap;
fn main() {
    let mut map = HashMap::from([("x", 10i32), ("y", 20), ("z", 30)]);
    map.retain(|_, v| *v > 15);
    let mut keys: Vec<&&str> = map.keys().collect();
    keys.sort();
    println!("{keys:?}");
    println!("{}", map.values().sum::<i32>());
}
```

---

## Section B — Will It Compile?

### Q5
```rust
use std::collections::HashMap;
fn main() {
    let key = String::from("hello");
    let mut map = HashMap::new();
    map.insert(key, 42);
    println!("{key}");  // key after insert
}
```

### Q6
```rust
use std::collections::HashMap;
fn main() {
    let mut map: HashMap<i32, i32> = HashMap::new();
    map.insert(1, 10);
    println!("{}", map[&1]);
    println!("{}", map[&99]); // missing key
}
```

### Q7
```rust
use std::collections::HashMap;
fn main() {
    let map = HashMap::from([(1u32, "one"), (2, "two"), (3, "three")]);
    for (k, v) in &map {
        println!("{k}: {v}");
    }
    println!("still usable: {}", map.len());
}
```

### Q8
```rust
use std::collections::HashMap;
#[derive(PartialEq, Eq)]  // no Hash
struct Point { x: i32, y: i32 }

fn main() {
    let mut map: HashMap<Point, &str> = HashMap::new();
    map.insert(Point { x: 0, y: 0 }, "origin");
}
```

---

## Section C — Fix the Bugs

### Q9 — Safe key access
```rust
use std::collections::HashMap;
fn main() {
    let map = HashMap::from([("a", 1), ("b", 2)]);
    let val = map["c"];   // BUG: panics — key not found
    println!("{val}");
}
```

### Q10 — entry or_insert misuse
```rust
use std::collections::HashMap;
fn count(text: &str) -> HashMap<&str, i32> {
    let mut freq = HashMap::new();
    for word in text.split_whitespace() {
        let count = freq.entry(word).or_insert(0);
        count += 1;    // BUG: forgot to dereference
    }
    freq
}
fn main() { println!("{:?}", count("a b a c a b")); }
```

### Q11 — Ownership after insert
```rust
use std::collections::HashMap;
fn main() {
    let name = String::from("Alice");
    let mut map = HashMap::new();
    map.insert(name, 30u32);
    println!("Hello, {name}!");  // BUG: name moved
}
```
Fix without changing the insert call (don't clone the whole map). Hint: insert a reference or clone only what's needed.

---

## Section D — Write It Yourself

### Q12 — Roadmap Practice Task: Word frequency counter
This is the exact roadmap task. Write `fn word_frequency(text: &str) -> HashMap<String, usize>` that:
- Lowercases all words
- Strips punctuation from start/end of each word
- Counts occurrences

In `main`: count frequency in a longer paragraph, then print:
1. The top-5 most frequent words
2. All words that appear exactly once
3. Total unique word count

### Q13 — Group by first letter
Write `fn group_by_first_letter(words: &[&str]) -> HashMap<char, Vec<&str>>` — groups words by their first character. Sort each group alphabetically.

### Q14 — Invert a map
Write `fn invert<K, V>(map: &HashMap<K, V>) -> HashMap<V, K>` where `K: Clone + Eq + Hash` and `V: Clone + Eq + Hash`. Test on `{1→"one", 2→"two", 3→"three"}`.

### Q15 — Memoised Fibonacci
Write `fn fib(n: u64, memo: &mut HashMap<u64, u64>) -> u64` using HashMap as a cache. Measure that `fib(50)` is instant.

### Q16 — Full program: Student registry
```rust
struct Student { name: String, grades: Vec<f64> }
```
Build a `HashMap<String, Student>` registry:
- `fn register(registry: &mut HashMap<String, Student>, name: &str)`
- `fn add_grade(registry: &mut HashMap<String, Student>, name: &str, grade: f64)`
- `fn average(registry: &HashMap<String, Student>, name: &str) -> Option<f64>`
- `fn top_student(registry: &HashMap<String, Student>) -> Option<&str>`
- `fn report(registry: &HashMap<String, Student>)` — sorted by average descending

Test with 4 students and multiple grades each.

---

## Section E — Deep Understanding

### Q17 — True or False?
1. HashMap guarantees iteration order matches insertion order.
2. `map.get(&k)` returns `Option<&V>` — it never panics.
3. `map[&k]` panics if the key is not present.
4. `map.entry(k).or_insert(v)` overwrites an existing value with `v`.
5. `String` can be a HashMap key out of the box.
6. For a custom type to be a key it needs to derive `PartialEq`, `Eq`, and `Hash`.
7. `map.retain(|k, v| ...)` is equivalent to filtering and rebuilding the map.
8. Inserting a value with `map.insert(k, v)` moves both `k` and `v` into the map.
9. `map.values().sum()` works for any value type.
10. `map.entry("key").or_default()` requires the value type to implement `Default`.

### Q18 — entry API deep dive
Explain what each of these does and when you'd use each:
```rust
map.entry(k).or_insert(v)
map.entry(k).or_insert_with(|| compute())
map.entry(k).or_default()
map.entry(k).and_modify(|v| *v += 1).or_insert(0)
```

### Q19 — Big debug challenge (5 bugs)
```rust
use std::collections::HashMap;

fn word_count(text: &str) -> HashMap<&str, i32> {
    let mut map = HashMap::new();
    for word in text.split(" ") {          // Bug 1: doesn't handle multiple spaces
        let count = map.entry(word).or_insert(0);
        count = count + 1;                 // Bug 2: forgot dereference, wrong syntax
    }
    map
}

fn main() {
    let text = "the cat sat on the mat the cat";
    let freq = word_count(text);
    println!("the: {}", freq["the"]);
    println!("dog: {}", freq["dog"]);     // Bug 3: panics — key not present

    let mut scores: HashMap<&str, i32> = HashMap::new();
    scores.insert("Alice", 90);
    scores.insert("Bob",   80);

    // Bug 4: modifying while iterating
    for (name, score) in &scores {
        scores.insert(name, score + 10);  // can't mutate while borrowing
    }

    // Bug 5: key type mismatch
    let mut map2: HashMap<String, i32> = HashMap::new();
    map2.insert(String::from("x"), 1);
    println!("{}", map2["x"]);            // &str key on String-keyed map
}
```
