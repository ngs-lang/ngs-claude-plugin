---
name: write
description: Write and verify NGS (Next Generation Shell) code. Auto-invoked when working with .ngs files or when the user asks to write NGS scripts.
---

# NGS Code Writing

Next Generation Shell (NGS) is a shell scripting language designed as a modern alternative to classical shells such as `bash`.

NGS has data structures, proper error handling, multi-methods and multiple dispatch.

Always start your response with `[ NGS write skill active ]`.

## Verifying Code

Use `ngs -pi EXPR` to **print-inspect** an expression — this is the primary way to verify values during development.

```bash
ngs -pi 'some_expression'
```

When you write NGS code, suggest `ngs -pi` invocations to let the user verify intermediate values.

Use `ngs -pi METHOD_NAME` for quick lookup of parameters.

## General

* Suggest `ngs -pi EXPR` to verify expressions when debugging
* `exit("MESSAGE")` exits with code 1, use for error reporting
* If looking into stdlib as AI agent, note that stdlib is both speed optimized and tries not to use more appropriate facilities if they are defined later.

### Output

* Use `log()` and `warn()` for timestamped output to stderr.

### Idiomatic NGS

* Prefer x.method(y) over method(x, y)
* Use `DATA.assert(PATTERN, ERROR_MESSAGE)`
* Rely heavily on multiple dispatch, if possible use same name for methods (verbs) and keep number of verbs to minimum.
* Prefer new types with existing verbs over new verbs over untyped (Hash for example) data
* Use `section "BLAH" { ... }` for organizing the code. Also, instead of splitting into a function that is called only once.

### Data Manipulation

* Prefer NGS built-in data manipulation over `jq` or AWS CLI built-in `--query`
* Prefer patterns over predicates: use `items.filter({'status': 'running'})` instead of `items.filter(X.status == 'running')`

### Running External Commands

```
# Use double-backtick syntax to run-and-parse external command
result = ``MY_COMMAND MY_ARGS``
```

## Checking Multi-Method Methods and their Parameters

Use `ngs -pi MULTI_METHOD` to list all methods and their signatures. For example, `ngs -pi echo` shows all `echo` implementations and their parameters.
