# 🧪 Lesson 86 — Questions: DSTs (AT4)

> **Lesson:** [lesson_86_dst.md](./lesson_86_dst.md)  
> **Answers:** [lesson_86_answers.md](./lesson_86_answers.md)

---

## Section A — Compile or Not?

### Q1
```rust
fn print_it(s: str) { println!("{s}"); }
fn main() { print_it("hello"); }
```

### Q2
```rust
fn print_it<T: std::fmt::Display + ?Sized>(s: &T) { println!("{s}"); }
fn main() { print_it("hello"); }
```

---

## Section B — Write It Yourself

### Q3 — Fix the function (Roadmap Practice Task)
Fix `fn f(s: str)` so it compiles. Show 3 different valid signatures.

### Q4 — Fat pointer sizes
Write a program that prints the size of `&i32`, `&str`, `&[i32]`, `&dyn Display`, `Box<dyn Display>`, and `String`. Explain each.

---

## Section C — True or False?

### Q5
1. `str` is a Dynamically Sized Type.
2. All generic type parameters are implicitly `Sized`.
3. `?Sized` means the type must NOT be `Sized`.
4. `&str` is a fat pointer containing a data pointer and a length.
5. `dyn Trait` can live directly on the stack.
6. A struct can have at most one DST field, and it must be the last.

---

*DSTs: size matters — or sometimes, it doesn't! 🦀*
