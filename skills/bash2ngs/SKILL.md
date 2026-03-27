---
name: bash2ngs
description: Convert Bash scripts or snippets to idiomatic NGS (Next Generation Shell). Auto-invoked when the user asks to convert or rewrite Bash code in NGS.
argument-hint: "[bash script or snippet]"
---

# Bash to NGS Conversion

Convert Bash code to idiomatic NGS — not a mechanical transliteration, but a rewrite using NGS constructs and stdlib.

## Key Principles

- Use NGS data types (Hash, Arr, etc.) instead of Bash string manipulation
- Replace Bash pipelines with NGS functional equivalents where appropriate
- Use NGS stdlib instead of external commands where available
- Replace error-prone Bash patterns with NGS's structured error handling

## Common Mappings

<!-- TODO: add bash→ngs mapping table (variables, loops, conditionals, pipes, etc.) -->

## Verification

After conversion, suggest `ngs -pi EXPR` invocations to verify key expressions.
