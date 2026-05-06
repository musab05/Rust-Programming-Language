# ✅ Lesson 61 — Answers: Unsafe Rust (U1)

---

## Section A

### A1 — ❌ Won't compile
Dereferencing a raw pointer requires an `unsafe` block. Error: `dereference of raw pointer is unsafe`.

### A2 — ✅ Compiles
Raw pointer created safely, dereferenced inside `unsafe`. Output: `42`.

### A3 — ❌ Won't compile
Accessing `static mut` requires an `unsafe` block. Error: `use of mutable static is unsafe`.

---

## Section B

### A4
```rust
struct SafeArray<T> {
    ptr: *mut T,
    len: usize,
}

impl<T: Default + Clone> SafeArray<T> {
    fn new(size: usize) -> Self {
        let mut data = vec![T::default(); size];
        let ptr = data.as_mut_ptr();
        std::mem::forget(data);  // prevent Vec from freeing memory
        SafeArray { ptr, len: size }
    }

    fn get(&self, index: usize) -> &T {
        assert!(index < self.len, "index {index} out of bounds (len {})", self.len);
        unsafe { &*self.ptr.add(index) }
    }

    fn set(&mut self, index: usize, value: T) {
        assert!(index < self.len, "index {index} out of bounds (len {})", self.len);
        unsafe { std::ptr::write(self.ptr.add(index), value); }
    }
}

impl<T> Drop for SafeArray<T> {
    fn drop(&mut self) {
        unsafe {
            let _ = Vec::from_raw_parts(self.ptr, self.len, self.len);
        }
    }
}

fn main() {
    let mut arr = SafeArray::<i32>::new(5);
    arr.set(0, 10);
    arr.set(4, 50);
    println!("arr[0] = {}", arr.get(0));  // 10
    println!("arr[4] = {}", arr.get(4));  // 50
}
```

### A5
```rust
fn split_at_mut(slice: &mut [i32], mid: usize) -> (&mut [i32], &mut [i32]) {
    let len = slice.len();
    assert!(mid <= len, "mid ({mid}) > len ({len})");
    let ptr = slice.as_mut_ptr();
    unsafe {
        (
            std::slice::from_raw_parts_mut(ptr, mid),
            std::slice::from_raw_parts_mut(ptr.add(mid), len - mid),
        )
    }
}

fn main() {
    let mut v = vec![1, 2, 3, 4, 5];
    let (a, b) = split_at_mut(&mut v, 2);
    a[0] = 10;
    b[0] = 30;
    println!("{:?} {:?}", a, b);  // [10, 2] [30, 4, 5]
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **False** | `unsafe` only unlocks 5 specific superpowers; borrow checker still applies |
| 2 | **True** | Creating is safe, dereferencing requires `unsafe` |
| 3 | **False** | `static mut` is a data race risk; accessing it is always `unsafe` |
| 4 | **True** | `unsafe impl` means YOU assert the invariant, not the compiler |
| 5 | **False** | `unsafe` has specific rules — only 5 additional capabilities |
| 6 | **True** | `AtomicT` and `Mutex` are thread-safe alternatives to `static mut` |

### A7
1. **Dereference raw pointer:** `unsafe { *ptr }`
2. **Call unsafe function:** `unsafe { dangerous_fn() }`
3. **Access mutable static:** `unsafe { COUNTER += 1 }`
4. **Implement unsafe trait:** `unsafe impl Send for MyType {}`
5. **Access union field:** `unsafe { my_union.field_a }`

---

## 🏆 Lesson 61 Complete!

**Next up:** [Lesson 62 — Declarative Macros](../lesson_62_macros/lesson_62_macros.md) 🦀
