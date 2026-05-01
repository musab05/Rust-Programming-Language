# ✅ Lesson 35 — Answers: Collecting & FromIterator (C7)

---

## Section A

### A1
```
Rust
```
`collect()` gathers chars into a `String`.

### A2
```
3
```
`HashSet` deduplicates: {1, 2, 3} → len 3.

### A3
```
Ok([1, 2, 3])
```
All parses succeed, so `collect` produces `Ok(Vec)`.

### A4
```
true
```
`"bad"` fails to parse, so `collect` returns `Err`.

---

## Section B

### A5
```rust
use std::collections::HashMap;

fn main() {
    let freq: HashMap<char, usize> = "hello world".chars()
        .filter(|c| !c.is_whitespace())
        .fold(HashMap::new(), |mut acc, c| {
            *acc.entry(c).or_insert(0) += 1;
            acc
        });
    println!("{:?}", freq);
    // {'h': 1, 'e': 1, 'l': 3, 'o': 2, 'w': 1, 'r': 1, 'd': 1}
}
```

### A6
```rust
fn parse_all(items: &[&str]) -> Result<Vec<f64>, std::num::ParseFloatError> {
    items.iter().map(|s| s.parse::<f64>()).collect()
}

fn main() {
    let valid = vec!["1.5", "2.7", "3.14"];
    println!("{:?}", parse_all(&valid));  // Ok([1.5, 2.7, 3.14])

    let invalid = vec!["1.5", "oops", "3.14"];
    println!("{:?}", parse_all(&invalid));  // Err(ParseFloatError)
}
```

### A7
```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
    let (div3, rest): (Vec<i32>, Vec<i32>) = numbers.into_iter()
        .partition(|x| x % 3 == 0);
    println!("Div by 3: {:?}", div3);  // [3, 6, 9]
    println!("Rest:     {:?}", rest);  // [1, 2, 4, 5, 7, 8, 10]
}
```

### A8
```rust
struct CsvRow {
    fields: Vec<String>,
}

impl std::iter::FromIterator<String> for CsvRow {
    fn from_iter<I: IntoIterator<Item = String>>(iter: I) -> Self {
        CsvRow { fields: iter.into_iter().collect() }
    }
}

impl std::fmt::Display for CsvRow {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "{}", self.fields.join(","))
    }
}

fn main() {
    let row: CsvRow = vec!["Alice", "30", "Engineer"]
        .into_iter()
        .map(String::from)
        .collect();
    println!("{row}");  // Alice,30,Engineer
}
```

---

## Section C

### A9
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `collect` can produce any type implementing `FromIterator` |
| 2 | **True** | It short-circuits on the first `Err` |
| 3 | **False** | You can use type annotation on the variable instead |
| 4 | **True** | `partition` consumes the iterator and returns two collections |
| 5 | **True** | `String` implements `FromIterator<char>` and `FromIterator<&str>` |

### A10
`collect::<Result<Vec<_>, _>>()` is concise, idiomatic, and handles the "all or nothing" pattern cleanly. A manual loop would need a mutable Vec, a match on each Result, and early return logic — more code, more room for bugs. The `collect` approach also composes well with `?` for propagation.

---

## 🏆 Lesson 35 Complete!

✅ collect() into Vec, HashSet, BTreeSet, HashMap, String  
✅ Turbofish syntax  
✅ Collecting Results and Options  
✅ partition and unzip  
✅ FromIterator trait  

**Next up:** [Lesson 36 — Custom Error Types](../lesson_36_custom_errors/lesson_36_custom_errors.md) 🦀
