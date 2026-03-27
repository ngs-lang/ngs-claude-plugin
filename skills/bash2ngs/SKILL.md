---
name: bash2ngs
description: Convert Bash scripts or snippets to idiomatic NGS (Next Generation Shell). Auto-invoked when the user asks to convert or rewrite Bash code in NGS.
argument-hint: "[bash script or snippet]"
---

# Bash to NGS Conversion

Convert Bash code to idiomatic NGS — not a mechanical transliteration, but a rewrite using NGS constructs and stdlib.

## Defaults

* `set -eu` in bash is the default behaviour in NGS

## JSON

* `jq` for reading/parsing JSON files -> use `fetch(FILE)` which reads and auto-parses JSON. Access fields with native dot notation (`payload.resources`) instead of `jq '.resources'`.
* `jq -n --argjson ...` for constructing JSON -> use native Hash/Arr literals and `.encode_json()`: `{'key': val, 'arr': [item]}.encode_json()`.
* Output of a command parsed with `jq` -> double backticks auto-parse JSON output. Example:
  ```ngs
  response = ``curl ...``
  echo(response.field1)
  ```

## Control flow

* `for i in $(seq 0 $((N-1)))` with manual indexing -> `.each_idx_val(F(i, item) { ... })` to iterate with index and value directly.
* Polling loop with `while`/`sleep` -> `retry(times=N, sleep=SECS, body={ ... })`. The body returns a Bool — `true` means success/stop.

## Running commands

* Repeated command-line arguments duplicated across calls -> extract into an array with `%[--flag1 --flag2 ...]` and splat with `$*{var}` in the command:
  ```ngs
  ``mycmd $*{common_arguments} --extra arg``
  ```

## Output

* `echo` for status/progress messages -> `log()`, which writes to stderr (with timestamp).
* `echo "error message"; exit 1` -> `exit("error message")` which prints to stderr and exits with code 1.

## Misc

* `$ENV_VAR` -> `ENV.VAR_NAME`. Environment variables are accessed via the `ENV` hash. Accessing a missing key throws an exception whose message mentions the environment variable name.

## Verification

After conversion, suggest `ngs -pi EXPR` invocations to verify key expressions.

## Checking Multi-Method Methods and their Parameters

Use `ngs -pi MULTI_METHOD` to list all methods and their signatures. For example, `ngs -pi echo` shows all `echo` implementations and their parameters.
