# ✅ Lesson 09 — Answers: Move vs Copy

> Questions are in [`lesson_09_questions.md`](./lesson_09_questions.md)

---

## Section A — Copy or Move?

### A1 — Copy ✅ Compiles

`u8` implements `Copy`. `b` is an independent copy. Both valid.

### A2 — Move ❌ Error

`String` is not `Copy`. `s` is moved into `t`. `println!("{s}")` fails: value borrowed after move.

### A3 — Copy ✅ Compiles

`[i32; 5]` — arrays of `Copy` types are `Copy`. Both `arr` and `arr2` are valid.

### A4 — Copy ✅ Compiles

`(i32, bool)` — tuple of `Copy` types is `Copy`. Both valid.

### A5 — Move ❌ Error

`(String, i32)` — contains a `String` (not `Copy`) so the tuple is not `Copy`. `mixed` is moved into `mixed2`. Cannot use `mixed` after.

### A6 — Move ✅ Partial compile

`Vec<f64>` is not `Copy`. `v` moves into `v2`. `v2` is valid. (Code as written only uses `v2`, so it compiles fine.)

### A7 — Copy ✅ Compiles

`Color` derives `Copy` and contains only `u8` fields (all `Copy`). Both `red` and `also_red` are valid.

### A8 — Move ❌ Error

`Named` derives `Clone` but NOT `Copy`. `n1` is moved into `n2`. `println!("{n1:?}")` fails.

---

## Section B — Predict the Output

### A9 — Clone vs Move

**Output:**

```
cloned: Rust
moved:  Rust
```

`original.clone()` makes a deep copy → `cloned` owns one String. Then `let moved = original` moves `original` into `moved`. `original` is now invalid.

Adding `println!("{original}")` would **NOT compile** — `original` was moved into `moved`.

---

### A10 — Drop order with Copy

**Output:**

```
middle
drop 2
drop 1
```

**Trace:**

- `a = Noisy(1)` created
- `b = Noisy(2)` created
- `c = a` — **moves** `a` (Noisy is not Copy) into `c`. `a` is invalid.
- `"middle"` prints
- End of scope: `c` (previously `a`, Noisy(1)) dropped? Wait — `c` holds the value that was in `a`, which is `Noisy(1)`. And `b` is `Noisy(2)`.
- Drop order (reverse of creation of current live variables: `b`, then `c`):
  - `b` (Noisy(2)) dropped → "drop 2"
  - `c` (Noisy(1)) dropped → "drop 1"

Note: `a` is not dropped again because ownership moved to `c` — only `c` runs the destructor.

---

### A11 — Partial move

```
alpha 42
```

`data.1` (`String::from("beta")`) was NOT moved (the `_` in the pattern ignores it — it stays in `data`). So `data.1` is still valid.

Wait — in destructuring `let (first, _, num) = data`, the `_` pattern does NOT bind the value — but for a non-Copy type, does it move it? In Rust, `_` specifically does NOT move — it's a wildcard that discards the value without binding. For Copy types it copies, for non-Copy types `_` neither moves nor binds. After this destructuring, `data.1` is still valid.

**Output:**

```
alpha 42
beta
```

---

### A12 — Reference is Copy

**Output:**

```
hello
hello
hello
```

`&String` is a shared reference and implements `Copy`. `let r2 = r1` copies the reference — both `r1` and `r2` are valid references to `s`. All three `println!` succeed.

---

## Section C — Fix the Errors

### A13 — Use after move

```rust
// Fix (a): clone before each call
fn process(v: Vec<i32>) -> i32 { v.iter().sum() }
fn main() {
    let data = vec![1, 2, 3, 4, 5];
    let sum1 = process(data.clone());
    let sum2 = process(data);
    println!("{sum1} {sum2}"); // 15 15
}

// Fix (b): change to borrow — better!
fn process(v: &[i32]) -> i32 { v.iter().sum() }
fn main() {
    let data = vec![1, 2, 3, 4, 5];
    let sum1 = process(&data);
    let sum2 = process(&data);
    println!("{sum1} {sum2}"); // 15 15
}
```

---

### A14 — Non-Copy struct

```rust
#[derive(Debug, Clone, Copy)]  // Add Copy and Clone
struct Point { x: i32, y: i32 }

fn translate(p: Point, dx: i32, dy: i32) -> Point {
    Point { x: p.x + dx, y: p.y + dy }
}

fn main() {
    let origin = Point { x: 0, y: 0 };
    let moved1 = translate(origin, 1, 0); // Copy — origin still valid
    let moved2 = translate(origin, 0, 1); // ✅ Copy again
    println!("{:?} {:?}", moved1, moved2);
}
```

---

### A15 — Clone overkill

```rust
// ✅ Refactored to use references
fn get_length(s: &str) -> usize {
    s.len()
}

fn get_uppercase(s: &str) -> String {
    s.to_uppercase()
}

fn main() {
    let word = String::from("rustacean");
    let len   = get_length(&word);    // borrow
    let upper = get_uppercase(&word); // borrow
    println!("{word} has {len} chars, uppercase: {upper}");
}
```

No clones needed! The functions borrow `&str` (a string slice), which works with any string type.

---

## Section D — Write It Yourself

### A16 — Implement Copy and Clone

```rust
#[derive(Debug, Clone, Copy)]
struct Vector2D {
    x: f64,
    y: f64,
}

fn scale(v: Vector2D, factor: f64) -> Vector2D {
    Vector2D { x: v.x * factor, y: v.y * factor }
}

fn main() {
    let v = Vector2D { x: 3.0, y: 4.0 };
    let scaled = scale(v, 2.0); // v is COPIED into scale (Copy trait)
    println!("Original: {:?}", v);  // ✅ still valid
    println!("Scaled:   {:?}", scaled);
}
```

---

### A17 — Matrix2x2 Transpose

```rust
#[derive(Debug, Clone, Copy)]
struct Matrix2x2 {
    data: [[f64; 2]; 2],
}

fn transpose(m: Matrix2x2) -> Matrix2x2 {
    Matrix2x2 {
        data: [
            [m.data[0][0], m.data[1][0]],
            [m.data[0][1], m.data[1][1]],
        ]
    }
}

fn main() {
    let m = Matrix2x2 { data: [[1.0, 2.0], [3.0, 4.0]] };
    let t = transpose(m); // Copy — m still valid
    println!("Original:  {:?}", m.data);
    println!("Transposed:{:?}", t.data);
}
```

---

### A18 — mem::replace drain

```rust
use std::mem;

struct Queue<T> {
    items: Vec<T>,
}

impl<T> Queue<T> {
    fn new() -> Self { Queue { items: Vec::new() } }
    fn push(&mut self, item: T) { self.items.push(item); }

    fn drain(&mut self) -> Vec<T> {
        mem::replace(&mut self.items, Vec::new())
        // puts an empty Vec in self.items, returns the old one
    }
}

fn main() {
    let mut q: Queue<String> = Queue::new();
    q.push(String::from("first"));
    q.push(String::from("second"));
    q.push(String::from("third"));

    let drained = q.drain();
    println!("Drained: {:?}", drained);
    println!("Queue empty: {}", q.items.is_empty());
}
```

---

## Section E — Deep Understanding

### A19 — True or False?

| #   | Statement                                             | Answer    | Explanation                                                                                                   |
| --- | ----------------------------------------------------- | --------- | ------------------------------------------------------------------------------------------------------------- |
| 1   | `f32`/`f64` implement `Copy`                          | **True**  | All primitive numeric types implement `Copy`                                                                  |
| 2   | `Vec<i32>` is `Copy` because `i32` is `Copy`          | **False** | `Vec<T>` owns heap data — cannot be `Copy` regardless of `T`                                                  |
| 3   | `(String, i32)` is `Copy`                             | **False** | `String` is not `Copy`, so the tuple isn't either                                                             |
| 4   | `.clone()` always allocates on heap                   | **False** | For `Copy` types, `.clone()` just copies stack bytes — no heap needed. But for heap types like `String`, yes. |
| 5   | Struct with only `Copy` fields auto-implements `Copy` | **False** | You must explicitly `#[derive(Copy)]` — it's not automatic                                                    |
| 6   | Moving a value is O(n) for heap size n                | **False** | Move is O(1) — only the stack representation (ptr/len/cap) is copied                                          |
| 7   | `&T` is always `Copy`                                 | **True**  | Shared references are always `Copy`                                                                           |
| 8   | Can implement `Copy` without `Clone`                  | **False** | `Copy: Clone` — implementing `Copy` requires `Clone`                                                          |
| 9   | After partial move, whole struct invalid              | **True**  | You can't use the struct as a whole, though untouched fields may still be accessible                          |
| 10  | `mem::swap` creates two independent copies            | **False** | `swap` exchanges the VALUES — it's still one value per binding, just swapped                                  |

---

### A20 — Big Debug Challenge

**Bug 1:** `send(p)` moves `p`. `log(p)` tries to use already-moved `p`. Fix: either use references, clone, or restructure.

**Bug 2:** `desc` is `String` (not `Copy`), moved into `desc2`. `println!("{desc}")` fails.

**Bug 3:** `nums` moved into `nums2`. `nums.iter().sum()` uses moved value.

**Bug 4:** NOT a bug. `i32` is `Copy`. `a` is still valid after `let b = a`. `a + b = 20` is fine.

**Bug 5:** `s1.clone` without `()` is a method reference, not a call. Should be `s1.clone()`.

**Bug 6:** NOT a bug. `bool` is `Copy`. Both `flag` and `flag2` are valid.

**Bug 7:** `pair.0` is moved into `a`. Trying to move `pair.0` again into `b` fails — it was already moved.

```rust
// ✅ Corrected program
#[derive(Debug)]
struct Packet {
    id: u32,
    payload: String,
}

fn send(packet: &Packet) {       // borrow
    println!("Sending: {:?}", packet);
}

fn log(packet: &Packet) {        // borrow
    println!("Logging: {}", packet.id);
}

fn describe(id: u32) -> String {
    format!("Packet #{id}")
}

fn main() {
    let p = Packet { id: 1, payload: String::from("data") };
    send(&p);                    // fix 1: borrow
    log(&p);                     // fix 1: borrow

    let desc = describe(42u32);
    let desc2 = desc.clone();    // fix 2: clone so desc is still valid
    println!("{desc}");

    let nums = vec![1, 2, 3];
    let nums2 = nums.clone();    // fix 3: clone so nums is still valid
    let total: i32 = nums.iter().sum();
    println!("{total}");
    let _ = nums2;

    let a = 10i32;
    let b = a;
    let c = a + b;               // fine — i32 is Copy
    println!("{c}");

    let s1 = String::from("hello");
    let s2 = s1.clone();         // fix 5: add ()
    println!("{s2}");

    let flag = true;
    let flag2 = flag;
    println!("{flag} {flag2}");  // fine — bool is Copy

    let pair = (String::from("x"), String::from("y"));
    let a = pair.0.clone();      // fix 7: clone pair.0
    let b = pair.0;              // now move original
    println!("{a} {b}");
}
```

---

## 🏆 Lesson Complete!

You've mastered:

- ✅ The Copy trait — which types copy and why
- ✅ Move semantics — what moves, what stays, O(1) cost
- ✅ Why heap-owning types cannot be Copy (double-free prevention)
- ✅ Clone — explicit deep copy, when to use it
- ✅ Partial moves from structs and tuples
- ✅ Moves in pattern matching
- ✅ `mem::swap` and `mem::replace` for in-place modification
- ✅ Preferring borrowing over cloning for efficiency

**Up next:** [Lesson 10 — References](../lesson_10_references_borrowing/lesson_10_references.md) 🦀
