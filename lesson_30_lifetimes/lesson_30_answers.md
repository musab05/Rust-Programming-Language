# ✅ Lesson 30 — Answers: Lifetimes Basics (O5)

---

## Section A

### A1 — ❌ Won't compile
`x` is dropped at the end of the inner block. `r` would be a dangling reference. Error: `x does not live long enough`.

### A2 — ✅ Compiles
`x` lives until the end of `main`. `r` borrows `x` and is used while `x` is still alive.

### A3 — ❌ Won't compile
`result` could point to `s2`, which is dropped at the end of the inner block. `println!("{result}")` is outside that scope. Error: `s2 does not live long enough`.

### A4 — ✅ Compiles
`result` is used **inside** the inner block (where `s2` is still alive). Both `s1` and `s2` are valid when `println!` runs.

### A5 — ✅ Compiles
`first` always returns `x` (which borrows from `s1`). The return type only depends on `x`'s lifetime (`'a`), not `_y`'s. `s1` is alive when `result` is used.

---

## Section B

### A6 — ✅ Elision handles it
One input reference, one output reference → Rule 2 applies. Equivalent to `fn first_word<'a>(s: &'a str) -> &'a str`.

### A7 — ❌ Needs annotation
Two input references, one output → rules don't apply. Must annotate: `fn longer<'a>(a: &'a str, b: &'a str) -> &'a str`.

### A8 — ✅ No annotation needed
No reference in the output → lifetimes not relevant for the return type.

### A9 — ✅ Elision handles it
One input reference, one output → Rule 2. Equivalent to `fn identity<'a>(s: &'a str) -> &'a str`.

### A10 — ✅ No annotation needed
Returns `String` (owned), not a reference. No lifetime needed.

---

## Section C

### A11
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}
```
Two input references → must annotate. Both inputs tied to the same lifetime `'a`.

### A12
```rust
// Can't return a reference to a local — return owned data instead
fn get_str() -> String {
    String::from("hello")
}
```
The `String` is created inside the function and would be dropped when the function returns. Returning a reference to it is impossible. Return `String` instead.

### A13
```rust
fn pick_first<'a>(a: &'a str, _b: &str) -> &'a str {
    a
}
```
Two input references → elision fails. Since we always return `a`, only `a`'s lifetime matters for the output.

---

## Section D

### A14 — Longest of two strings
```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() { x } else { y }
}

fn main() {
    // Test 1: Both in same scope
    let s1 = String::from("hello world");
    let s2 = String::from("hi");
    let result = longest(s1.as_str(), s2.as_str());
    println!("Longest: {result}");  // "hello world"

    // Test 2: One in inner scope — used within inner scope
    let s3 = String::from("Rust is great");
    {
        let s4 = String::from("Go");
        let result2 = longest(s3.as_str(), s4.as_str());
        println!("Longest: {result2}");  // "Rust is great"
    }

    // Test 3: Would fail if result used outside inner scope
    // let result3;
    // {
    //     let s5 = String::from("temp");
    //     result3 = longest(s3.as_str(), s5.as_str());
    // }
    // println!("{result3}");  // ❌ s5 doesn't live long enough
}
```

### A15 — First non-empty
```rust
fn first_non_empty<'a>(a: &'a str, b: &'a str) -> &'a str {
    if !a.is_empty() {
        a
    } else {
        b
    }
}

fn main() {
    println!("{}", first_non_empty("hello", "world"));  // "hello"
    println!("{}", first_non_empty("", "fallback"));    // "fallback"
    println!("{}", first_non_empty("", ""));            // ""
    println!("{}", first_non_empty("first", ""));       // "first"
}
```

### A16 — Trim and return
```rust
fn trim_start_matches_char<'a>(s: &'a str, c: char) -> &'a str {
    s.trim_start_matches(c)
}

fn main() {
    let text = "###hello###";
    let trimmed = trim_start_matches_char(text, '#');
    println!("'{trimmed}'");  // 'hello###'

    let text2 = "   spaces";
    let trimmed2 = trim_start_matches_char(text2, ' ');
    println!("'{trimmed2}'");  // 'spaces'

    let text3 = "no_match";
    let trimmed3 = trim_start_matches_char(text3, '#');
    println!("'{trimmed3}'");  // 'no_match'
}
```

---

## Section E

### A17 — True or False?
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Every reference has a lifetime, though most are inferred |
| 2 | **False** | Annotations describe relationships — they don't change actual lifetimes |
| 3 | **False** | `'static` means "valid for the entire program"; string literals are in the binary, not heap |
| 4 | **True** | Elision Rule 2: one input ref → output gets the same lifetime |
| 5 | **False** | Local variables are dropped when the function returns — can't reference them |
| 6 | **True** | String literals are embedded in the binary and live for the entire program |
| 7 | **False** | It means the output's lifetime is tied to `x`'s lifetime, but the output could be computed from anything — it's about validity, not origin. However, in practice the compiler checks that the return value actually comes from something with lifetime `'a` |
| 8 | **False** | Elision rules handle most cases automatically |

### A18 — Explain the error
`data` is a `Vec` allocated on the stack. It's dropped at `}`. `result = &data[0]` borrows an element of the vec. After the block, `data` is gone, so `result` would be a **dangling reference** — pointing to freed memory.

The compiler prevents this to avoid undefined behaviour. The fix is either:
1. Move `data` to the outer scope
2. Clone the value: `result = data[0];` (copy the i32, not borrow it)

### A19 — Elision rules

1. `fn first(s: &str) -> &str`
   - Rule 1: `fn first<'a>(s: &'a str) -> &str`
   - Rule 2: one input ref → `fn first<'a>(s: &'a str) -> &'a str` ✅ **Works**

2. `fn skip(s: &str, n: usize) -> &str`
   - Rule 1: `fn skip<'a>(s: &'a str, n: usize) -> &str` (usize has no lifetime)
   - Rule 2: one **reference** input → `fn skip<'a>(s: &'a str, n: usize) -> &'a str` ✅ **Works**

3. `fn combine(a: &str, b: &str) -> &str`
   - Rule 1: `fn combine<'a, 'b>(a: &'a str, b: &'b str) -> &str`
   - Rule 2: two input refs → doesn't apply
   - Rule 3: no `&self` → doesn't apply
   - ❌ **Fails** — must annotate manually

---

## 🏆 Lesson 30 Complete!

✅ What lifetimes are and why they exist  
✅ Lifetime annotation syntax: `'a`, `&'a str`  
✅ Lifetime annotations in function signatures  
✅ Multiple lifetime parameters  
✅ Lifetime elision rules (3 rules)  
✅ `'static` lifetime  
✅ Common lifetime errors and fixes  
✅ Mental model for thinking about lifetimes  

**🎉 You've completed the entire Beginner curriculum!**

The next lesson enters **Intermediate** territory:  
**Lesson 31 — Lifetimes in Structs & Advanced** 🦀
