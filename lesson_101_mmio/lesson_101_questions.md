# 🧪 Lesson 101 — Questions: Memory-Mapped Registers & SVD2Rust (EM3)

> **Lesson:** [lesson_101_mmio.md](./lesson_101_mmio.md)  
> **Answers:** [lesson_101_answers.md](./lesson_101_answers.md)

---

## Section A — Conceptual

### Q1
What are memory-mapped registers? Why do peripherals (GPIO, UART, SPI) use fixed memory addresses instead of dedicated CPU instructions?

### Q2
Why must register access use `read_volatile` and `write_volatile` instead of normal pointer dereference? What optimizations could the compiler incorrectly apply without volatile?

### Q3
Explain what SVD2Rust does. What is the input, what is the output, and how does the output improve safety over raw pointer access?

---

## Section B — Write It Yourself

### Q4 — Raw GPIO toggle (Roadmap Practice Task)
Write a bare-metal function that toggles GPIO pin PA5 on an STM32F4 using raw volatile access:
1. Enable the GPIOA clock via `RCC_AHB1ENR` (set bit 0)
2. Set PA5 as output via `GPIOA_MODER` (bits [11:10] = 01)
3. Toggle PA5 via `GPIOA_ODR` (XOR bit 5)
4. Use `read_volatile` / `write_volatile` for all register access

### Q5 — Type-safe register wrapper
Write a `Register<T>` wrapper struct that encapsulates volatile access:
1. `new(addr: usize) -> Self`
2. `read(&self) -> T`
3. `write(&self, val: T)`
4. `modify(&self, f: impl FnOnce(T) -> T)` — read-modify-write
5. Explain why all methods should be `unsafe` or require `unsafe` construction

### Q6 — PAC-style comparison
Rewrite the Q4 raw GPIO toggle using PAC-style pseudo-code with named methods:
```rust
dp.RCC.ahb1enr.modify(|_, w| w.gpioaen().enabled());
dp.GPIOA.moder.modify(|_, w| w.moder5().output());
dp.GPIOA.odr.modify(|r, w| w.odr5().bit(!r.odr5().bit()));
```
Explain each line and why this is safer than the raw version.

---

## Section C — True or False?

### Q7
1. Memory-mapped registers are at fixed physical addresses defined by the chip vendor.
2. `read_volatile` may be optimized away by the compiler.
3. SVD files are XML descriptions provided by chip vendors.
4. PACs implement `embedded-hal` traits directly.
5. `.modify()` on a PAC register performs a read-modify-write operation.
6. `write_volatile` guarantees the write will reach the hardware register.

### Q8
Explain the full stack: SVD → PAC → HAL → Driver → Application. For each transition, what does the next layer add? Why not just use PAC registers directly in application code?

---

## 🎉🏆 ROADMAP COMPLETE! ALL 101 LESSONS DONE! 🦀

**From Hello World to bare-metal registers — congratulations!**
