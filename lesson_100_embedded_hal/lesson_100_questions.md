# 🧪 Lesson 100 — Questions: Embedded HAL Pattern (EM2)

> **Lesson:** [lesson_100_embedded_hal.md](./lesson_100_embedded_hal.md)  
> **Answers:** [lesson_100_answers.md](./lesson_100_answers.md)

---

## Section A — Conceptual

### Q1
What is `embedded-hal` and why does it define **traits** instead of implementations? What problem does this solve?

### Q2
Explain the four layers of the embedded Rust ecosystem: PAC, HAL, Driver, BSP. What does each layer provide?

### Q3
Why does `OutputPin::set_high()` return `Result<(), Self::Error>` instead of just `()`? In what scenario could setting a GPIO pin fail?

---

## Section B — Write It Yourself

### Q4 — Portable LED driver (Roadmap Practice Task)
Write a generic `Led<P: OutputPin>` struct with:
1. `new(pin: P) -> Self`
2. `on(&mut self) -> Result<(), P::Error>` — drives pin high
3. `off(&mut self) -> Result<(), P::Error>` — drives pin low
4. `toggle(&mut self) -> Result<(), P::Error>` — switches state
5. `is_on(&self) -> bool` — returns current state

### Q5 — Traffic light controller
Write a `TrafficLight<R, Y, G>` (all `OutputPin`) with:
1. `all_off()`, `stop()` (red), `caution()` (yellow), `go()` (green)
2. A `cycle(&mut self, delay: &mut impl DelayNs)` method that runs one full red→yellow→green cycle with timing
3. Make it generic over the delay type too

### Q6 — I2C temperature sensor
Write a generic `TempSensor<I: I2c>` driver that:
1. Stores the I2C bus and device address
2. Has `read_raw(&mut self) -> Result<u16, I::Error>` that reads 2 bytes
3. Has `read_celsius(&mut self) -> Result<f32, I::Error>` that converts the raw reading
4. Explain why this driver works on ANY microcontroller with an I2C peripheral

---

## Section C — True or False?

### Q7
1. `embedded-hal` defines traits, not implementations.
2. PAC stands for "Peripheral Access Crate" and provides register-level access.
3. Drivers written using `embedded-hal` traits are chip-specific.
4. `DelayNs` provides `delay_ms()` and `delay_us()` methods.
5. A BSP (Board Support Package) configures pins and peripherals for a specific board.
6. `embedded-hal` version 1.0 uses associated error types on each trait.

### Q8
Explain why embedded-hal drivers are said to have "write once, run anywhere" portability. What would you have to change to move a driver from an STM32 to a Raspberry Pi Pico?

---

*Write once, run on any microcontroller! 🦀*
