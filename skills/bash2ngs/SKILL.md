---
name: bash2ngs
description: Convert Bash scripts or snippets to idiomatic NGS (Next Generation Shell). Auto-invoked when the user asks to convert or rewrite Bash code in NGS.
argument-hint: "[bash script or snippet]"
---

# Bash to NGS Conversion

*THIS IS WORK IN PROGRESS, DO NOT USE*

Convert Bash code to idiomatic NGS — not a mechanical transliteration, but a rewrite using NGS constructs and stdlib.

## Notes

* `set -eu` in bash is the default behaviour in NGS

## Verification

After conversion, suggest `ngs -pi EXPR` invocations to verify key expressions.
