# ✅ Lesson 51 — Answers: Box (SP1)

---

## Section A

### A1 — ❌ Won't compile
`List` contains itself directly — the compiler can't determine its size (infinite recursion). Error: `recursive type has infinite size`.

### A2 — ✅ Compiles
`Box<List>` is a pointer (fixed 8-byte size), breaking the infinite recursion.

### A3 — ❌ Won't compile
`Box<i32>` doesn't implement `Add<i32>`. Need to dereference: `*b + 1`.

---

## Section B

### A4
```rust
#[derive(Debug)]
enum Tree<T> {
    Empty,
    Leaf(T),
    Node { value: T, left: Box<Tree<T>>, right: Box<Tree<T>> },
}

impl<T: PartialOrd> Tree<T> {
    fn insert(self, val: T) -> Self {
        match self {
            Tree::Empty => Tree::Leaf(val),
            Tree::Leaf(v) => {
                if val < v {
                    Tree::Node { value: v, left: Box::new(Tree::Leaf(val)), right: Box::new(Tree::Empty) }
                } else {
                    Tree::Node { value: v, left: Box::new(Tree::Empty), right: Box::new(Tree::Leaf(val)) }
                }
            }
            Tree::Node { value, left, right } => {
                if val < value {
                    Tree::Node { value, left: Box::new(left.insert(val)), right }
                } else {
                    Tree::Node { value, left, right: Box::new(right.insert(val)) }
                }
            }
        }
    }

    fn contains(&self, target: &T) -> bool {
        match self {
            Tree::Empty => false,
            Tree::Leaf(v) => v == target,
            Tree::Node { value, left, right } => {
                if target == value { true }
                else if target < value { left.contains(target) }
                else { right.contains(target) }
            }
        }
    }

    fn count(&self) -> usize {
        match self {
            Tree::Empty => 0,
            Tree::Leaf(_) => 1,
            Tree::Node { left, right, .. } => 1 + left.count() + right.count(),
        }
    }
}

fn main() {
    let tree = Tree::Empty.insert(5).insert(3).insert(7).insert(1).insert(9);
    println!("Contains 3: {}", tree.contains(&3));  // true
    println!("Contains 4: {}", tree.contains(&4));  // false
    println!("Count: {}", tree.count());             // 5
}
```

### A5
```rust
use std::fmt;

enum List<T> { Cons(T, Box<List<T>>), Nil }

impl<T> List<T> {
    fn new() -> Self { List::Nil }
    fn push_front(self, val: T) -> Self { List::Cons(val, Box::new(self)) }
}

impl<T: Clone> List<T> {
    fn to_vec(&self) -> Vec<T> {
        let mut v = vec![];
        let mut current = self;
        loop {
            match current {
                List::Cons(val, next) => { v.push(val.clone()); current = next; }
                List::Nil => break,
            }
        }
        v
    }
}

impl<T: fmt::Display> fmt::Display for List<T> {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            List::Cons(val, next) => write!(f, "{val} → {next}"),
            List::Nil => write!(f, "∅"),
        }
    }
}

fn main() {
    let list = List::new().push_front(3).push_front(2).push_front(1);
    println!("{list}");                   // 1 → 2 → 3 → ∅
    println!("{:?}", list.to_vec());      // [1, 2, 3]
}
```

### A6
```rust
use std::fmt::Display;

fn main() {
    let items: Vec<Box<dyn Display>> = vec![
        Box::new(42),
        Box::new(String::from("hello")),
        Box::new(3.14_f64),
    ];
    for item in &items { println!("{item}"); }
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `Box::new(val)` allocates `val` on the heap |
| 2 | **False** | `Box` has single ownership; use `Rc`/`Arc` for shared |
| 3 | **True** | `Deref` allows `Box<T>` to behave like `&T` |
| 4 | **True** | The compiler must know every type's size; indirection gives a fixed pointer size |
| 5 | **True** | `drop()` immediately runs the destructor and frees memory |
| 6 | **True** | `Box<dyn Trait>` stores a data pointer + vtable pointer |

### A8
Without Box: `List = i32 + List = i32 + i32 + List = ...` — the size is infinite. The compiler can't allocate a struct of unknown/infinite size on the stack.

With Box: `List = i32 + Box<List> = i32 + pointer(8 bytes)` — fixed size. The `Box` is a pointer to heap memory, and the compiler knows a pointer is always 8 bytes regardless of what it points to.

---

## 🏆 Lesson 51 Complete!

**Next up:** [Lesson 52 — Rc & Arc](../lesson_52_rc_arc/lesson_52_rc_arc.md) 🦀
