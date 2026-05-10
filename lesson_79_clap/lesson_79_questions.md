# 🧪 Lesson 79 — Questions: CLI Apps with clap (RW1)

> **Lesson:** [lesson_79_clap.md](./lesson_79_clap.md)  
> **Answers:** [lesson_79_answers.md](./lesson_79_answers.md)

---

## Section A — Conceptual

### Q1
What is the difference between a positional argument and a flag (`--name`)?

---

## Section B — Write It Yourself

### Q2 — Todo CLI (Roadmap Practice Task)
Build a CLI with subcommands: `add <text> --priority <1-5>`, `list --filter <all|done|pending>`, `done <id>`. Use clap derive API.

### Q3 — Custom validation
Create an argument `--port` that only accepts values 1024–65535 and an `--email` that must contain `@`.

---

## Section C — True or False?

### Q4
1. `#[derive(Parser)]` auto-generates argument parsing from struct fields.
2. `#[arg(short, long)]` creates both `-n` and `--name` flags.
3. `Option<String>` makes an argument optional.
4. clap generates `--help` output automatically from doc comments.
5. Subcommands require a separate `#[derive(Subcommand)]` enum.
6. `ValueEnum` lets you use enums as argument choices.

---

*clap: command-line parsing done right! 🦀*
