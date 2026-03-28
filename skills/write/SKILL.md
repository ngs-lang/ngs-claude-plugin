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

Do not guess APIs, use `ngs -pi METHOD_NAME` for quick lookup of parameters. For example, `ngs -pi echo` shows all `echo` implementations and their parameters.

## General

* Suggest `ngs -pi EXPR` to verify expressions when debugging
* `exit("MESSAGE")` exits with code 1, use for error reporting
* If looking into stdlib as AI agent, note that stdlib is both speed optimized and tries not to use more appropriate facilities if they are defined later.

## Syntax

* `{ ... }` in NGS creates a method reference (equivalent to `F(...) ...`), NOT an inline evaluated block — unless it's in a specific syntactic position like after `if` or `while`.

## Pitfalls

* NGS default argument (ex: `path=[]`) is NOT fresh per call — it's the same shared mutable list as in Python's default argument gotcha.

## Output

* Use `log()` and `warn()` for timestamped output to stderr.
* Use `debug(FACILITY, MESSAGE)` or `debug(MESSAGE)` for debug output to stderr, controlled by `DEBUG=FACILITY1,FACILITY2,...` environment variable.

## Idiomatic NGS

* Prefer x.method(y) over method(x, y)
* Use `DATA.assert(PATTERN, ERROR_MESSAGE)`
* Rely heavily on multiple dispatch, if possible use same name for methods (verbs) and keep number of verbs to minimum.
* Prefer new types with existing verbs over new verbs over untyped (Hash for example) data
* Use `section "BLAH" { ... }` for organizing the code. Also, instead of splitting into a function that is called only once.

## Code Structure

* Use `NAME = ns { ... }` to define namespaces. Prefix private/internal functions with `_` (e.g. `_chunks`) to keep them out of public namespace API.

## Data Manipulation

* Prefer patterns over predicates: use `items.filter({'status': 'running'})` instead of `items.filter(X.status == 'running')`
* Deep pattern matching: patterns are recursive, `AnyOf(arr)` matches any element, `.filter(pattern)` works directly (no need for `F(r) r =~ pattern`). Hash patterns can use newline-separated keys (no commas), merge multiple `.filter()` calls into one pattern:
  ```
  records.filter({
      'Type': 'CNAME'
      'ResourceRecords': [{
          'Value': AnyOf(targets)
      }]
  })
  ```
* `Hash.get(key)` defaults to `null` (no second argument needed)
* `Box` exists in NGS (wraps value in optional container) but prefer `.get()` for simple key lookups
* Use `.the_one(PATTERN)` over `.filter(PATTERN)[0]` to express expectation of exactly one element.
* Prefer `VALUE.when(COND, CB_OR_NEW_VALUE)` over `if COND then CB(VALUE) else VALUE` / `if COND then NEW_VALUE else VALUE`. Example: `["ssh", "IP", "w"].map(X.when("IP", "10.0.0.1"))` gives `["ssh", "10.0.0.1", "w"]`. Any time you'd write `if COND then f(x) else x`, consider `x.when(COND, f)` instead. Works well in method chains.
* Prefer NGS built-in data manipulation over `jq` or AWS CLI built-in `--query`

### Patterns

* NGS has `Pfx`, `Sfx`, `MaybePfx`, `MaybeSfx` patterns (among others)

### AWS CLI

* NGS's double backtick syntax for AWS CLI commands does more than parse JSON — it normalizes AWS-specific structures. For example, AWS Tags (`[{Key: "Name", Value: "foo"}, ...]`) are converted to a regular Hash (`{"Name": "foo", ...}`).
* For many commands, double backtick syntax returns `Arr` at the top level. For ``aws ec2 describe-instances`` no drill down is required with `.Reservations.Instances`.
  ```
  instances = ``aws ec2 describe-instances``.sort('LaunchTime').Hash('InstanceId')
  ```
  (test it to know if your specific command is like this)

## Flow Control

* Use `block NAME { }` for non-local exit with `NAME.return()` (returns `null`) and `NAME.return(VALUE)`
* `paginate(F(token) { ... })` is a built-in for pagination loops — initializes token to null, loops until callback returns falsy. Does NOT collect results — wrap in `collector { }` to use `collect`
* C-style `for(i=0; i<n; i+=step)` works in NGS — prefer over `i=0; while ... i+=step`. Use `for(i;n)` when step is 1. Prefer `each` to `for`.

## Running External Commands

```
# Use double-backtick syntax to run-and-parse external command
result = ``MY_COMMAND MY_ARGS``
```
