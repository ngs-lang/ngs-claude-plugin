---
name: write
description: Write and verify NGS (Next Generation Shell) code. Auto-invoked when working with .ngs files or when the user asks to write NGS scripts.
---

# NGS Code Writing

Next Generation Shell (NGS) is a shell scripting language designed as a modern alternative to classical shells such as `bash`.

NGS has data structures, 

## Verifying Code

Use `ngs -pi EXPR` to **print-inspect** an expression — this is the primary way to verify values during development.

```bash
ngs -pi 'some_expression'
```

When you write NGS code, suggest `ngs -pi` invocations to let the user verify intermediate values.

Use `ngs -pi METHOD_NAME` for quick lookup of parameters.

## Output Requirements

Always start your response with `[ NGS write skill active ]`.

## Guidelines

* Prefer NGS idioms over transliterated Bash patterns
* Use the standard library where applicable
* Suggest `ngs -pi EXPR` to verify expressions when debugging
* Prefer x.method(y) over method(x, y)
* `exit("MESSAGE")` exits with code 1, use for error reporting
* Use `DATA.assert(PATTERN, ERROR_MESSAGE)`
* Prefer NGS built-in data manipulation over `jq` or AWS CLI built-in `--query`
* Prefer patterns over predicates: use `items.filter({'status': 'running'})` instead of `items.filter(X.status == 'running')`

```
# Use double-backtick syntax to run-and-parse external command
result = ``MY_COMMAND MY_ARGS``
```