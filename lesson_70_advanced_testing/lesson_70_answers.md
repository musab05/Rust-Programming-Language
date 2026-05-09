# ✅ Lesson 70 — Answers: Mocking & Property-Based Testing (TE2)

---

## Section A

### A1
- **Stub**: returns pre-configured values. You control WHAT it returns. Focus: providing test data.
- **Mock**: records interactions and verifies them. You check HOW it was called (which methods, how many times, in what order). Focus: verifying behavior.

In practice, many test doubles blend both (provide canned responses AND verify calls).

### A2
- **Example-based**: tests specific inputs you thought of. May miss edge cases.
- **Property-based**: tests hundreds/thousands of random inputs against invariants. Finds edge cases you never considered. Automatically shrinks failing inputs to the simplest reproduction.

---

## Section B

### A3
```rust
use std::collections::HashMap;

trait Database {
    fn get(&self, key: &str) -> Option<String>;
    fn set(&mut self, key: &str, value: &str);
}

struct MockDatabase { store: HashMap<String, String> }
impl MockDatabase {
    fn new() -> Self { MockDatabase { store: HashMap::new() } }
}

impl Database for MockDatabase {
    fn get(&self, key: &str) -> Option<String> { self.store.get(key).cloned() }
    fn set(&mut self, key: &str, value: &str) { self.store.insert(key.into(), value.into()); }
}

struct Cache<D: Database> { db: D }

impl<D: Database> Cache<D> {
    fn new(db: D) -> Self { Cache { db } }
    fn get_or_default(&self, key: &str, default: &str) -> String {
        self.db.get(key).unwrap_or_else(|| default.into())
    }
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_cache_hit() {
        let mut db = MockDatabase::new();
        db.set("name", "Alice");
        let cache = Cache::new(db);
        assert_eq!(cache.get_or_default("name", "unknown"), "Alice");
    }

    #[test]
    fn test_cache_miss() {
        let db = MockDatabase::new();
        let cache = Cache::new(db);
        assert_eq!(cache.get_or_default("name", "unknown"), "unknown");
    }
}
```

### A4
```rust
use proptest::prelude::*;

proptest! {
    #[test]
    fn test_sort_reverse_is_descending(mut v in prop::collection::vec(any::<i32>(), 0..50)) {
        v.sort();
        v.reverse();
        for window in v.windows(2) {
            assert!(window[0] >= window[1], "{:?} is not descending", v);
        }
    }
}
```

### A5
```rust
/// Clamps a value to the range [min, max].
///
/// # Examples
///
/// ```
/// assert_eq!(clamp(5, 1, 10), 5);   // within range
/// assert_eq!(clamp(-5, 0, 10), 0);  // below min
/// assert_eq!(clamp(15, 0, 10), 10); // above max
/// assert_eq!(clamp(5, 5, 5), 5);    // min == max
/// ```
pub fn clamp(val: i32, min: i32, max: i32) -> i32 {
    if val < min { min }
    else if val > max { max }
    else { val }
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `#[automock]` generates `MockTraitName` automatically |
| 2 | **False** | Property tests run with MANY random inputs (hundreds by default) |
| 3 | **True** | `proptest` shrinks failing inputs to the smallest reproducing case |
| 4 | **True** | Doc examples in `///` blocks are compiled and tested |
| 5 | **True** | Manual mocks offer full customization; auto-mocks are more convenient |
| 6 | **True** | `.times(n)` in mockall verifies exact call count |

---

## 🏆 Lesson 70 Complete — Congratulations! 🎉

✅ 70 Rust lessons completed!  
✅ Beginner → Intermediate → Advanced  
✅ From variables to async, macros, FFI, and testing  

**You have a solid foundation in Rust.** The next phase covers real-world application development: Serde, web frameworks, databases, and projects! 🦀
