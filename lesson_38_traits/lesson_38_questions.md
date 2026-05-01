# 🧪 Lesson 38 — Questions: Traits (T1)

> **Lesson:** [lesson_38_traits.md](./lesson_38_traits.md)  
> **Answers:** [lesson_38_answers.md](./lesson_38_answers.md)

---

## Section A — Predict: Compile or Not?

### Q1
```rust
trait Greet {
    fn hello(&self) -> String;
}

struct Dog;

fn main() {
    let d = Dog;
    println!("{}", d.hello());
}
```

### Q2
```rust
trait Greet {
    fn hello(&self) -> String;
}

struct Dog { name: String }

impl Greet for Dog {
    fn hello(&self) -> String {
        format!("Woof! I'm {}", self.name)
    }
}

fn main() {
    let d = Dog { name: "Rex".into() };
    println!("{}", d.hello());
}
```

### Q3
```rust
impl std::fmt::Display for Vec<i32> {
    fn fmt(&self, f: &mut std::fmt::Formatter) -> std::fmt::Result {
        write!(f, "my vec")
    }
}
```

---

## Section B — Write It Yourself

### Q4 — Summary trait (Roadmap Practice Task)
Define a `Summary` trait with a `summarize(&self) -> String` method. Create:
1. `Article` with fields: `title`, `author`, `content`
2. `Tweet` with fields: `username`, `content`, `likes`

Implement `Summary` for both. Write a function `print_summary(item: &impl Summary)` that prints the summary.

### Q5 — Shape trait with defaults
Define a `Shape` trait with:
- Required: `area(&self) -> f64`
- Default: `description(&self) -> String` that returns `"A shape with area X"`

Implement for `Circle` and `Rectangle`. Override `description` for `Circle`.

### Q6 — Multiple traits
Create `Printable` and `Saveable` traits. Implement both for a `Document` struct. Write a function that accepts something implementing both traits.

---

## Section C — Deep Understanding

### Q7 — True or False?
1. A trait can have both required and default methods.
2. `impl Trait` in return position can return different concrete types.
3. The orphan rule prevents implementing foreign traits on foreign types.
4. Default methods cannot call required methods on the same trait.
5. Trait inheritance means one trait can require another.

### Q8
Why does Rust have the orphan rule? What problem would arise without it?

---

*Traits define what types can DO — the foundation of Rust's type system! 🦀*
