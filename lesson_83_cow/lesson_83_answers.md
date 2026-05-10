# ✅ Lesson 83 — Answers: Cow\<T\> (SP5)

---

## Section A

### A1
```
true false
```
- `"hello"` has no spaces → `Cow::Borrowed` → `matches!` = true
- `"hello world"` has spaces → `Cow::Owned("hello_world")` → `matches!(_, Borrowed)` = false

---

## Section B

### A2
```rust
use std::borrow::Cow;

fn ensure_lowercase(s: &str) -> Cow<str> {
    if s.chars().all(|c| !c.is_uppercase()) {
        Cow::Borrowed(s)
    } else {
        Cow::Owned(s.to_lowercase())
    }
}

fn main() {
    let r1 = ensure_lowercase("hello");
    let r2 = ensure_lowercase("Hello");
    println!("{r1} (borrowed: {})", matches!(r1, Cow::Borrowed(_)));  // true
    println!("{r2} (borrowed: {})", matches!(r2, Cow::Borrowed(_)));  // false
}
```

### A3
```rust
use std::borrow::Cow;

fn escape_html(s: &str) -> Cow<str> {
    if s.contains('&') || s.contains('<') || s.contains('>') {
        Cow::Owned(
            s.replace('&', "&amp;")
             .replace('<', "&lt;")
             .replace('>', "&gt;")
        )
    } else {
        Cow::Borrowed(s)
    }
}

fn main() {
    println!("{}", escape_html("safe text"));        // borrowed
    println!("{}", escape_html("<script>bad</script>")); // owned: &lt;script&gt;...
}
```

---

## Section C

### A4
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Borrowed variant holds `&'a B` — no heap allocation |
| 2 | **True** | Owned variant holds the owned type (e.g., String) |
| 3 | **True** | `to_mut()` clones borrowed data to get `&mut` access |
| 4 | **True** | Cow implements `Deref<Target = str>` |
| 5 | **False** | Cow adds complexity; only use when many calls skip allocation |
| 6 | **False** | If already Owned, `into_owned()` just moves — no clone |

---

## 🏆 Lesson 83 Complete!

**Next up:** [Lesson 84 — Async Streams & Channels](../lesson_84_async_streams/lesson_84_async_streams.md) 🦀
