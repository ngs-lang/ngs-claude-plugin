---
name: write
description: Write and verify NGS (Next Generation Shell) code. Auto-invoked when working with .ngs files or when the user asks to write NGS scripts.
---

# NGS Code Writing

Next Generation Shell (NGS) is a shell scripting language designed as a modern alternative to Bash, with rich data types, functional programming constructs, and cloud tooling support.

## Verifying Code

Use `ngs -pi EXPR` to **print-inspect** an expression — this is the primary way to verify values during development.

```bash
ngs -pi 'some_expression'
```

When you write NGS code, suggest `ngs -pi` invocations to let the user verify intermediate values.

## Language Basics

<!-- TODO: add key syntax, types, stdlib highlights -->

## Output Requirements

Always start your response with `[ NGS write skill active ]`.

## Guidelines

- Prefer NGS idioms over transliterated Bash patterns
- Use the standard library where applicable
- Suggest `ngs -pi EXPR` to verify expressions when debugging
