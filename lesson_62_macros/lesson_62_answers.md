# ✅ Lesson 62 — Answers: Declarative Macros (MC1)

---

## Section A

### A1
```
8
```
The macro expands to `(3 + 1) * 2 = 8`. `$x:expr` captures the full expression `3 + 1`, so it's parenthesized in expansion.

### A2
```
4
```
Recursive counting: `1 + 1 + 1 + 1 + 0 = 4`.

---

## Section B

### A3
```rust
macro_rules! hashmap {
    ($($key:expr => $val:expr),* $(,)?) => {
        {
            let mut map = std::collections::HashMap::new();
            $( map.insert($key, $val); )*
            map
        }
    };
}

fn main() {
    let m = hashmap!{ "a" => 1, "b" => 2, "c" => 3 };
    println!("{:?}", m);
}
```

### A4
```rust
macro_rules! debug_print {
    ($($var:expr),+ $(,)?) => {
        $(
            println!("{} = {:?}", stringify!($var), $var);
        )+
    };
}

fn main() {
    let x = 42;
    let name = "Alice";
    let v = vec![1, 2, 3];
    debug_print!(x, name, v);
    // x = 42
    // name = "Alice"
    // v = [1, 2, 3]
}
```

### A5
```rust
macro_rules! min {
    ($a:expr, $b:expr) => {
        if $a < $b { $a } else { $b }
    };
    ($a:expr, $($rest:expr),+) => {
        min!($a, min!($($rest),+))
    };
}

fn main() {
    println!("{}", min!(5, 3));           // 3
    println!("{}", min!(5, 3, 8, 1, 4));  // 1
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Macros are expanded at **compile time** |
| 2 | **True** | `ident` matches identifiers like variable and function names |
| 3 | **True** | `*` means zero or more repetitions |
| 4 | **False** | Rust macros ARE hygienic — they don't leak scope |
| 5 | **True** | `stringify!` converts to a string literal without evaluating |
| 6 | **True** | `#[macro_export]` makes the macro public for other crates |

---

## 🏆 Lesson 62 Complete!

**Next up:** [Lesson 63 — Procedural Macros](../lesson_63_proc_macros/lesson_63_proc_macros.md) 🦀
