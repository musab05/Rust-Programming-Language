# ✅ Lesson 45 — Answers: Associated Types (T8)

---

## Section A

### A1
- **Associated type**: Each implementing type chooses ONE concrete type. `impl Iterator for Foo { type Item = u32; }` — Foo always produces `u32`.
- **Generic parameter**: A type can implement the trait multiple times with different type parameters. `impl ConvertTo<f64> for Foo` AND `impl ConvertTo<String> for Foo`.

### A2
Because any type should iterate over exactly ONE item type. If `Iterator<T>` were generic, `Vec<i32>` could implement `Iterator<i32>`, `Iterator<String>`, etc. — which makes no sense. Associated types enforce that there's one logical answer.

---

## Section B

### A3
```rust
trait Container {
    type Item;
    fn add(&mut self, item: Self::Item);
    fn get(&self, idx: usize) -> Option<&Self::Item>;
}

struct NumberList { data: Vec<f64> }

impl Container for NumberList {
    type Item = f64;
    fn add(&mut self, item: f64) { self.data.push(item); }
    fn get(&self, idx: usize) -> Option<&f64> { self.data.get(idx) }
}

fn main() {
    let mut list = NumberList { data: vec![] };
    list.add(3.14);
    list.add(2.71);
    println!("{:?}", list.get(0));  // Some(3.14)
}
```

### A4
```rust
trait Graph {
    type Node: std::fmt::Display;
    type Weight: std::fmt::Display;
    fn nodes(&self) -> &[Self::Node];
    fn edges(&self) -> &[(usize, usize, Self::Weight)];
}

struct SocialNetwork {
    users: Vec<String>,
    friendships: Vec<(usize, usize, u32)>,
}

impl Graph for SocialNetwork {
    type Node = String;
    type Weight = u32;
    fn nodes(&self) -> &[String] { &self.users }
    fn edges(&self) -> &[(usize, usize, u32)] { &self.friendships }
}

fn main() {
    let net = SocialNetwork {
        users: vec!["Alice".into(), "Bob".into(), "Charlie".into()],
        friendships: vec![(0, 1, 10), (1, 2, 5)],
    };
    for node in net.nodes() { println!("User: {node}"); }
    for (a, b, w) in net.edges() {
        println!("  {} ↔ {} (strength: {w})", net.nodes()[*a], net.nodes()[*b]);
    }
}
```

### A5
```rust
trait Pilot { fn name(&self) -> &str; }
trait Wizard { fn name(&self) -> &str; }

struct Hero;
impl Pilot for Hero { fn name(&self) -> &str { "Captain Hero" } }
impl Wizard for Hero { fn name(&self) -> &str { "Merlin Hero" } }

fn main() {
    let h = Hero;
    println!("{}", <Hero as Pilot>::name(&h));  // Captain Hero
    println!("{}", <Hero as Wizard>::name(&h)); // Merlin Hero
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Associated types fix one implementation per type |
| 2 | **True** | E.g., `type Item: Display + Clone;` constrains the implementor |
| 3 | **True** | `Self::Item` resolves to the concrete type chosen in the impl |
| 4 | **True** | That's the fully qualified syntax for disambiguation |
| 5 | **True** | `impl Add for MyType { type Output = MyType; }` |

---

## 🏆 Lesson 45 Complete!

**Next up:** [Lesson 46 — Workspaces](../lesson_46_workspaces/lesson_46_workspaces.md) 🦀
