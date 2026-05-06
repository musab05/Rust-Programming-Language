# ✅ Lesson 65 — Answers: Advanced Lifetimes (AL1)

---

## Section A

### A1 — ✅ Compiles
`first` returns `&'a str` tied to `s1`'s lifetime. `s2`'s lifetime (`'b`) is independent. Even after `s2` is dropped, `result` only borrows from `s1`. Output: `hello`.

### A2 — ❌ Won't compile
`pick` uses the SAME lifetime `'a` for both inputs AND the output. The result could borrow from either, so the compiler constrains it to the SHORTER of the two lifetimes. Since `s2` is dropped, `result` can't outlive that scope.

### A3 — ❌ Won't compile
`&s` is a reference to a local variable — it's not `'static`. The reference would dangle when `s` goes out of scope. Fix: `requires_static(s)` (move the owned String, which IS `'static`).

---

## Section B

### A4
```rust
fn extract_field<'data, 'schema>(
    data: &'data str,
    schema: &'schema str,
) -> &'data str {
    // Schema tells us where to look, but result borrows from data
    let end = schema.len().min(data.len());
    &data[..end]
}

fn main() {
    let data = String::from("Alice,30,Engineer");
    let result;
    {
        let schema = String::from("name");  // 4 chars
        result = extract_field(&data, &schema);
    }
    // schema is dropped, but result only borrows data
    println!("{result}");  // "Alic"
}
```

### A5
```rust
struct Processor {
    handler: Box<dyn for<'a> Fn(&'a str) -> String>,
}

impl Processor {
    fn new<F>(f: F) -> Self
    where F: for<'a> Fn(&'a str) -> String + 'static
    {
        Processor { handler: Box::new(f) }
    }

    fn process(&self, input: &str) -> String {
        (self.handler)(input)
    }
}

fn main() {
    let p = Processor::new(|s| format!("[processed] {s}"));
    println!("{}", p.process("hello"));
    println!("{}", p.process("world"));
}
```

### A6
```rust
struct LineIter<'a> {
    remaining: &'a str,
}

impl<'a> Iterator for LineIter<'a> {
    type Item = &'a str;

    fn next(&mut self) -> Option<&'a str> {
        if self.remaining.is_empty() {
            return None;
        }
        match self.remaining.find('\n') {
            Some(pos) => {
                let line = &self.remaining[..pos];
                self.remaining = &self.remaining[pos + 1..];
                Some(line)
            }
            None => {
                let line = self.remaining;
                self.remaining = "";
                Some(line)
            }
        }
    }
}

fn lines(text: &str) -> LineIter<'_> {
    LineIter { remaining: text }
}

fn main() {
    let text = "hello\nworld\nrust";
    for line in lines(text) {
        println!("Line: {line}");
    }
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `T: 'static` means T contains no non-static borrows — not that it lives forever |
| 2 | **True** | String literals (`"hello"`) have type `&'static str` |
| 3 | **True** | HRTB: the function works for any lifetime, not a specific one |
| 4 | **True** | `'b: 'a` means `'b` outlives `'a` |
| 5 | **True** | `tokio::spawn` requires `Future: Send + 'static` |
| 6 | **True** | `'_` is the anonymous lifetime — compiler infers it |

### A8
- **`T: 'static`** — The TYPE `T` has no non-`'static` borrows. An owned type like `String`, `i32`, or `Vec<u8>` satisfies this. It does NOT mean the value lives forever — it means it CAN live as long as needed because it owns all its data.
- **`&'static T`** — A REFERENCE that is valid for the entire program duration. The data it points to truly lives forever (like string literals in the binary or leaked memory).

Example: `String` satisfies `T: 'static`, but `&local_string` does NOT satisfy `&'static String`.

---

## 🏆 Lesson 65 Complete!

✅ Multiple lifetime parameters  
✅ Lifetime subtyping (`'b: 'a`)  
✅ Higher-Ranked Trait Bounds (`for<'a>`)  
✅ `'static` lifetime (owned vs referenced)  
✅ Lifetimes in closures and async  
✅ Common lifetime patterns  

**Next up:** Lesson 66 — Advanced Traits & Type-Level Programming 🦀
