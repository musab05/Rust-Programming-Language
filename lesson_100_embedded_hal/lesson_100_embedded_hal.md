# 📘 Lesson 100 — Embedded HAL Pattern (EM2)

> **Series:** Rust From Zero · Expert Level (Gap Fill)  
> **Roadmap ID:** EM2 · Category: 🔩 Embedded  
> **Previous:** [Lesson 99 — #![no_std]](../lesson_99_no_std/lesson_99_no_std.md)  
> **Next:** [Lesson 101 — Memory-Mapped Registers](../lesson_101_mmio/lesson_101_mmio.md)  
> **Practice:** [Questions](./lesson_100_questions.md) · [Answers](./lesson_100_answers.md)  
> **Practice Task:** Write a portable LED driver using embedded-hal traits

---

## Table of Contents

1. [What Is embedded-hal?](#1-what-is-embedded-hal)
2. [The HAL Architecture](#2-the-hal-architecture)
3. [GPIO Traits](#3-gpio-traits)
4. [Writing Portable Drivers](#4-writing-portable-drivers)
5. [SPI and I2C Traits](#5-spi-and-i2c-traits)
6. [Delay Traits](#6-delay-traits)
7. [Real-World Driver Example](#7-real-world-driver-example)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. What Is embedded-hal?

`embedded-hal` defines **traits** for hardware peripherals — GPIO, SPI, I2C, UART, etc. Drivers code against traits, not specific hardware:

```
┌─────────────────────────────┐
│   Your Driver (portable)    │  ← uses embedded_hal traits
├─────────────────────────────┤
│       embedded-hal          │  ← trait definitions
├──────────┬──────────────────┤
│ stm32-hal│ nrf52-hal│rp2040 │  ← HAL implementations
├──────────┴──────────┴───────┤
│        Hardware (PAC)       │  ← register-level access
└─────────────────────────────┘
```

---

## 2. The HAL Architecture

```toml
[dependencies]
embedded-hal = "1.0"
```

The ecosystem layers:
1. **PAC** (Peripheral Access Crate) — auto-generated from SVD, register-level
2. **HAL** (Hardware Abstraction Layer) — implements embedded-hal traits
3. **Drivers** — portable, use traits only
4. **BSP** (Board Support Package) — board-specific pin configs

---

## 3. GPIO Traits

```rust
use embedded_hal::digital::{OutputPin, InputPin, StatefulOutputPin};

// The traits (simplified):
// pub trait OutputPin {
//     type Error;
//     fn set_high(&mut self) -> Result<(), Self::Error>;
//     fn set_low(&mut self) -> Result<(), Self::Error>;
// }
//
// pub trait InputPin {
//     type Error;
//     fn is_high(&mut self) -> Result<bool, Self::Error>;
//     fn is_low(&mut self) -> Result<bool, Self::Error>;
// }

// A portable LED abstraction
struct Led<P: OutputPin> {
    pin: P,
    is_on: bool,
}

impl<P: OutputPin> Led<P> {
    fn new(pin: P) -> Self { Led { pin, is_on: false } }

    fn on(&mut self) -> Result<(), P::Error> {
        self.pin.set_high()?;
        self.is_on = true;
        Ok(())
    }

    fn off(&mut self) -> Result<(), P::Error> {
        self.pin.set_low()?;
        self.is_on = false;
        Ok(())
    }

    fn toggle(&mut self) -> Result<(), P::Error> {
        if self.is_on { self.off() } else { self.on() }
    }
}
```

---

## 4. Writing Portable Drivers

```rust
use embedded_hal::digital::OutputPin;
use embedded_hal::delay::DelayNs;

/// A traffic light controller — works on ANY embedded platform
struct TrafficLight<R: OutputPin, Y: OutputPin, G: OutputPin> {
    red: R,
    yellow: Y,
    green: G,
}

impl<R: OutputPin, Y: OutputPin, G: OutputPin> TrafficLight<R, Y, G>
where
    R::Error: core::fmt::Debug,
    Y::Error: core::fmt::Debug,
    G::Error: core::fmt::Debug,
{
    fn new(red: R, yellow: Y, green: G) -> Self {
        TrafficLight { red, yellow, green }
    }

    fn all_off(&mut self) {
        self.red.set_low().unwrap();
        self.yellow.set_low().unwrap();
        self.green.set_low().unwrap();
    }

    fn go(&mut self) {
        self.all_off();
        self.green.set_high().unwrap();
    }

    fn caution(&mut self) {
        self.all_off();
        self.yellow.set_high().unwrap();
    }

    fn stop(&mut self) {
        self.all_off();
        self.red.set_high().unwrap();
    }

    fn cycle(&mut self, delay: &mut impl DelayNs) {
        self.stop();
        delay.delay_ms(3000);
        self.caution();
        delay.delay_ms(1000);
        self.go();
        delay.delay_ms(3000);
    }
}
```

---

## 5. SPI and I2C Traits

```rust
use embedded_hal::spi::SpiDevice;
use embedded_hal::i2c::I2c;

/// Temperature sensor driver (I2C)
struct TempSensor<I: I2c> {
    i2c: I,
    address: u8,
}

impl<I: I2c> TempSensor<I> {
    fn new(i2c: I, address: u8) -> Self { TempSensor { i2c, address } }

    fn read_temp(&mut self) -> Result<f32, I::Error> {
        let mut buf = [0u8; 2];
        self.i2c.read(self.address, &mut buf)?;
        let raw = ((buf[0] as u16) << 8) | buf[1] as u16;
        Ok(raw as f32 * 0.0625)
    }
}

/// Display driver (SPI)
struct Display<S: SpiDevice> {
    spi: S,
}

impl<S: SpiDevice> Display<S> {
    fn new(spi: S) -> Self { Display { spi } }

    fn write_command(&mut self, cmd: u8) -> Result<(), S::Error> {
        self.spi.write(&[cmd])
    }

    fn write_data(&mut self, data: &[u8]) -> Result<(), S::Error> {
        self.spi.write(data)
    }
}
```

---

## 6. Delay Traits

```rust
use embedded_hal::delay::DelayNs;

fn blink<P: OutputPin, D: DelayNs>(pin: &mut P, delay: &mut D, times: u32)
where P::Error: core::fmt::Debug
{
    for _ in 0..times {
        pin.set_high().unwrap();
        delay.delay_ms(500);
        pin.set_low().unwrap();
        delay.delay_ms(500);
    }
}
```

---

## 7. Real-World Driver Example

```rust
use embedded_hal::i2c::I2c;

const BME280_ADDR: u8 = 0x76;

struct Bme280<I: I2c> {
    i2c: I,
}

impl<I: I2c> Bme280<I> {
    fn new(i2c: I) -> Self { Bme280 { i2c } }

    fn chip_id(&mut self) -> Result<u8, I::Error> {
        let mut buf = [0u8];
        self.i2c.write_read(BME280_ADDR, &[0xD0], &mut buf)?;
        Ok(buf[0]) // should be 0x60
    }

    fn read_raw_temp(&mut self) -> Result<i32, I::Error> {
        let mut buf = [0u8; 3];
        self.i2c.write_read(BME280_ADDR, &[0xFA], &mut buf)?;
        let raw = ((buf[0] as i32) << 12) | ((buf[1] as i32) << 4) | ((buf[2] as i32) >> 4);
        Ok(raw)
    }
}
```

---

## 8. Summary Cheat Sheet

```
EMBEDDED-HAL TRAITS
────────────────────────────────────────────────────────────
OutputPin         set_high(), set_low()
InputPin          is_high(), is_low()
SpiDevice         write(), read(), transfer()
I2c               read(), write(), write_read()
DelayNs           delay_ns(), delay_ms(), delay_us()

ECOSYSTEM
────────────────────────────────────────────────────────────
PAC       register-level (auto-generated from SVD)
HAL       implements embedded-hal traits for a chip
Driver    portable, uses traits only
BSP       board-specific pin/peripheral config

PORTABILITY
────────────────────────────────────────────────────────────
fn driver<P: OutputPin>(pin: &mut P) { ... }
Works on STM32, nRF52, RP2040, ESP32 — any HAL!
```

---

## What's Next?

**Lesson 101 — Memory-Mapped Registers (SVD2Rust)** — The final lesson! Auto-generated register access.

---

*embedded-hal: write once, run on any microcontroller! 🦀*
