# /pr-title

Come up with a PR title for all the changes in the current branch.

## Workflow

**Step 1: Read the diff.**
Run `dcc diff` and read the full output. Don't skim. The goal isn't to summarize line-by-line motion but to understand what the change *does* — what's different about the codebase after this PR that wasn't true before. Note anything that surprises you, anything that wasn't obvious from filenames alone, and any change in shape (dependencies, signatures, responsibilities, behavior) that's distinct from the literal code motion.

**Step 2: Compile a list of candidate lenses for framing this PR.**
A lens is a way of answering "what's the most important thing a reader needs to know about this change?" Examples — *not a closed list, just to seed your thinking*: intent, dependency shift, signature change, before→after, symptom, root cause, scope, magnitude, user-visible effect, motivation. Derive lenses that actually fit *this* change; ignore ones that don't; invent new ones if the change calls for it. Treat this list as inspirational scaffolding, not a menu to pick from exhaustively.

**Step 3: Draft ~4 candidate titles freely.**
Keep the lens list in peripheral vision but don't bind titles 1:1 to lenses. A title can draw on one lens, blend two, or reach for a framing not on your list. Write each title to stand on its own — concrete, scannable, the kind of thing a reader skimming `git log` can absorb in a glance. Focus on the substance of the framing; surface polish comes later.

**Step 4: Diversity audit against the lens list.**
Look back at your drafts. Are three of them reworded versions of the same framing? If so, replace at least one with a draft that leans on a lens you haven't used. This is the only step where the lens list acts as a constraint — and it constrains the *titles*, not the lenses.

**Step 5: For each surviving candidate, name what it conveys and what it fails to convey.**
One short line each. This is a critique pass over concrete titles, not abstract lenses. Surface tradeoffs: "names the motion but hides the dependency removal," "precise but uses a verb the repo doesn't use," "captures intent but loses the before-state." The goal isn't scoring — it's making the tradeoffs visible before picking.

**Step 6: Pick by fit.**
Ask: *what does a reader of `git log` most need to know about this PR in five seconds?* The winner is the title whose framing answers that question for *this specific change*. State the pick and a one-sentence rationale — the rationale should reference the change itself, not the title's wording ("this PR's headline fact is the lost dependency, so the decoupling framing wins" — not "this one sounds cleanest").

**Step 7: Adjust the winning title to match repo conventions.**
First read the last 40 PR titles to ground yourself in this repo's style:

```bash
git log --oneline -100 | grep -E '\(#[0-9]+\)$' | head -40
```

Note the prefix conventions (e.g., `serving-box:`), casing, tense, length norms, and characteristic idioms (`s/X/Y/`, `++coverage`, `a sweeping change:`). Adjust the winner to conform — lowercase the verb, add the module prefix, trim length, swap to repo-idiomatic phrasing. The framing decision from step 6 stands; only the surface changes.

If conforming requires changing the *framing* rather than the *surface*, return to step 6 — that's a signal the pick was wrong, not that style should override substance.

## Output

Present steps 3–7 to the user. **Number every candidate title with an ordinal (1, 2, 3, …) and keep that numbering stable across all subsequent steps** — the critiques in step 5, the pick in step 6, and the style-adjusted final in step 7 should all refer back to candidates by their ordinal (e.g., "winner: #2", "adjusted from #2"). This makes it easy for the user to follow up with "go with #3" or "blend #1 and #4" without having to re-quote the title text.

Format roughly like:

```
Candidates:
  1. <title>
  2. <title>
  3. <title>
  4. <title>

Critiques:
  1. <what it conveys / what it misses>
  2. ...

Pick: #N — <one-sentence rationale grounded in the change>

Final (style-adjusted from #N): <title>
```
