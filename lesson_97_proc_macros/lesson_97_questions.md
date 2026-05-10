# 🧪 Lesson 97 — Questions: Attribute & Function-like Macros (MA3)

> **Lesson:** [lesson_97_proc_macros.md](./lesson_97_proc_macros.md)  
> **Answers:** [lesson_97_answers.md](./lesson_97_answers.md)

---

## Section A — Conceptual

### Q1
Name the three kinds of procedural macros and give the function signature attribute for each.

### Q2
Why must proc macros live in a **separate crate** with `proc-macro = true`? Why can't they be in the same crate as your application code?

### Q3
Explain the roles of the `syn` and `quote` crates. What does each transform, and in which direction?

---

## Section B — Write It Yourself

### Q4 — `#[log_calls]` attribute macro (Roadmap Practice Task)
Write an attribute macro `#[log_calls]` that wraps any function to print its name on entry and exit. It should:
1. Print `"→ Entering <fn_name>"` before the body executes
2. Print `"← Leaving <fn_name>"` after the body executes
3. Return the original function's return value
4. Preserve the function's visibility, attributes, and signature

### Q5 — `sql!()` function-like macro
Write a function-like proc macro `sql!("SELECT ...")` that:
1. Accepts a string literal
2. Validates at compile time that it starts with a valid SQL keyword (`SELECT`, `INSERT`, `UPDATE`, `DELETE`)
3. Returns a compile error with a helpful message if validation fails
4. Returns the validated query string if it passes

### Q6 — `#[derive(Builder)]`
Write a derive macro that generates a builder pattern for a struct. For a struct with fields `name: String` and `age: u32`, it should generate:
- A `FooBuilder` struct with `Option<T>` fields
- A `Foo::builder()` method returning `FooBuilder`
- Setter methods on `FooBuilder` for each field
- A `build()` method that returns `Result<Foo, String>`

---

## Section C — True or False?

### Q7
1. Proc macros run at compile time, not runtime.
2. The `syn` crate generates Rust code from a template.
3. `#[proc_macro_attribute]` functions receive two `TokenStream` arguments (attrs + item).
4. Proc macros can read files from disk or make network requests at compile time.
5. `quote!` uses `#variable` syntax to interpolate Rust variables into generated code.
6. A derive macro can add new methods to the struct it's applied to.

### Q8
What happens if a proc macro returns `TokenStream` that contains a syntax error? How does `syn::Error::new_spanned()` improve the developer experience compared to `panic!`?

---

*Code that writes code! 🦀*
