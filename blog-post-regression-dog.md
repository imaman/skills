---
title: A 20-line skill that replaced my code review safety net
published: false
tags: claudecode, ai, codereview, devtools
---

I stopped waiting for Greptile and co.

Instead I run a skill I call `regression-dog`. It's open source, simple (about 20 lines of markdown), and integrates right into my terminal/coding agent.

Initially, I imagined I'd do a few quick passes locally and then still lean on the dedicated review bots for the real deep analysis. To my surprise, it's usually just as thorough as those bots, and sometimes better.

## What it does

It reads the diff of your branch (or last commit, or last N commits - you choose the scope) and lists every behavioral change it can find. Not style nits, not "consider renaming this variable" — just behavioral deltas. This used to do X, now it does Y.

The output is split into two sections. First, numbered regressions with severity ratings — things like "the retry loop now exits after 3 attempts instead of 5" or "the error response no longer includes the request ID." Then a "Cleared" section listing every change it reviewed and found safe. That second section matters more than it sounds — I'll explain why below.

A typical output looks like this:

[screenshot goes here]

## Let's unpack the magic

The whole prompt is meticulously crafted to keep the LLM focused on a well-grounded task. Every line is there for a reason:

> Do NOT run tests, typechecks, linters, or build commands.

Protects the agent's context window so the underlying LLM can invest its reasoning where it matters.

> Enumerate behavioral differences: this used to do X, now it does Y.

It poses a specific, concrete question: what are all the behavioral changes in this diff? This is a grounded task, unlike open-ended "review my code" prompts.

> Do not judge whether the old or new behavior is correct - just surface the delta.

Picking apart code is an LLM's happy place. Judging intent isn't. This keeps it where it's strongest.

> Do not flag pre-existing issues or suggest improvements

Shuts down a rabbit hole agents love going down.

> Add a "Cleared" section listing items that were reviewed and found to have no issues.

This sounds cosmetic but it's load-bearing. When the agent inspects a code change, it has two outlets: either the change is safe (goes to "Cleared") or it's a behavioral difference (goes to "Regressions"). This symmetry forces an explicit decision on every change instead of quietly skipping things it's unsure about. It improved recall noticeably when I added it.

## How I actually use it

I work on something. When I think it's ready, I open a fresh Claude Code session and type something like:

```
flag all regressions in this branch
```

A fresh session matters — it keeps the review unbiased by the conversation that produced the code.

It comes back fast - often under a minute, depending on scope. Even with bigger changes it's significantly faster than the push-to-GitHub-wait-for-bot loop. And it's right there in my terminal, no tab-switching.

Then I fix things in that same session. It's already loaded with high quality context about the branch, so fixes are fast. Some "regressions" are intentional, so I just move on. Then I open yet another clean session and run it again. Repeat until it comes back clean, or until every flagged item is something I'm deliberately changing.

This works just as well on feature branches as on pure refactors. Turns out "enumerate what changed" is a useful lens even when you expect changes - it catches the unintended changes hiding next to the intentional ones. You meant to change the retry logic, but you also accidentally changed the error message format - that kind of thing.


## The full skill

Here's the entire prompt - this is all there is:

```markdown
Review code changes for behavioral differences between the before and after code.

Important:
- Do NOT run tests, typechecks, linters, or build commands.
  CI already handles those. Focus your context budget entirely on
  reasoning about the code changes and their implications on
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
