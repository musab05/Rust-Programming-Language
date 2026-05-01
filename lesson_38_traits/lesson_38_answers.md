# ✅ Lesson 38 — Answers: Traits (T1)

---

## Section A

### A1 — ❌ Won't compile
`Dog` doesn't implement `Greet`. Defining a trait doesn't automatically implement it. Error: `method hello not found for Dog`.

### A2 — ✅ Compiles
`Dog` implements `Greet` with a concrete `hello` method. Output: `Woof! I'm Rex`.

### A3 — ❌ Won't compile
Orphan rule violation: both `Display` (from std) and `Vec<i32>` (from std) are foreign. You can't implement a foreign trait for a foreign type.

---

## Section B

### A4 — Summary trait
```rust
trait Summary {
    fn summarize(&self) -> String;
}

struct Article {
    title: String,
    author: String,
    content: String,
}

struct Tweet {
    username: String,
    content: String,
    likes: u32,
}

impl Summary for Article {
    fn summarize(&self) -> String {
        format!("{}, by {} — {}...",
            self.title, self.author,
            &self.content[..self.content.len().min(50)])
    }
}

impl Summary for Tweet {
    fn summarize(&self) -> String {
        format!("@{}: {} [♥ {}]", self.username, self.content, self.likes)
    }
}

fn print_summary(item: &impl Summary) {
    println!("📰 {}", item.summarize());
}

fn main() {
    let article = Article {
        title: "Rust 2024".into(),
        author: "Ferris".into(),
        content: "Rust continues to grow and evolve rapidly...".into(),
    };
    let tweet = Tweet {
        username: "rustlang".into(),
        content: "Rust 1.80 released!".into(),
        likes: 5000,
    };

    print_summary(&article);
    print_summary(&tweet);
}
```

### A5 — Shape with defaults
```rust
trait Shape {
    fn area(&self) -> f64;

    fn description(&self) -> String {
        format!("A shape with area {:.2}", self.area())
    }
}

struct Circle { radius: f64 }
struct Rectangle { width: f64, height: f64 }

impl Shape for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * self.radius * self.radius
    }

    fn description(&self) -> String {
        format!("Circle (r={:.1}, area={:.2})", self.radius, self.area())
    }
}

impl Shape for Rectangle {
    fn area(&self) -> f64 {
        self.width * self.height
    }
    // Uses default description
}

fn main() {
    let c = Circle { radius: 5.0 };
    let r = Rectangle { width: 4.0, height: 6.0 };

    println!("{}", c.description());  // Circle (r=5.0, area=78.54)
    println!("{}", r.description());  // A shape with area 24.00
}
```

### A6 — Multiple traits
```rust
trait Printable {
    fn print(&self);
}

trait Saveable {
    fn save(&self) -> Result<(), String>;
}

struct Document {
    title: String,
    content: String,
}

impl Printable for Document {
    fn print(&self) {
        println!("=== {} ===\n{}", self.title, self.content);
    }
}

impl Saveable for Document {
    fn save(&self) -> Result<(), String> {
        println!("Saving '{}'...", self.title);
        Ok(())
    }
}

fn process(item: &(impl Printable + Saveable)) {
    item.print();
    item.save().unwrap();
}

fn main() {
    let doc = Document {
        title: "Report".into(),
        content: "Important data here.".into(),
    };
    process(&doc);
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Traits can have required methods (no body) and default methods (with body) |
| 2 | **False** | `impl Trait` in return position can only return ONE concrete type |
| 3 | **True** | You must own either the trait or the type to implement it |
| 4 | **False** | Default methods CAN call required methods — that's a key feature |
| 5 | **True** | `trait Child: Parent` means implementing Child requires implementing Parent |

### A8
Without the orphan rule, two different crates could both implement the same trait for the same type (e.g., `Display for Vec<i32>`). The compiler wouldn't know which implementation to use, leading to **coherence conflicts**. The orphan rule ensures there's always exactly one implementation of any trait for any type, making dispatch unambiguous.

---

## 🏆 Lesson 38 Complete!

✅ Trait definition with required and default methods  
✅ Implementing traits for custom types  
✅ Traits as function parameters (impl Trait)  
✅ Returning impl Trait  
✅ The orphan rule and newtype workaround  
✅ Multiple traits on one type  
✅ Supertraits (trait inheritance)  

**Next up:** [Lesson 39 — Trait Bounds](../lesson_39_trait_bounds/lesson_39_trait_bounds.md) 🦀
