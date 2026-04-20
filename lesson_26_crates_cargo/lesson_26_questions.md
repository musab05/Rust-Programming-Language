# 🧪 Lesson 26 — Questions: Crates & Cargo (M3)

> **Lesson:** [lesson_26_crates_cargo.md](./lesson_26_crates_cargo.md)  
> **Answers:** [lesson_26_answers.md](./lesson_26_answers.md)

---

## Section A — Conceptual Questions

### Q1 — Binary vs Library
Which of the following are true? (may be multiple)
1. A library crate has a `fn main()`.
2. A binary crate's root is `src/main.rs`.
3. A package can have both a binary and a library crate.
4. You create a library with `cargo new mylib --lib`.
5. `cargo run` works on library crates.

### Q2 — Cargo.toml
What's wrong with this `Cargo.toml`?
```toml
[package]
name = "my app"
version = "1.0"

[dependencies]
serde = 1.0
```

### Q3 — SemVer
Given the dependency `serde = "1.2"`, which versions would Cargo allow?
1. `1.2.0`
2. `1.2.5`
3. `1.3.0`
4. `2.0.0`
5. `1.1.9`

### Q4 — Cargo.lock
True or False:
1. `Cargo.lock` should be committed to git for binary projects.
2. `Cargo.lock` is written manually by the developer.
3. `cargo update` regenerates `Cargo.lock`.
4. `Cargo.lock` contains version requirements like `"^1.0"`.

---

## Section B — Fix the Bugs

### Q5
This `Cargo.toml` has issues. Fix them:
```toml
[package]
name = my-project
version = "0.1"
edition = 2021

[dependencies]
serde = { version = "1.0" features = ["derive"] }
```

### Q6
```rust
// The developer forgot to add serde to Cargo.toml
use serde::{Serialize, Deserialize};

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    host: String,
    port: u16,
}

fn main() {
    let config = Config { host: "localhost".to_string(), port: 8080 };
    let json = serde_json::to_string(&config).unwrap();
    println!("{json}");
}
```
What needs to be added to `Cargo.toml` for this to compile?

---

## Section C — Write It Yourself

### Q7 — Serialize a struct (Roadmap Practice Task)
Assuming `serde` and `serde_json` are in your `Cargo.toml`, write a program that:
1. Defines a `Book` struct with fields: `title: String`, `author: String`, `pages: u32`, `rating: f64`
2. Creates two `Book` instances
3. Serializes them to JSON and prints the result
4. Takes a JSON string and deserializes it back into a `Book`
5. Prints the deserialized struct with `Debug`

### Q8 — Design a library
Sketch out (in pseudocode/Rust) a library crate called `calculator` that:
- Has `src/lib.rs` as the root
- Exposes public functions: `add`, `subtract`, `multiply`, `divide`
- `divide` returns `Result<f64, String>` (error on division by zero)
- Includes unit tests for all 4 functions

### Q9 — Cargo commands
For each task, write the cargo command:
1. Create a new binary project called "my_app"
2. Add the `rand` crate as a dependency
3. Build in release mode
4. Run tests that contain "math" in their name
5. View the dependency tree
6. Format all source code
7. Run the linter
8. Generate and open documentation

---

## Section D — Deep Understanding

### Q10 — True or False?
1. A package must contain at least one crate.
2. `cargo check` is faster than `cargo build` because it skips code generation.
3. `[dev-dependencies]` are included in the final binary.
4. Features allow conditional compilation of code.
5. `cargo add` requires you to manually edit `Cargo.toml`.
6. The `edition` field in `Cargo.toml` determines which Rust edition features are available.
7. External crates are automatically downloaded and compiled by Cargo.
8. `src/bin/tool.rs` creates an additional binary in your package.

---

*Cargo makes Rust development a breeze! 🦀*
