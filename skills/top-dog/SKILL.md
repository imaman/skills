---
name: top-dog
description: 'Orchestrate the review skills (review-dog, regression-dog) into one deduplicated, adjudicated report. TRIGGER when: the user wants a combined/full/aggregated review of a branch or commit (e.g., "run the full review", "review this branch with everything", "give me the aggregated review").'
---

Run review-dog and regression-dog over the same change, then adjudicate their findings into one human-friendly report. You are an adjudicator, not a relay: nothing reaches the output unless you have judged it real and material.

Important:
- Do NOT run tests, typechecks, linters, or build commands. CI already handles those. Spend your context budget on reasoning about the findings and the code they cite.

Run the dogs:
- Launch one subagent per dog — review-dog and regression-dog — IN PARALLEL, each passed the same scope argument you received. Instruct each to invoke its skill on that range and return its findings verbatim.
- Use subagents, not inline reviews. This keeps each dog's heavy lifting — reading the diff, opening files, its chain of reasoning — OUT of your context: only the findings come back. It also stops the two lenses from biasing each other.

Adjudicate. The dogs' own severity labels (critical/warning/nit) are a soft communication signal, NOT a grounded verdict — do not filter on them. Reason about each finding yourself:

1. Collect every raw finding from both dogs into an internal ledger and record each one's disposition (kept / merged / dropped + why). You will need to recall dropped findings if asked, so never discard the ledger.
2. Dedup and merge: when findings point at the same root issue (usually the same file:line), fold them into one. Overlap across two different lenses is a CONFIDENCE signal — promote the merged finding and note it was flagged by both, don't bury it.
3. Adjudicate each merged finding. The dogs' findings are self-justifying — they quote the code and explain the mechanism — so judge from the description and lean on their reading instead of redoing it:
   - Material? Drop true nits — cosmetic preferences, restated-obvious observations, things no reviewer would act on. This call needs only the description.
   - Holds? Trust it by default. When you're about to drop one as wrong, or two findings conflict, verify before acting — read just the cited lines (not the whole diff), or re-engage the originating dog with a follow-up (its review context is still loaded, so it answers without re-reading). Use follow-ups to CLARIFY, not to validate: a dog defends its own finding, so make the holds/severity call yourself. (The dogs are already change-scoped, so don't re-verify that a finding is about new code.)
   - GUARD: never drop a finding citing a CLAUDE.md/AGENTS.md "MUST"/"NEVER"/"ALWAYS" rule — not a nit however minor it looks.
4. Assign your OWN severity (critical / warning / nit) to each survivor, based on impact — independently of whatever the dog called it.

Output format:
- List surviving findings first, numbered, strongest first. Each finding has three parts:
  1. Title line: a tight one-liner (≤ ~12 words) naming the specific problem. Strong verbs, concrete nouns — a reader should be able to prioritize from the title alone.
  2. Subtitle (italicized, immediately under the title): `severity · aspect · flagged by <review-dog|regression-dog|both> — path/to/file.ts:line`.
  3. Body: explain what's wrong and why it matters; quote the offending code or point at specific identifiers.
- Then a "Cleared" section: the aspects/areas the dogs reviewed and found clean, merged and deduped across both, each prefixed with ✅.
- End with a single dropped-count line, e.g.: `Dropped 7 findings: 3 duplicates, 4 low-value. Ask to see them.` Do not list the dropped findings inline — but keep them in your ledger so you can spell them out on request.

Explaining a finding:
- Hand the request to the originating dog — it still has the diff and its reasoning loaded.
- Ask it to prefer snippets (annotated where useful) over prose, when the finding pins to code.
- Ask it to keep the writeup tight and direct.

Scope (pass as arguments):
- No arguments: review the most recent commit
- `main`: review all commits since the last merge from main
- `HEAD~3`: review the last 3 commits
- Any git revision range (e.g., `abc123..HEAD`, `main..HEAD`)
