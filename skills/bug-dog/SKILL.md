---
name: bug-dog
description: 'Review new/changed code for bugs. TRIGGER when: checking for bugs, auditing new logic, or validating correctness (e.g., "any bugs here?", "check this for bugs", "is this correct?").'
---

Review new/changed code for bugs.

Important:
- Do NOT run tests, typechecks, linters, or build commands. CI already handles those. Focus your context budget entirely on reasoning about the code.
- Only review new or changed code. Do not flag pre-existing issues in unchanged code. Do not suggest improvements, style changes, or refactors.

Step 1 — Read the diff. For each meaningful change, describe what is the new/changed behavior.

Step 2 — Stress-test. For each new/changed behavior, ask: does this make sense? Does it miss edge cases? Does it handle sad paths? Look for: logic errors, off-by-one errors, null/undefined gaps, missing edge cases, race conditions, incorrect assumptions, wrong operator, swapped arguments, resource leaks, short-circuit mistakes, boundary conditions, silent data loss, and incorrect error propagation.

Output format:
- List bugs first, numbered, with severity. For each bug, state what the code does and what it should do instead.
- After the bugs section, add a "Cleared" section listing items that were reviewed and found to have no issues. Prefix each cleared item with a ✅ emoji.

Scope (pass as arguments):
- No arguments: review the most recent commit
- `main`: review all commits since the last merge from main
- `HEAD~3`: review the last 3 commits
- Any git revision range (e.g., `abc123..HEAD`, `main..HEAD`)
