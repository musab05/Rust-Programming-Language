# ✅ Lesson 20 — Answers: HashMap\<K,V\> (C3)

---

## Section A

### A1
```
Some(99)
None
2
```
Second insert("a", 99) overwrites the first. "c" never inserted. len=2.

### A2
```
3
2
0
```
"hello" × 3, "world" × 2, "python" absent → `unwrap_or(&0)` → 0.

### A3
```
[1, 3]
[2]
2
```
`or_insert_with(Vec::new)` creates the Vec only if the key is absent. Each push adds to the existing Vec.

### A4
```
["y", "z"]
50
```
`retain` removes "x" (10 ≤ 15). Remaining: y=20, z=30. Keys sorted: ["y","z"]. Sum = 20+30 = 50.

---

## Section B

### A5 — ❌ No
`map.insert(key, 42)` moves `key` into the map. `println!("{key}")` uses moved value. **Error:** use of moved value `key`. Fix: insert `key.clone()` or insert `&key`.

### A6 — ✅ Compiles, ❌ Runtime panic on `map[&99]`
`map[&1]` = 10 fine. `map[&99]` panics — key not found. Fix: use `map.get(&99).unwrap_or(&0)`.

### A7 — ✅ Yes
`for (k, v) in &map` borrows — map is still usable after. Order not guaranteed, but all 3 pairs print. Output: 3 lines (order varies) then `still usable: 3`.

### A8 — ❌ No
`Point` derives `PartialEq` and `Eq` but NOT `Hash`. HashMap keys require all three. **Error:** `the trait Hash is not implemented for Point`. Fix: add `Hash` to the derive list.

---

## Section C

### A9 — Safe key access
```rust
use std::collections::HashMap;
fn main() {
    let map = HashMap::from([("a", 1), ("b", 2)]);
    // Fix: use get() which returns Option
    let val = map.get("c").copied().unwrap_or(0);
    println!("{val}"); // 0
}
```

### A10 — entry or_insert misuse
```rust
use std::collections::HashMap;
fn count(text: &str) -> HashMap<&str, i32> {
    let mut freq = HashMap::new();
    for word in text.split_whitespace() {
        let count = freq.entry(word).or_insert(0);
        *count += 1;   // fix: dereference with *
    }
    freq
}
fn main() { println!("{:?}", count("a b a c a b")); }
// {"a": 3, "b": 2, "c": 1}
```

### A11 — Ownership after insert
```rust
use std::collections::HashMap;
fn main() {
    let name = String::from("Alice");
    let mut map = HashMap::new();
    map.insert(name.clone(), 30u32);  // fix: clone name before moving
    println!("Hello, {name}!");        // ✅ original name still valid
}
```

---

## Section D

### A12 — Word frequency (roadmap practice task)
```rust
use std::collections::HashMap;

fn word_frequency(text: &str) -> HashMap<String, usize> {
    let mut freq: HashMap<String, usize> = HashMap::new();
    for word in text.split_whitespace() {
        let clean: String = word
            .chars()
            .filter(|c| c.is_alphabetic())
            .collect::<String>()
            .to_lowercase();
        if !clean.is_empty() {
            *freq.entry(clean).or_insert(0) += 1;
        }
    }
    freq
}

fn main() {
    let text = "To be or not to be that is the question \
                Whether tis nobler in the mind to suffer \
                The slings and arrows of outrageous fortune \
                Or to take arms against a sea of troubles";

    let freq = word_frequency(text);

    // Top 5 most frequent
    let mut pairs: Vec<(&String, &usize)> = freq.iter().collect();
    pairs.sort_by(|a, b| b.1.cmp(a.1).then(a.0.cmp(b.0)));
    println!("=== Top 5 ===");
    for (word, count) in pairs.iter().take(5) {
        println!("  {word}: {count}");
    }

    // Words appearing exactly once
    let singletons: Vec<&String> = freq.iter()
        .filter(|(_, &c)| c == 1)
        .map(|(w, _)| w)
        .collect();
    println!("Singletons: {}", singletons.len());

    // Unique word count
    println!("Unique words: {}", freq.len());
}
```

### A13 — Group by first letter
```rust
use std::collections::HashMap;

fn group_by_first_letter<'a>(words: &[&'a str]) -> HashMap<char, Vec<&'a str>> {
    let mut map: HashMap<char, Vec<&str>> = HashMap::new();
    for &word in words {
        if let Some(first) = word.chars().next() {
            map.entry(first).or_insert_with(Vec::new).push(word);
        }
    }
    for group in map.values_mut() { group.sort(); }
    map
}

fn main() {
    let words = ["banana", "apple", "cherry", "avocado", "blueberry", "apricot"];
    let grouped = group_by_first_letter(&words);
    let mut keys: Vec<char> = grouped.keys().cloned().collect();
    keys.sort();
    for k in keys {
        println!("{k}: {:?}", grouped[&k]);
    }
}
```

### A14 — Invert a map
```rust
use std::collections::HashMap;
use std::hash::Hash;

fn invert<K, V>(map: &HashMap<K, V>) -> HashMap<V, K>
where K: Clone + Eq + Hash, V: Clone + Eq + Hash
{
    map.iter().map(|(k, v)| (v.clone(), k.clone())).collect()
}

fn main() {
    let original = HashMap::from([
        (1u32, "one"), (2, "two"), (3, "three")
    ]);
    let inv = invert(&original);
    let mut keys: Vec<&&str> = inv.keys().collect();
    keys.sort();
    for k in keys { println!("{k} → {}", inv[k]); }
}
```

### A15 — Memoised Fibonacci
```rust
use std::collections::HashMap;

fn fib(n: u64, memo: &mut HashMap<u64, u64>) -> u64 {
    if n <= 1 { return n; }
    if let Some(&v) = memo.get(&n) { return v; }
    let result = fib(n - 1, memo) + fib(n - 2, memo);
    memo.insert(n, result);
    result
}

fn main() {
    let mut cache = HashMap::new();
    for i in [10, 20, 30, 40, 50] {
        println!("fib({i}) = {}", fib(i, &mut cache));
    }
}
```

### A16 — Student registry
```rust
use std::collections::HashMap;

struct Student { name: String, grades: Vec<f64> }

fn register(reg: &mut HashMap<String, Student>, name: &str) {
    reg.entry(name.to_string()).or_insert(Student { name: name.to_string(), grades: vec![] });
}

fn add_grade(reg: &mut HashMap<String, Student>, name: &str, grade: f64) {
    if let Some(s) = reg.get_mut(name) { s.grades.push(grade); }
}

fn average(reg: &HashMap<String, Student>, name: &str) -> Option<f64> {
    reg.get(name).map(|s| {
        if s.grades.is_empty() { return 0.0; }
        s.grades.iter().sum::<f64>() / s.grades.len() as f64
    })
}

fn top_student(reg: &HashMap<String, Student>) -> Option<&str> {
    reg.iter()
        .max_by(|(_, a), (_, b)| {
            let avg_a = a.grades.iter().sum::<f64>() / a.grades.len().max(1) as f64;
            let avg_b = b.grades.iter().sum::<f64>() / b.grades.len().max(1) as f64;
            avg_a.partial_cmp(&avg_b).unwrap()
        })
        .map(|(name, _)| name.as_str())
}

fn report(reg: &HashMap<String, Student>) {
    let mut students: Vec<(&String, f64)> = reg.iter()
        .map(|(name, s)| {
            let avg = if s.grades.is_empty() { 0.0 }
                      else { s.grades.iter().sum::<f64>() / s.grades.len() as f64 };
            (name, avg)
        }).collect();
    students.sort_by(|a, b| b.1.partial_cmp(&a.1).unwrap());
    println!("=== Student Report ===");
    for (name, avg) in students { println!("  {name}: {avg:.1}"); }
}

fn main() {
    let mut reg = HashMap::new();
    for name in ["Alice", "Bob", "Cara", "Dave"] { register(&mut reg, name); }
    let data = [("Alice",&[95.0,88.0,92.0][..]),("Bob",&[72.0,68.0,75.0]),
                ("Cara",&[85.0,90.0,88.0]),("Dave",&[60.0,55.0,70.0])];
    for (name, grades) in data {
        for &g in grades { add_grade(&mut reg, name, g); }
    }
    report(&reg);
    println!("Top: {:?}", top_student(&reg));
    println!("Alice avg: {:?}", average(&reg, "Alice"));
}
```

---

## Section E

### A17 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | HashMap is unordered — use `BTreeMap` for sorted order |
| 2 | **True** | `get` returns `None` if key absent — never panics |
| 3 | **True** | `map[&k]` panics with "key not found" if absent |
| 4 | **False** | `or_insert(v)` only inserts if the key is **absent** — existing value is unchanged |
| 5 | **True** | `String` implements `Eq + Hash` out of the box |
| 6 | **True** | All three are required: `PartialEq`, `Eq`, `Hash` |
| 7 | **False** | `retain` modifies in place — it's more efficient than filter+collect |
| 8 | **True** | Both `k` and `v` are moved into the map on `insert` |
| 9 | **False** | `.sum()` only works when the value type implements `Sum` — not every type |
| 10 | **True** | `or_default()` calls `Default::default()` — the type must implement `Default` |

### A18 — entry API deep dive
```
map.entry(k).or_insert(v)
  → Insert the value v if k is absent. Return &mut V.
  → Use when the default is a cheap constant value (0, false, "").

map.entry(k).or_insert_with(|| compute())
  → Insert the result of calling the closure if k is absent.
  → Use when the default is expensive to compute (allocates, reads from disk, etc.)
    — the closure is only called when the key is missing.

map.entry(k).or_default()
  → Insert Default::default() if k is absent.
  → Cleanest option when the type has a sensible zero/empty default.
  → Equivalent to or_insert_with(Default::default) but shorter.

map.entry(k).and_modify(|v| *v += 1).or_insert(0)
  → If k exists: call the closure to modify the existing value.
  → If k is absent: insert 0.
  → Use for increment patterns where the first occurrence should start at 0
    and subsequent ones should modify the existing value.
    (More explicit than *or_insert(0) += 1 when the init and update differ.)
```

### A19 — Big debug challenge
```rust
use std::collections::HashMap;

fn word_count(text: &str) -> HashMap<&str, i32> {
    let mut map = HashMap::new();
    for word in text.split_whitespace() {    // fix 1: split_whitespace handles any whitespace
        let count = map.entry(word).or_insert(0);
        *count += 1;                          // fix 2: dereference with *
    }
    map
}

fn main() {
    let text = "the cat sat on the mat the cat";
    let freq = word_count(text);
    println!("the: {}", freq["the"]);
    // fix 3: use get().unwrap_or instead of direct index for missing key
    println!("dog: {}", freq.get("dog").unwrap_or(&0));

    let mut scores: HashMap<&str, i32> = HashMap::new();
    scores.insert("Alice", 90);
    scores.insert("Bob",   80);

    // fix 4: collect keys first to avoid mutating while iterating
    let names: Vec<&str> = scores.keys().cloned().collect();
    for name in names {
        *scores.entry(name).or_insert(0) += 10;
    }
    println!("{:?}", scores);

    // fix 5: use &str key with & for String-keyed maps
    let mut map2: HashMap<String, i32> = HashMap::new();
    map2.insert(String::from("x"), 1);
    println!("{}", map2["x"]);  // ✅ &str coerces via Borrow<str>
    // Actually this already works! HashMap<String,_> accepts &str keys via Borrow.
    // If it didn't, fix would be: map2[&String::from("x")]
}
```

---

## 🏆 Lesson 20 Complete!

✅ `HashMap<K,V>` — key-value store with O(1) average lookup  
✅ Creating: `new()`, `from([...])`, `collect()` from zipped iterators  
✅ `insert` / `get` / `map[&k]` — inserting and accessing safely  
✅ The `entry` API — `or_insert`, `or_insert_with`, `or_default`, `and_modify`  
✅ Iterating: `&map`, `keys()`, `values()`, `values_mut()`  
✅ `remove` / `retain` / `clear`  
✅ Ownership — keys and values are moved into the map  
✅ Custom key types with `#[derive(PartialEq, Eq, Hash)]`  
✅ Common patterns: counting, grouping, inverting, memoising  
✅ Word frequency counter — roadmap practice ✓  

**Up next: Lesson 21 — panic! & unwrap 🦀**
