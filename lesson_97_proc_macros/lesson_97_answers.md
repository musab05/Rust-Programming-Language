# ✅ Lesson 97 — Answers: Attribute & Function-like Macros (MA3)

---

## Section A

### A1
| Kind | Attribute | Signature |
|------|-----------|-----------|
| **Derive** | `#[proc_macro_derive(Name)]` | `fn(TokenStream) -> TokenStream` |
| **Attribute** | `#[proc_macro_attribute]` | `fn(TokenStream, TokenStream) -> TokenStream` (attrs, item) |
| **Function-like** | `#[proc_macro]` | `fn(TokenStream) -> TokenStream` |

### A2
Proc macros are **compiler plugins** — they run during compilation, not at runtime. The Rust compiler loads the proc-macro crate as a dynamic library (`.dll`/`.so`) during compilation. This requires a separate compilation unit because:
1. The macro code must be **fully compiled** before it can process other code
2. The macro crate links against the compiler's `proc_macro` API
3. Mixing compile-time and runtime code in the same crate would create circular dependencies

### A3
- **`syn`**: Parses `TokenStream` → Rust AST. Direction: **tokens → structured data**. Takes raw token streams and produces typed Rust structures like `ItemFn`, `DeriveInput`, etc.
- **`quote`**: Generates `TokenStream` from Rust-like templates. Direction: **structured data → tokens**. Uses `quote! { ... }` with `#variable` interpolation to produce token streams.

Together: `TokenStream → syn → (your logic) → quote → TokenStream`.

---

## Section B

### A4
```rust
// my_macros/src/lib.rs
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, ItemFn};

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
```
Usage:
```rust
use my_macros::log_calls;

#[log_calls]
fn greet(name: &str) -> String {
    format!("Hello, {name}!")
}

fn main() {
    let msg = greet("Alice");
    // Output:
    //   → Entering greet
    //   ← Leaving greet
    println!("{msg}");
}
```
Key design: We wrap the original body in a closure `(|| block)()` so the return value is captured before the "Leaving" message prints.

### A5
```rust
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, LitStr, parse::Parse, parse::ParseStream};

struct SqlInput {
    query: LitStr,
}

impl Parse for SqlInput {
    fn parse(input: ParseStream) -> syn::Result<Self> {
        let query: LitStr = input.parse()?;
        Ok(SqlInput { query })
    }
}

#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
    let input = parse_macro_input!(input as SqlInput);
    let query_str = input.query.value();
    let upper = query_str.trim().to_uppercase();

    let valid_starts = ["SELECT", "INSERT", "UPDATE", "DELETE"];
    if !valid_starts.iter().any(|kw| upper.starts_with(kw)) {
        return syn::Error::new(
            input.query.span(),
            format!(
                "Invalid SQL: must start with SELECT, INSERT, UPDATE, or DELETE. Got: '{}'",
                query_str.chars().take(20).collect::<String>()
            ),
        )
        .to_compile_error()
        .into();
    }

    let expanded = quote! {
        {
            let query: &str = #query_str;
            query
        }
    };
    TokenStream::from(expanded)
}
```

### A6
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
            _ => panic!("Builder only supports structs with named fields"),
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
                self.#name = Some(val);
                self
            }
        }
    });

    let build_fields = fields.iter().map(|f| {
        let name = &f.ident;
        let name_str = name.as_ref().unwrap().to_string();
        quote! {
            #name: self.#name.ok_or(format!("missing field: {}", #name_str))?
        }
    });

    let none_fields = fields.iter().map(|f| {
        let name = &f.ident;
        quote! { #name: None }
    });

    let expanded = quote! {
        pub struct #builder_name {
            #(#builder_fields,)*
        }

        impl #builder_name {
            #(#builder_methods)*

            pub fn build(self) -> Result<#name, String> {
                Ok(#name {
                    #(#build_fields,)*
                })
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
Usage:
```rust
#[derive(Builder, Debug)]
struct Config {
    host: String,
    port: u16,
    verbose: bool,
}

fn main() {
    let cfg = Config::builder()
        .host("localhost".into())
        .port(8080)
        .verbose(true)
        .build()
        .unwrap();
    println!("{cfg:?}");

    let err = Config::builder().host("x".into()).build();
    assert!(err.is_err()); // missing port and verbose
}
```

---

## Section C

### A7
| # | Answer | Explanation |
|---|--------|-------------|
| 1 | **True** | Proc macros execute during compilation to transform code |
| 2 | **False** | `syn` *parses* tokens into AST. `quote` *generates* code |
| 3 | **True** | First arg = attribute arguments, second = the annotated item |
| 4 | **True** | Proc macros are regular Rust code at compile time — they can do I/O (though this is generally discouraged) |
| 5 | **True** | `quote!` uses `#var` for interpolation and `#(#iter)*` for repetition |
| 6 | **True** | Derive macros can add `impl` blocks with new methods |

### A8
If a proc macro returns syntactically invalid `TokenStream`, the compiler reports an error, but the error will point to the **macro call site** with a confusing message like "unexpected token". The user has no idea what went wrong inside the macro.

`syn::Error::new_spanned()` improves this by:
1. **Pointing to the exact span** in the user's code that caused the problem
2. **Providing a custom message** explaining what's wrong
3. Using `.to_compile_error()` to produce a valid `TokenStream` that the compiler renders as a helpful error

Example: Instead of a generic "unexpected token", the user sees: `error: Builder only supports structs with named fields` pointing at their enum definition.

---

## 🏆 Lesson 97 Complete!

**Next up:** [Lesson 98 — SIMD & Intrinsics](../lesson_98_simd/lesson_98_simd.md) 🦀
