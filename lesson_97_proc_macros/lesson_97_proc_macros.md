# 📘 Lesson 97 — Attribute & Function-like Macros (MA3)

> **Series:** Rust From Zero · Expert Level (Gap Fill)  
> **Roadmap ID:** MA3 · Category: 🔮 Macros  
> **Previous:** [Lesson 96 — Inline Assembly](../lesson_96_inline_asm/lesson_96_inline_asm.md)  
> **Next:** [Lesson 98 — SIMD & Intrinsics](../lesson_98_simd/lesson_98_simd.md)  
> **Practice:** [Questions](./lesson_97_questions.md) · [Answers](./lesson_97_answers.md)  
> **Practice Task:** Create a `#[log_calls]` attribute macro

---

## Table of Contents

1. [Three Kinds of Proc Macros](#1-three-kinds-of-proc-macros)
2. [Project Setup](#2-project-setup)
3. [Derive Macros (Review)](#3-derive-macros-review)
4. [Attribute Macros](#4-attribute-macros)
5. [Function-like Macros](#5-function-like-macros)
6. [Working with TokenStreams](#6-working-with-tokenstreams)
7. [Error Handling in Macros](#7-error-handling-in-macros)
8. [Summary Cheat Sheet](#8-summary-cheat-sheet)

---

## 1. Three Kinds of Proc Macros

| Kind | Syntax | Example |
|---|---|---|
| **Derive** | `#[derive(MyTrait)]` | Auto-implement traits |
| **Attribute** | `#[my_attr]` | Transform functions/structs |
| **Function-like** | `my_macro!(...)` | Custom syntax |

All proc macros:
- Live in a **separate crate** (`proc-macro = true`)
- Transform `TokenStream → TokenStream`
- Run at **compile time**

---

## 2. Project Setup

```
my_project/
├── Cargo.toml
├── src/main.rs
└── my_macros/            ← separate crate
    ├── Cargo.toml
    └── src/lib.rs
```

```toml
# my_macros/Cargo.toml
[package]
name = "my_macros"
version = "0.1.0"

[lib]
proc-macro = true

[dependencies]
quote = "1"
syn = { version = "2", features = ["full"] }
proc-macro2 = "1"
```

```toml
# Root Cargo.toml
[dependencies]
my_macros = { path = "./my_macros" }
```

---

## 3. Derive Macros (Review)

```rust
// my_macros/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(Describe)]
pub fn describe_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let name_str = name.to_string();

    let expanded = quote! {
        impl #name {
            pub fn describe() -> &'static str {
                concat!("This is a ", #name_str)
            }
        }
    };
    TokenStream::from(expanded)
}
```

```rust
// src/main.rs
use my_macros::Describe;

#[derive(Describe)]
struct User { name: String }

fn main() { println!("{}", User::describe()); }
// "This is a User"
```

---

## 4. Attribute Macros

Transform an entire item (function, struct, impl):

```rust
// my_macros/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn};

/// #[log_calls] — prints function name on entry/exit
#[proc_macro_attribute]
pub fn log_calls(_attr: TokenStream, item: TokenStream) -> TokenStream {
    let input = parse_macro_input!(item as ItemFn);
    let fn_name = &input.sig.ident;
    let fn_name_str = fn_name.to_string();
    let fn_block = &input.block;
    let fn_sig = &input.sig;
    let fn_vis = &input.vis;
    let fn_attrs = &input.attrs;

    let expanded = quote! {
        #(#fn_attrs)*
        #fn_vis #fn_sig {
            println!("→ Entering {}", #fn_name_str);
            let __result = (|| #fn_block)();
            println!("← Leaving {}", #fn_name_str);
            __result
        }
    };
    TokenStream::from(expanded)
}

/// #[timed] — measures execution time
#[proc_macro_attribute]
pub fn timed(_attr: TokenStream, item: TokenStream) -> TokenStream {
    let input = parse_macro_input!(item as ItemFn);
    let fn_name = &input.sig.ident;
    let fn_name_str = fn_name.to_string();
    let fn_block = &input.block;
    let fn_sig = &input.sig;
    let fn_vis = &input.vis;

    let expanded = quote! {
        #fn_vis #fn_sig {
            let __start = ::std::time::Instant::now();
            let __result = (|| #fn_block)();
            println!("[{}] {:?}", #fn_name_str, __start.elapsed());
            __result
        }
    };
    TokenStream::from(expanded)
}
```

```rust
// src/main.rs
use my_macros::{log_calls, timed};

#[log_calls]
fn greet(name: &str) -> String {
    format!("Hello, {name}!")
}

#[timed]
fn slow_work() -> u64 {
    (0..1_000_000u64).sum()
}

fn main() {
    println!("{}", greet("Alice"));
    println!("Result: {}", slow_work());
}
```

---

## 5. Function-like Macros

Custom syntax that looks like a function call:

```rust
// my_macros/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, LitStr, Token, parse::Parse, parse::ParseStream, punctuated::Punctuated};

struct SqlInput {
    query: LitStr,
}

impl Parse for SqlInput {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let query: LitStr = input.parse()?;
        Ok(SqlInput { query })
    }
}

/// sql!("SELECT * FROM users") — validates SQL at compile time
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as SqlInput);
    let query = input.query.value();

    // Basic compile-time SQL validation
    let upper = query.to_uppercase();
    if !upper.starts_with("SELECT") && !upper.starts_with("INSERT")
        && !upper.starts_with("UPDATE") && !upper.starts_with("DELETE") {
        return syn::Error::new(input.query.span(), "SQL must start with SELECT/INSERT/UPDATE/DELETE")
            .to_compile_error().into();
    }

    let expanded = quote! {
        {
            let query: &str = #query;
            query
        }
    };
    TokenStream::from(expanded)
}
```

```rust
// src/main.rs
use my_macros::sql;

fn main() {
    let q = sql!("SELECT * FROM users WHERE id = 1");
    println!("Query: {q}");
    // sql!("INVALID STUFF");  // ❌ compile error!
}
```

---

## 6. Working with TokenStreams

```rust
use proc_macro::TokenStream;
use quote::{quote, format_ident};
use syn::{parse_macro_input, DeriveInput, Data, Fields};

#[proc_macro_derive(Builder)]
pub fn builder_derive(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    let name = &input.ident;
    let builder_name = format_ident!("{}Builder", name);

    let fields = match &input.data {
        Data::Struct(data) => match &data.fields {
            Fields::Named(fields) => &fields.named,
            _ => panic!("Builder only supports named fields"),
        },
        _ => panic!("Builder only supports structs"),
    };

    let builder_fields = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! { #name: Option<#ty> }
    });

    let builder_methods = fields.iter().map(|f| {
        let name = &f.ident;
        let ty = &f.ty;
        quote! {
            pub fn #name(mut self, val: #ty) -> Self {
                self.#name = Some(val); self
            }
        }
    });

    let build_fields = fields.iter().map(|f| {
        let name = &f.ident;
        let name_str = name.as_ref().unwrap().to_string();
        quote! { #name: self.#name.ok_or(format!("missing field: {}", #name_str))? }
    });

    let none_fields = fields.iter().map(|f| {
        let name = &f.ident;
        quote! { #name: None }
    });

    let expanded = quote! {
        pub struct #builder_name { #(#builder_fields,)* }
        impl #builder_name {
            #(#builder_methods)*
            pub fn build(self) -> Result<#name, String> {
                Ok(#name { #(#build_fields,)* })
            }
        }
        impl #name {
            pub fn builder() -> #builder_name {
                #builder_name { #(#none_fields,)* }
            }
        }
    };
    TokenStream::from(expanded)
}
```

---

## 7. Error Handling in Macros

```rust
use syn::Error;

// Return compile errors with span information
fn validate(input: &DeriveInput) -> Result<(), Error> {
    match &input.data {
        Data::Struct(_) => Ok(()),
        _ => Err(Error::new_spanned(
            &input.ident,
            "MyMacro only supports structs"
        )),
    }
}

// In the macro:
#[proc_macro_derive(MyMacro)]
pub fn my_macro(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as DeriveInput);
    match validate(&input) {
        Ok(()) => { /* generate code */ quote!{}.into() }
        Err(e) => e.to_compile_error().into(),
    }
}
```

---

## 8. Summary Cheat Sheet

```
CRATE SETUP
────────────────────────────────────────────────────────────
[lib]
proc-macro = true
deps: syn, quote, proc-macro2

THREE TYPES
────────────────────────────────────────────────────────────
#[proc_macro_derive(Name)]         derive macro
#[proc_macro_attribute]            attribute macro
#[proc_macro]                      function-like macro

KEY CRATES
────────────────────────────────────────────────────────────
syn        parse TokenStream → AST
quote      generate TokenStream from Rust-like syntax
quote! { #var }   interpolate variables

PATTERN
────────────────────────────────────────────────────────────
fn my_macro(input: TokenStream) -> TokenStream {
    let parsed = parse_macro_input!(input as Type);
    let expanded = quote! { /* generated code */ };
    TokenStream::from(expanded)
}
```

---

## What's Next?

**Lesson 98 — SIMD & Intrinsics** — Parallel data processing with CPU vector instructions.

---

*Proc macros: code that writes code! 🦀*
