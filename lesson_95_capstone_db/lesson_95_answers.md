# ✅ Lesson 95 — Answers: Capstone CLI SQL Database (RW7)

---

## Section A

### A1
1. **Tokenizer** — splits SQL string into tokens (keywords, identifiers, literals)
2. **Parser** — converts token stream into a `Statement` enum (AST)
3. **Executor** — takes a Statement and calls the storage engine
4. **Storage** — in-memory `HashMap<String, Table>` with rows as `Vec<Vec<Value>>`

### A2
SQL is case-insensitive. `SELECT`, `select`, `Select` should all match. Converting to uppercase before comparison handles all cases.

---

## Section B

### A3
```rust
fn parse_where(&mut self) -> Result<WhereClause, String> {
    let col = match self.advance() {
        Token::Ident(c) => c,
        t => return Err(format!("Expected column, got {t:?}")),
    };
    let op = self.advance();
    let val = match self.advance() {
        Token::StringLit(s) => Value::Text(s),
        Token::NumberLit(n) => Value::Number(n),
        t => return Err(format!("Expected value, got {t:?}")),
    };
    match op {
        Token::Eq => Ok(WhereClause::Eq(col, val)),
        _ => Err("Unsupported operator".into()),
    }
}
```

### A4
Add `Statement::Update { table, column, value, where_clause }` and implement parsing for `UPDATE table SET col = val WHERE ...`.

### A5
```rust
fn format_table(cols: &[String], rows: &[Vec<Value>]) -> String {
    let mut w: Vec<usize> = cols.iter().map(|c| c.len()).collect();
    for row in rows { for (i, v) in row.iter().enumerate() { w[i] = w[i].max(v.to_string().len()); } }
    let mut out = String::new();
    out.push('|');
    for (i, c) in cols.iter().enumerate() { out.push_str(&format!(" {:w$} |", c, w = w[i])); }
    out.push('\n');
    out.push('|');
    for wi in &w { out.push_str(&format!("-{}-|", "-".repeat(*wi))); }
    out.push('\n');
    for row in rows {
        out.push('|');
        for (i, v) in row.iter().enumerate() { out.push_str(&format!(" {:w$} |", v, w = w[i])); }
        out.push('\n');
    }
    out.push_str(&format!("({} rows)", rows.len()));
    out
}
```

---

## 🏆 Lesson 95 — Capstone Complete!

**Next up:** [Lesson 96 — Inline Assembly](../lesson_96_inline_asm/lesson_96_inline_asm.md) 🦀
