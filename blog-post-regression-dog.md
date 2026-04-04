---
title: A 20-line skill that replaced my code review safety net
published: false
tags: claudecode, ai, codereview, devtools
---

I stopped waiting for Greptile.

For a while my workflow was: push a branch, wait for the GitHub bot to chew on it, context-switch to something else, come back minutes later, scroll through findings. It worked, but it had this annoying latency tax — both wall-clock and mental.

Then I wrote a Claude Code skill called `regression-dog`. It's about 20 lines of markdown. No code, no server, no API keys. And it quietly became the thing I reach for on almost every PR.

## What it does

It reads the diff of your branch (or last commit, or last N commits — you choose the scope) and lists every behavioral change it can find. That's it. No style nits, no "consider renaming this variable," no improvement suggestions. Just: "this used to do X, now it does Y."

The prompt tells the LLM three things:

1. Don't run tests, linters, or builds. CI handles that. Spend your entire context budget on reasoning about what changed.
2. Don't judge. Don't decide whether old or new behavior is "better." Just surface the delta.
3. List what's cleared, too — things you reviewed and found safe.

That third point sounds cosmetic but it's actually load-bearing. When the LLM inspects a code change, it now has two outlets: either the change is safe (goes to "Cleared") or it's a behavioral difference (goes to "Regressions"). This symmetry forces the model to make an explicit decision on every change instead of quietly skipping things it's unsure about. It improved recall noticeably when I added it.

## How I actually use it

I work on something. When I think it's ready, I open a fresh Claude Code session — fresh, so the review isn't biased by the conversation that produced the code — and type something like:

```
flag all regressions in this branch
```

It comes back in seconds. Right there in my terminal. No push, no wait, no tab-switching to GitHub.

Then I fix things in that same session. It already has rich context about the branch, so fixes are fast. I apply judgment — some "regressions" are intentional behavior changes, and I skip those. Then I open yet another clean session and run it again. Repeat until it comes back clean, or until every flagged item is something I'm deliberately changing.

The whole loop is maybe two minutes. Compare that to push → wait for CI bot → read findings → go back to editor → push again.

## Why it works on non-refactoring PRs too

I originally built this for pure refactors where zero behavioral changes are expected. A refactor regression-dog is straightforward: any behavioral delta is a bug.

But then I started running it on feature branches, bug fixes, everything. Turns out "enumerate what changed" is a useful frame even when changes are expected. It catches the *unintended* changes hiding next to the intentional ones. You meant to change the retry logic, but you also accidentally changed the error message format — that kind of thing.

The "don't judge" instruction is key here. A model that tries to assess whether each change is good or bad will get confused on feature branches where most changes *are* good. A model that just lists deltas gives you a clean inventory to scan with your own brain.

## What it doesn't do

It doesn't flag code quality issues. Sometimes it will mention one in passing, but it's not looking for them and I think that's right. Mixing "you have a regression" with "you should rename this variable" would cloud the model's focus. Dedicated tools for dedicated jobs.

It also doesn't replace tests. It reasons about code statically. It can catch things tests miss (subtle behavioral shifts in untested paths), and tests catch things it misses (runtime behavior, integration effects). They're complementary.

## The full skill

Here's the entire prompt — this is all there is:

```markdown
Review code changes for behavioral differences between the before and after code.

Important:
- Do NOT run tests, typechecks, linters, or build commands.
  CI already handles those. Focus your context budget entirely on
  reasoning about the code changes and its implications on
  logic/behavior/data/etc.
- Enumerate behavioral differences: "this used to do X, now it does Y."
  Do not judge whether the old or new behavior is correct — just
  surface the delta. Do not flag pre-existing issues or suggest
  improvements.

Output format:
- List regressions first, numbered, with severity.
- After the regressions section, add a "Cleared" section listing items
  that were reviewed and found to have no issues.
  Prefix each cleared item with a ✅ emoji.

Scope (pass as arguments):
- No arguments: review the most recent commit
- `main`: review all commits since the last merge from main
- `HEAD~3`: review the last 3 commits
- Any git revision range (e.g., `abc123..HEAD`, `main..HEAD`)
```

## Install it

```bash
npx skills add https://github.com/imaman/skills --skill regression-dog
```

After installing, open a Claude Code session in your repo and ask it to review your branch for regressions. It'll pick up the skill automatically.

There are a couple more skills in [the same repo](https://github.com/imaman/skills) if you want to browse around.
