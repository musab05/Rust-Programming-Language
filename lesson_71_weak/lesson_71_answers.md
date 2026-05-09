# ✅ Lesson 71 — Answers: Weak\<T\> (SP4)

---

## Section A

### A1 — `None`
The `Rc` is dropped at the end of the inner block. `weak.upgrade()` returns `None` because the value no longer exists.

### A2
```
strong=2, weak=1
```
`Rc::clone` increases strong count. `Rc::downgrade` increases weak count only.

---

## Section B

### A3
```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    name: String,
    parent: RefCell<Weak<Node>>,
    children: RefCell<Vec<Rc<Node>>>,
}

impl Node {
    fn new(name: &str) -> Rc<Self> {
        Rc::new(Node {
            name: name.into(),
            parent: RefCell::new(Weak::new()),
            children: RefCell::new(vec![]),
        })
    }

    fn add_child(parent: &Rc<Node>, child: &Rc<Node>) {
        *child.parent.borrow_mut() = Rc::downgrade(parent);
        parent.children.borrow_mut().push(Rc::clone(child));
    }

    fn path_to_root(&self) -> Vec<String> {
        let mut path = vec![self.name.clone()];
        let mut current = self.parent.borrow().upgrade();
        while let Some(node) = current {
            path.push(node.name.clone());
            current = node.parent.borrow().upgrade();
        }
        path.reverse();
        path
    }
}

fn main() {
    let root = Node::new("root");
    let child = Node::new("child");
    let leaf = Node::new("leaf");
    Node::add_child(&root, &child);
    Node::add_child(&child, &leaf);

    println!("Path: {:?}", leaf.path_to_root());  // ["root", "child", "leaf"]
}
```

### A4
```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct DLNode {
    value: i32,
    next: Option<Rc<RefCell<DLNode>>>,
    prev: Option<Weak<RefCell<DLNode>>>,
}

fn main() {
    let a = Rc::new(RefCell::new(DLNode { value: 1, next: None, prev: None }));
    let b = Rc::new(RefCell::new(DLNode { value: 2, next: None, prev: None }));
    let c = Rc::new(RefCell::new(DLNode { value: 3, next: None, prev: None }));

    a.borrow_mut().next = Some(Rc::clone(&b));
    b.borrow_mut().prev = Some(Rc::downgrade(&a));
    b.borrow_mut().next = Some(Rc::clone(&c));
    c.borrow_mut().prev = Some(Rc::downgrade(&b));

    // Forward: 1 → 2 → 3
    print!("Forward: ");
    let mut cur = Some(Rc::clone(&a));
    while let Some(node) = cur {
        print!("{} ", node.borrow().value);
        cur = node.borrow().next.as_ref().map(|n| Rc::clone(n));
    }
    println!();

    // Backward: 3 → 2 → 1
    print!("Backward: ");
    let mut cur = Some(Rc::clone(&c));
    while let Some(node) = cur {
        print!("{} ", node.borrow().value);
        cur = node.borrow().prev.as_ref().and_then(|w| w.upgrade());
    }
    println!();
}
```

---

## Section C

### A5
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Weak does NOT prevent dropping — that's the whole point |
| 2 | **True** | Returns `Some(Rc<T>)` if alive, `None` if dropped |
| 3 | **True** | Only strong count matters for drop |
| 4 | **False** | `downgrade()` increases the **weak** count, not strong |
| 5 | **True** | `Weak::new()` has no backing Rc — upgrade always returns None |
| 6 | **True** | Weak parent refs break the cycle that Rc would create |

---

## 🏆 Lesson 71 Complete!

**Next up:** [Lesson 72 — Type Aliases](../lesson_72_type_aliases/lesson_72_type_aliases.md) 🦀
