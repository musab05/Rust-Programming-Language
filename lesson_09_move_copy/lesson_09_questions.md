# 🧪 Lesson 09 — Practice Questions: Move vs Copy

> Answers are in [`lesson_09_answers.md`](./lesson_09_answers.md)

---

## Section A — Copy or Move?

For each snippet, state whether assignment is a **Copy** or **Move** and whether the code compiles:

### Q1
```rust
let a: u8 = 255;
let b = a;
println!("{a} {b}");
```

### Q2
```rust
let s = String::from("hello");
let t = s;
println!("{s}");
```

### Q3
```rust
let arr = [1, 2, 3, 4, 5];
let arr2 = arr;
println!("{:?} {:?}", arr, arr2);
```

### Q4
```rust
let pair = (42, true);
let pair2 = pair;
println!("{:?} {:?}", pair, pair2);
```

### Q5
```rust
let mixed = (String::from("hi"), 42);
let mixed2 = mixed;
println!("{:?}", mixed);
```

### Q6
```rust
let v = vec![1.0, 2.0, 3.0];
let v2 = v;
println!("{v2:?}");
```

### Q7
```rust
#[derive(Debug, Clone, Copy)]
struct Color { r: u8, g: u8, b: u8 }

let red = Color { r: 255, g: 0, b: 0 };
let also_red = red;
println!("{red:?} {also_red:?}");
```

### Q8
```rust
#[derive(Debug, Clone)]
struct Named { name: String }

let n1 = Named { name: String::from("Alice") };
let n2 = n1;
println!("{n1:?}");
```

---

## Section B — Predict the Output

### Q9 — Clone vs Move

```rust
fn main() {
    let original = String::from("Rust");
    let cloned   = original.clone();
    let moved    = original;

    println!("cloned: {cloned}");
    println!("moved:  {moved}");
    // println!("{original}"); // would this compile?
}
```

**What prints? Would adding the commented println! compile?**

---

### Q10 — Drop order with Copy

```rust
struct Noisy(i32);
impl Drop for Noisy {
    fn drop(&mut self) { println!("drop {}", self.0); }
}

fn main() {
    let a = Noisy(1);
    let b = Noisy(2);
    let c = a; // move
    println!("middle");
}
```

**What is the output? Note: Noisy is NOT Copy.**

---

### Q11 — Partial move

```rust
fn main() {
    let data = (String::from("alpha"), String::from("beta"), 42u32);
    let (first, _, num) = data;
    println!("{first} {num}");
    println!("{}", data.1); // is this valid?
}
```

---

### Q12 — Reference is Copy

```rust
fn print_it(s: &String) {
    println!("{s}");
}

fn main() {
    let s = String::from("hello");
    let r1 = &s;
    let r2 = r1; // reference is Copy!
    print_it(r1);
    print_it(r2);
    println!("{s}");
}
```

**What is the output?**

---

## Section C — Fix the Errors

### Q13 — Use after move

```rust
fn process(v: Vec<i32>) -> i32 {
    v.iter().sum()
}

fn main() {
    let data = vec![1, 2, 3, 4, 5];
    let sum1 = process(data);
    let sum2 = process(data); // want to process again
    println!("{sum1} {sum2}");
}
```

**Fix with: (a) clone, (b) change process to borrow.**

---

### Q14 — Non-Copy struct

```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }

fn translate(p: Point, dx: i32, dy: i32) -> Point {
    Point { x: p.x + dx, y: p.y + dy }
}

fn main() {
    let origin = Point { x: 0, y: 0 };
    let moved1 = translate(origin, 1, 0);
    let moved2 = translate(origin, 0, 1); // error!
    println!("{:?} {:?}", moved1, moved2);
}
```

**Fix by deriving Copy (and Clone) on Point.**

---

### Q15 — Clone overkill

The following code compiles but clones unnecessarily. Refactor to use references:

```rust
fn get_length(s: String) -> usize {
    s.len()
}

fn get_uppercase(s: String) -> String {
    s.to_uppercase()
}

fn main() {
    let word = String::from("rustacean");
    let len = get_length(word.clone());
    let upper = get_uppercase(word.clone());
    println!("{word} has {len} chars, uppercase: {upper}");
}
```

---

## Section D — Write It Yourself

### Q16 — Implement Copy and Clone

Define a `Vector2D` struct with `x: f64, y: f64`. Derive both `Copy` and `Clone`. Write a function `scale(v: Vector2D, factor: f64) -> Vector2D`. In `main`, show that the original is still usable after passing to `scale`.

### Q17 — Clone-based deep copy

Define a `Matrix2x2` struct with `data: [[f64; 2]; 2]`. Arrays of `f64` are `Copy`, so the struct can be `Copy` too. Write a `fn transpose(m: Matrix2x2) -> Matrix2x2` that returns the transpose. Show that the original is still accessible after calling transpose.

### Q18 — mem::replace in practice

Write a struct `Queue<T>` with a single field `items: Vec<T>`. Implement a method `drain(&mut self) -> Vec<T>` that takes all items out of the queue, leaving it empty, using `std::mem::replace`. In `main`, demonstrate draining a `Queue<String>`.

---

## Section E — Deep Understanding

### Q19 — True or False?

1. `f32` and `f64` implement the `Copy` trait.
2. A `Vec<i32>` is `Copy` because `i32` is `Copy`.
3. A tuple `(String, i32)` is `Copy`.
4. `.clone()` always creates a heap allocation.
5. A struct with only `Copy` fields automatically implements `Copy`.
6. Moving a value is O(n) where n is the size of the heap data.
7. `&T` (a shared reference) is always `Copy`.
8. You can implement `Copy` without also implementing `Clone`.
9. After a partial move from a struct, the whole struct is invalid.
10. `mem::swap` copies both values, resulting in two independent copies.

---

### Q20 — Big Debug Challenge (7 bugs)

```rust
#[derive(Debug)]
struct Packet {
    id: u32,
    payload: String,
}

fn send(packet: Packet) {
    println!("Sending: {:?}", packet);
}

fn log(packet: Packet) {
    println!("Logging: {}", packet.id);
}

fn describe(id: u32) -> String {
    format!("Packet #{id}")
}

fn main() {
    let p = Packet { id: 1, payload: String::from("data") };

    send(p);
    log(p);              // Bug 1

    let desc = describe(42u32);
    let desc2 = desc;
    println!("{desc}");  // Bug 2

    let nums = vec![1, 2, 3];
    let nums2 = nums;
    let total: i32 = nums.iter().sum();  // Bug 3
    println!("{total}");

    let a = 10i32;
    let b = a;
    let c = a + b;       // Bug 4? Or fine?
    println!("{c}");

    let s1 = String::from("hello");
    let s2 = s1.clone;   // Bug 5
    println!("{s2}");

    let flag = true;
    let flag2 = flag;
    println!("{flag} {flag2}"); // Bug 6? Or fine?

    let pair = (String::from("x"), String::from("y"));
    let a = pair.0;
    let b = pair.0;      // Bug 7
    println!("{a} {b}");
}
```

---

*"Understanding move vs copy is understanding how data flows through your program." 🦀*
