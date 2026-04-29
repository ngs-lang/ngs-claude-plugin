---
name: write
description: Write and verify NGS (Next Generation Shell) code. Auto-invoked when working with .ngs files or when the user asks to write NGS scripts.
---

# NGS Code Writing

[Next Generation Shell (NGS)](https://github.com/ngs-lang/ngs) is a shell scripting language designed as a modern alternative to classical shells such as `bash`.

NGS has data structures, proper error handling, multi-methods and multiple dispatch.

Always start your response with `[ NGS write skill active ]`.

## Verifying Code

Use `ngs -pi EXPR` to **print-inspect** an expression — this is the primary way to verify values during development.

```bash
ngs -pi 'some_expression'
ngs -pl 'some_expression'  # print lines
ngs -pj 'some_expression'  # print as JSON
ngs -pt 'some_expression'  # print as table
```

When you write NGS code, suggest `ngs -pi` invocations to let the user verify intermediate values.

Do not guess APIs, use `ngs -pi METHOD_NAME` for quick lookup of parameters. For example, `ngs -pi echo` shows all `echo` implementations and their parameters.

## General

* Suggest `ngs -pi EXPR` to verify expressions when debugging
* `exit("MESSAGE")` exits with code 1, use for error reporting
* `die("MESSAGE")` exits with message and backtrace
* If looking into stdlib as AI agent, note that stdlib is both speed optimized and tries not to use more appropriate facilities if they are defined later.
* `require("path/to/file.ngs")` loads a module. **Current bug**: path is relative to current working directory, not the file that calls `require()`.
* `Program("name")` represents an external program; use `assert(Program("zip"))` to verify a program exists.

## Syntax

* `{ ... }` in NGS creates a method reference (equivalent to `F(...) ...`), NOT an inline evaluated block — unless it's in a specific syntactic position like after `if` or `while`.
* Only declare a symbol in global if you are defining it in the current scope.
* There is no trailing block syntax in NGS.

## Pitfalls

* NGS default argument (ex: `path=[]`) is NOT fresh per call — it's the same shared mutable list as in Python's default argument gotcha.
* Pipes are not passing objects. Currently pipes are only used to run external commands and function like pipes in bash.

## Output

* Use `log()` and `warn()` for timestamped output to stderr.
* Use `debug(FACILITY, MESSAGE)` or `debug(MESSAGE)` for debug output to stderr, controlled by `DEBUG=FACILITY1,FACILITY2,...` environment variable.

## Idiomatic NGS

* Prefer `x.method(y)` over `method(x, y)`
* Use `.=` for in-place method application: `var .= f(arg)` is equivalent to `var = var.f(arg)` / `var = f(var, arg)`
* Use `DATA.assert(ERROR_MESSAGE)` — checks DATA is truthy
* Use `DATA.assert(PATTERN, ERROR_MESSAGE)` — checks DATA matches pattern
* Rely heavily on multiple dispatch, if possible use same name for methods (verbs) and keep number of verbs to minimum.
* Prefer new types with existing verbs over new verbs over untyped (Hash for example) data
* Constants are uppercase. (but in this skill file, uppercase usually means placeholder)

## Code Structure

* Variables and functions/methods defined in an enclosing scope are accessible to inner functions as closures — works in `ns { }`, `F`, and other blocks. Use for shared constants instead of duplicating literals.
* Use `NAME = ns { ... }` to define namespaces. Prefix private/internal functions with `_` (e.g. `_chunks`) to keep them out of public namespace API.
* Do not comment if the code is obvious.
* Small sections of code - `# BLAH` comment before. 
* Larger sections of code - use `section "BLAH" { ... }` for organizing the code. Also, instead of splitting into a function that is called only once.
* If a function/method f1 is used only from within f2, it should be defined *inside* f2.


## Data Manipulation

* `Hash([['a', 0], ['b', 1]])` / `Hash(arr_of_pairs)` constructs a Hash from an array of key-value pairs
* Access environment variables with `ENV.varname` or `ENV.get('varname', 'default')`
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
* Prefer `VALUE.when(PATTERN, CB_OR_NEW_VALUE)` over `if COND then CB(VALUE) else VALUE` / `if COND then NEW_VALUE else VALUE`. (when `COND` does pattern matching between `VALUE` and `PATTERN` in the broad sense, including `VALUE == PATTERN` for scalars)
  * Example: `["ssh", "IP", "w"].map(X.when("IP", "10.0.0.1"))` gives `["ssh", "10.0.0.1", "w"]`.
  * Example: `v.when(Not(null), { ... })`
  * Works well in method chains.
* Prefer NGS built-in data manipulation over `jq` or AWS CLI built-in `--query`
* Prefer `fetch(PATH)` over `read(PATH).decode_json()`
* Prefer `store(PATH, DATA)` over `write(PATH, DATA.encode_json())`
  * `store(path, data, encode_hints)` accepts a third `encode_hints` Hash — same as `encode_json()`.
* `encode_json()` pretty-print API uses a hints Hash (encode_json(data, {"pretty": true})), not keyword arguments
* `pos()` returns `null` if substring not found

### Patterns

* NGS has `Pfx`, `Sfx`, `MaybePfx`, `MaybeSfx` patterns (among others)
* `Transformed(fn, pattern)` matches by applying `fn` first, then matching against `pattern`. Example: `arr.assert(Transformed(len, 3), "must have 3 elements")`

### AWS CLI

* NGS's double backtick syntax for AWS CLI commands does more than parse JSON — it normalizes AWS-specific structures. For example, AWS Tags (`[{Key: "Name", Value: "foo"}, ...]`) are converted to a regular Hash (`{"Name": "foo", ...}`).
* For many commands, double backtick syntax returns `Arr` at the top level. For ``aws ec2 describe-instances`` no drill down is required with `.Reservations.Instances`.
  ```
  instances = ``aws ec2 describe-instances``.sort('LaunchTime').Hash('InstanceId')
  ```
  (test it to know if your specific command is like this)

## Flow Control

* Use `block NAME { }` for non-local exit with `NAME.return()` (returns `null`) and `NAME.return(VALUE)`.
  * Idiomatic block name is `b`, but use a descriptive name (e.g. `cmp`) if `b` would shadow a variable.
  * Use it when deep, ex: `F f() block b { something.each({ ... b.return() ... }) }`.
  * If not deep, prefer `COND returns VAL` (only works from the function level, including anonymous functions `{...}`). Note that `for`, `while`, `if` do not create a new level.
* `paginate(F(token) { ... })` is a built-in for pagination loops — initializes token to null, loops until callback returns falsy. Does NOT collect results — wrap in `collector { }` to use `collect`
* C-style `for(i=0; i<n; i+=step)` works in NGS — prefer over `i=0; while ... i+=step`. Use `for(i;n)` when step is 1. Prefer `each` to `for`.
* Use `not(COND) returns VALUE` for early-exit guard clauses instead of `if not(COND) { return VALUE }` or `COND or return VALUE`
* Use `retry()`. (REVIEW THIS POINT)
  * It does not handle exceptions. To retry only on a specific exception type, catch it in the body and return null (falsy), other exceptions propagate naturally.
  * Use named arguments. Ex: `retry(times=..., sleep=..., body=...)`, etc.
* There is no ternary operator (`? :`). Use `if COND { A } else { B }` instead.
* There is `match` syntax: `match VAL { PAT1 EXPR1 PAT2 EXPR2 ... }`. First time VAL matches PAT, EXPR is evaluated and becomes the result of `match`.
* There is `cond` syntax: `cond { COND1 EXPR1 COND2 EXPR2 ...}`. First COND that evaluates to true, EXPR is evaluated and becomes the result of `cond`.

## Running External Commands

```
# Use double-backtick syntax to run-and-parse external command
result = ``MY_COMMAND MY_ARGS``
```

# Docs

## Manuals
- [Language reference (ngslang)](https://ngs-lang.org/doc/latest/man/ngslang.1.html) — syntax, variables, types, methods, exceptions
- [Tutorial (ngstut)](https://ngs-lang.org/doc/latest/man/ngstut.1.html) — examples-based getting started guide
- [Style guide (ngsstyle)](https://ngs-lang.org/doc/latest/man/ngsstyle.1.html) — code formatting and conventions
- [Motivation (ngswhy)](https://ngs-lang.org/doc/latest/man/ngswhy.1.html) — design philosophy
- [CLI reference (ngs)](https://ngs-lang.org/doc/latest/man/ngs.1.html) — command-line options and usage
- [AWS CLI (na)](https://ngs-lang.org/doc/latest/man/na.1.html) — NGS AWS command-line tool
- [Internals (ngsint)](https://ngs-lang.org/doc/latest/man/ngsint.1.html) — architecture, VM, compilation

## Generated Reference
- [Global namespace](https://ngs-lang.org/doc/latest/generated/index.html) — lists all methods and other namespaces

## Wiki
- [Home](https://github.com/ngs-lang/ngs/wiki)
- [Intro handson](https://github.com/ngs-lang/ngs/wiki/Intro-handson) — getting started with practical exercises
- [Running External Commands](https://github.com/ngs-lang/ngs/wiki/Running-External-Commands) — all syntaxes and modifiers
- [Pitfalls](https://github.com/ngs-lang/ngs/wiki/Pitfalls) — common mistakes and how to avoid them
- [Use Cases](https://github.com/ngs-lang/ngs/wiki/Use-Cases)
- [Examples](https://github.com/ngs-lang/ngs/wiki/Examples)
- [Code Snippets Comparisons](https://github.com/ngs-lang/ngs/wiki/Code-Snippets-Comparisons)

