# ✅ Lesson 63 — Answers: Procedural Macros (MC2)

---

## Section A

### A1
1. **Derive macros**: `#[derive(MyMacro)]` — auto-implement traits for structs/enums
2. **Attribute macros**: `#[my_attr]` — transform any item (function, struct, etc.)
3. **Function-like macros**: `my_macro!(...)` — arbitrary token transformation

### A2
- **`syn`**: Parses a `TokenStream` into a structured AST (Abstract Syntax Tree). Lets you inspect struct names, fields, types, etc.
- **`quote`**: Generates a `TokenStream` from Rust-like templates with interpolation (`#name`). The reverse of `syn`.

### A3
Proc macros are compiler plugins that run during compilation. Rust requires them in a separate crate with `proc-macro = true` so the compiler can load and execute them as a dynamic library during the build of the dependent crate. The proc-macro crate is compiled first, then used by the compiler while building the consuming crate.

---

## Section B

### A4
```rust
use serde::{Serialize, Deserialize};

#[derive(Debug, Clone, PartialEq, Serialize, Deserialize)]
struct Config {
    host: String,
    port: u16,
    debug: bool,
}

fn main() {
    let config = Config {
        host: "localhost".into(),
        port: 8080,
        debug: true,
    };

    // Serialize
    let json = serde_json::to_string_pretty(&config).unwrap();
    println!("JSON:\n{json}");

    // Deserialize
    let parsed: Config = serde_json::from_str(&json).unwrap();
    assert_eq!(config, parsed);
    println!("Round-trip: {:?}", parsed);
}
```

### A5
```
my_summary_derive/
├── Cargo.toml
└── src/
    └── lib.rs
```

**Cargo.toml:**
```toml
[package]
name = "my_summary_derive"
version = "0.1.0"
edition = "2021"

[lib]
proc-macro = true

[dependencies]
syn = { version = "2", features = ["full"] }
quote = "1"
```

**lib.rs:**
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Summary)]
pub fn summary_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;

    let expanded = quote! {
        impl #name {
            pub fn summary(&self) -> String {
                format!("Instance of {}", stringify!(#name))
            }
        }
    };

    TokenStream::from(expanded)
}
```

---

## Section C

### A6
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | `Debug` is a derive proc macro built into the compiler |
| 2 | **True** | Proc macros are full Rust programs that run at compile time |
| 3 | **False** | Derive macros can implement ANY trait, including custom ones |
| 4 | **True** | `quote!` generates `TokenStream` from Rust-like template syntax |
| 5 | **True** | Attribute macros receive `(attr_args, item)` as two TokenStreams |
| 6 | **False** | Proc macros MUST be in a separate crate with `proc-macro = true` |

---

## 🏆 Lesson 63 Complete!

**Next up:** [Lesson 64 — FFI: Foreign Function Interface](../lesson_64_ffi/lesson_64_ffi.md) 🦀
