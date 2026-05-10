# ✅ Lesson 100 — Answers: Embedded HAL Pattern (EM2)

---

## Section A

### A1
`embedded-hal` is a crate that defines **hardware abstraction traits** (`OutputPin`, `InputPin`, `SpiDevice`, `I2c`, `DelayNs`, etc.) for embedded peripherals.

It defines traits instead of implementations because:
- **Portability:** A driver written against `OutputPin` works on STM32, nRF52, RP2040, ESP32, or any chip that implements the trait
- **Separation of concerns:** Hardware vendors implement the traits for their chips; driver authors code against the traits
- **No vendor lock-in:** Switching chips only requires changing the HAL crate, not rewriting drivers

Without this pattern, every sensor driver would need a separate version for every microcontroller.

### A2
| Layer | Responsibility | Example |
|-------|---------------|---------|
| **PAC** (Peripheral Access Crate) | Raw register access, auto-generated from SVD files. Type-safe but low-level. | `stm32f4xx-pac` |
| **HAL** (Hardware Abstraction Layer) | Implements `embedded-hal` traits using the PAC. Chip-specific but provides a standard API. | `stm32f4xx-hal` |
| **Driver** | Portable device drivers written against `embedded-hal` traits. Chip-agnostic. | `bme280` sensor driver |
| **BSP** (Board Support Package) | Board-specific configuration — which pins are connected to LEDs, buttons, etc. | `stm32f4-discovery` |

Stack: `App → Driver → HAL → PAC → Hardware`

### A3
GPIO operations can fail in several scenarios:
- **I2C/SPI GPIO expanders** — the pin is on an external chip connected via a bus; the communication can fail
- **Shared bus errors** — if the GPIO peripheral is accessed through a shared resource that's locked
- **Virtual/software GPIO** — mock implementations for testing may simulate failures
- **Runtime permission errors** — on Linux GPIO (`/sys/class/gpio`), permission issues cause failures

The `Result` return type makes the API robust for ALL implementations, not just direct on-chip GPIO.

---

## Section B

### A4
```rust
use embedded_hal::digital::OutputPin;

pub struct Led<P: OutputPin> {
    pin: P,
    is_on: bool,
}

impl<P: OutputPin> Led<P> {
    pub fn new(pin: P) -> Self {
        Led { pin, is_on: false }
    }

    pub fn on(&mut self) -> Result<(), P::Error> {
        self.pin.set_high()?;
        self.is_on = true;
        Ok(())
    }

    pub fn off(&mut self) -> Result<(), P::Error> {
        self.pin.set_low()?;
        self.is_on = false;
        Ok(())
    }

    pub fn toggle(&mut self) -> Result<(), P::Error> {
        if self.is_on {
            self.off()
        } else {
            self.on()
        }
    }

    pub fn is_on(&self) -> bool {
        self.is_on
    }
}
```
This `Led` works on any microcontroller — you just pass in whatever pin type your HAL provides. On STM32: `Led::new(gpioa.pa5.into_push_pull_output())`. On RP2040: `Led::new(pins.gpio25.into_push_pull_output())`.

### A5
```rust
use embedded_hal::digital::OutputPin;
use embedded_hal::delay::DelayNs;

pub struct TrafficLight<R, Y, G>
where
    R: OutputPin,
    Y: OutputPin,
    G: OutputPin,
{
    red: R,
    yellow: Y,
    green: G,
}

impl<R, Y, G> TrafficLight<R, Y, G>
where
    R: OutputPin,
    Y: OutputPin,
    G: OutputPin,
    R::Error: core::fmt::Debug,
    Y::Error: core::fmt::Debug,
    G::Error: core::fmt::Debug,
{
    pub fn new(red: R, yellow: Y, green: G) -> Self {
        TrafficLight { red, yellow, green }
    }

    pub fn all_off(&mut self) {
        self.red.set_low().unwrap();
        self.yellow.set_low().unwrap();
        self.green.set_low().unwrap();
    }

    pub fn stop(&mut self) {
        self.all_off();
        self.red.set_high().unwrap();
    }

    pub fn caution(&mut self) {
        self.all_off();
        self.yellow.set_high().unwrap();
    }

    pub fn go(&mut self) {
        self.all_off();
        self.green.set_high().unwrap();
    }

    pub fn cycle(&mut self, delay: &mut impl DelayNs) {
        self.stop();
        delay.delay_ms(3000);   // Red for 3 seconds
        self.caution();
        delay.delay_ms(1000);   // Yellow for 1 second
        self.go();
        delay.delay_ms(3000);   // Green for 3 seconds
    }
}
```

### A6
```rust
use embedded_hal::i2c::I2c;

pub struct TempSensor<I: I2c> {
    i2c: I,
    address: u8,
}

impl<I: I2c> TempSensor<I> {
    pub fn new(i2c: I, address: u8) -> Self {
        TempSensor { i2c, address }
    }

    /// Read raw 16-bit temperature value from the sensor.
    pub fn read_raw(&mut self) -> Result<u16, I::Error> {
        let mut buf = [0u8; 2];
        self.i2c.read(self.address, &mut buf)?;
        let raw = ((buf[0] as u16) << 8) | buf[1] as u16;
        Ok(raw)
    }

    /// Read temperature in Celsius (assuming 0.0625°C per LSB).
    pub fn read_celsius(&mut self) -> Result<f32, I::Error> {
        let raw = self.read_raw()?;
        Ok(raw as f32 * 0.0625)
    }
}
```
**Why it works on ANY microcontroller:** The driver is generic over `I: I2c` — it doesn't know or care which chip it's running on. It only calls `i2c.read()`, which is defined by the `embedded-hal` trait. As long as the chip's HAL implements `I2c`, this driver works — whether it's STM32, nRF52, RP2040, or ESP32. Zero code changes needed.

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | embedded-hal is pure trait definitions — no hardware code |
| 2 | **True** | PACs are generated from SVD files and provide register-level access |
| 3 | **False** | Drivers using embedded-hal traits are **portable** — they work on any chip |
| 4 | **True** | `DelayNs` provides `delay_ms()`, `delay_us()`, and `delay_ns()` |
| 5 | **True** | BSPs map physical board features (LEDs, buttons) to HAL pin types |
| 6 | **True** | Each trait has `type Error` — implementations define their own error types |

### A8
embedded-hal drivers have "write once, run anywhere" portability because they're coded against **trait abstractions**, not hardware specifics. The driver calls `pin.set_high()`, not `write_register(0x4002_0014, 1 << 5)`.

**To move from STM32 to Raspberry Pi Pico, you change:**
1. The **HAL crate** dependency: `stm32f4xx-hal` → `rp2040-hal`
2. The **pin initialization code** in `main()`: different pin names and peripheral setup
3. The **linker script** and target triple

**What you DON'T change:**
- The driver code itself — `Led<P: OutputPin>` compiles unchanged
- The driver's logic, error handling, or API
- Any driver that uses `I2c`, `SpiDevice`, or `DelayNs` traits

The driver is decoupled from hardware — only the wiring code in `main()` is board-specific.

---

## 🏆 Lesson 100 Complete!

**Next up:** [Lesson 101 — Memory-Mapped Registers & SVD2Rust](../lesson_101_mmio/lesson_101_mmio.md) — the FINAL lesson! 🎉🦀
