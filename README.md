# Rust Programming Language - Learning Path

Welcome to the Rust Programming Language learning repository! This project is designed as a comprehensive course to help you master Rust, from absolute basics to advanced topics.

## 📚 Structure

The repository is divided into 65 progressive lessons. Each lesson has its own directory and typically contains three files:

- **`lesson_xx_<topic>.md`**: The main lesson material explaining the concepts.
- **`lesson_xx_questions.md`**: Exercises and questions to test your understanding.
- **`lesson_xx_answers.md`**: Solutions and explanations for the questions.

## 🗺️ Topics Covered

### 1. Basics

- Variables and Mutability
- Scalar and Compound Types
- Functions
- Control Flow (if/else, loops)
- Comments

### 2. Core Rust Concepts

- Ownership, Move, and Copy semantics
- References and Borrowing
- Slices
- Structs and Methods (`impl`)
- Enums and Pattern Matching (`if let`)

### 3. Standard Library Collections

- Strings vs `str`
- Vectors (`Vec`)
- HashMaps & HashSets
- BTree structures

### 4. Error Handling

- `panic!` and `unwrap`
- `Result` enum and the `?` operator
- Custom Errors and the `anyhow` crate

### 5. Project Management & Ecosystem

- Modules, Paths, and `use`
- Crates, Cargo, and Workspaces
- Publishing Packages
- Testing
- Clippy and Rustfmt

### 6. Advanced Types & Concepts

- Generics and Generic Structs
- Traits and Trait Bounds
- `impl Trait` vs `dyn Trait`
- Lifetimes (Basic and Advanced)
- Operator Overloading & Associated Types
- Closures, Function Traits, and Higher-Order Functions

### 7. Smart Pointers & Memory Management

- `Box`
- `Rc` and `Arc`
- `RefCell`

### 8. Concurrency

- Threads
- Channels (Message Passing)
- `Mutex` and `RwLock` (Shared State)
- `Send` and `Sync` traits

### 9. Asynchronous Programming

- `async`/`await`
- Tokio Runtime
- Async I/O

### 10. Advanced Rust 🦀

- Unsafe Rust
- Declarative Macros
- Procedural Macros
- FFI (Foreign Function Interface)

## 🚀 How to Use This Repository

1. **Clone the repository:**
   ```bash
   git clone <your-repo-url>
   cd "Rust Programming Language"
   ```
2. **Start at Lesson 1:** Navigate to `lesson_01_variables_and_mutability`.
3. **Read the Lesson:** Open the main lesson markdown file and read through the concepts.
4. **Test Yourself:** Open the `_questions.md` file and try to answer or write the code.
5. **Check Your Work:** Compare your solutions with the `_answers.md` file.
6. **Progress:** Move on to the next directory!

Happy learning!
