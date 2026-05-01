# ✅ Lesson 40 — Answers: Generics in Functions (T3)

---

## Section A

### A1
```
42
hello
```
`identity` works with any type. Compiler monomorphizes into `identity_i32` and `identity_str`.

### A2
```
1
```
`first(1, 2)` compiles — both args are `i32`. `first(1, "two")` would **NOT** compile because both parameters must be the same type `T` — can't be `i32` and `&str` simultaneously.

### A3
```
["hi", "hi", "hi"]
```
`vec![item; count]` requires `Clone`. `&str` is `Copy` (which implies `Clone`), so it works.

---

## Section B

### A4 — Generic Stack
```rust
#[derive(Debug)]
struct Stack<T> {
    elements: Vec<T>,
}

impl<T> Stack<T> {
    fn new() -> Self {
        Stack { elements: Vec::new() }
    }

    fn push(&mut self, item: T) {
        self.elements.push(item);
    }

    fn pop(&mut self) -> Option<T> {
        self.elements.pop()
    }

    fn peek(&self) -> Option<&T> {
        self.elements.last()
    }

    fn is_empty(&self) -> bool {
        self.elements.is_empty()
    }
}

fn main() {
    // i32 stack
    let mut nums = Stack::new();
    nums.push(1);
    nums.push(2);
    nums.push(3);
    println!("Peek: {:?}", nums.peek());  // Some(3)
    println!("Pop:  {:?}", nums.pop());   // Some(3)
    println!("Empty: {}", nums.is_empty()); // false

    // String stack
    let mut words = Stack::new();
    words.push("hello".to_string());
    words.push("world".to_string());
    println!("Peek: {:?}", words.peek());  // Some("world")
}
```

### A5 — Generic min
```rust
fn min_of<T: PartialOrd>(a: T, b: T) -> T {
    if a <= b { a } else { b }
}

fn main() {
    println!("{}", min_of(10, 20));      // 10
    println!("{}", min_of(3.14, 2.71));  // 2.71
    println!("{}", min_of("b", "a"));    // a
}
```

### A6 — Generic contains
```rust
fn contains<T: PartialEq>(haystack: &[T], needle: &T) -> bool {
    for item in haystack {
        if item == needle {
            return true;
        }
    }
    false
}

fn main() {
    let nums = vec![1, 2, 3, 4, 5];
    println!("Has 3: {}", contains(&nums, &3));  // true
    println!("Has 9: {}", contains(&nums, &9));  // false

    let words = vec!["hello", "world"];
    println!("Has 'world': {}", contains(&words, &"world"));  // true
}
```

### A7 — Generic transform
```rust
fn transform<T, U, F>(items: Vec<T>, f: F) -> Vec<U>
where
    F: Fn(T) -> U,
{
    items.into_iter().map(f).collect()
}

fn main() {
    let nums = vec![1, 2, 3, 4, 5];

    let doubled = transform(nums.clone(), |x| x * 2);
    println!("{:?}", doubled);  // [2, 4, 6, 8, 10]

    let strings = transform(nums, |x| x.to_string());
    println!("{:?}", strings);  // ["1", "2", "3", "4", "5"]

    let words = vec!["hello", "world"];
    let lengths = transform(words, |w| w.len());
    println!("{:?}", lengths);  // [5, 5]
}
```

---

## Section C

### A8
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | Monomorphization eliminates runtime overhead — generics are zero-cost |
| 2 | **True** | The compiler generates a specialized copy for each concrete type |
| 3 | **True** | Type inference usually determines the type from usage context |
| 4 | **False** | They have very different meanings; `impl ?Sized` relaxes the `Sized` bound |
| 5 | **False** | Turbofish is only needed when the compiler can't infer the type |

### A9
Two problems:
1. `list[0]` tries to **move** out of a borrowed slice (needs `Copy`)
2. `item > max` requires `PartialOrd`

**Fix 1 — Add bounds:**
```rust
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut max = list[0];
    for &item in &list[1..] {
        if item > max { max = item; }
    }
    max
}
```

**Fix 2 — Return a reference:**
```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut max = &list[0];
    for item in &list[1..] {
        if item > max { max = item; }
    }
    max
}
```

### A10
**Zero-cost abstraction** means generics compile down to the same machine code as hand-written type-specific functions. Rust achieves this via **monomorphization** — the compiler generates specialized code for each concrete type at compile time.

**Comparison:**
- **Rust**: Compile-time specialization (monomorphization). No runtime dispatch, no boxing. The generic `Vec<i32>` is as fast as a hand-crafted int array.
- **Java/C#**: Type erasure (Java) or JIT specialization (C# value types). Java generics use `Object` at runtime with boxing/unboxing overhead. C# is better — it specializes value types at JIT time, but reference types still use shared code.

Rust's approach trades compile time and binary size for maximum runtime performance.

---

## 🏆 Lesson 40 Complete!

✅ Generic function syntax `fn<T>`  
✅ Multiple type parameters  
✅ Monomorphization and zero-cost abstraction  
✅ Generics with trait bounds  
✅ Turbofish syntax  
✅ Common generic patterns  
✅ Generic Stack\<T\> implementation  

**You've completed 40 lessons!** 🎉  
The Intermediate curriculum continues with:  
**Lesson 41 — Generics in Structs & Enums** 🦀
