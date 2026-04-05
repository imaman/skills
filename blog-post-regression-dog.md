---
title: A 20-line skill that replaced my code review safety net
published: false
tags: claudecode, ai, codereview, devtools
---

I stopped waiting for Greptile and co.

Instead I run a skill I call `regression-dog`. It is open source, simple (about 20 lines of markdown), and integrates right into my terminal/coding agent.

Initially, I imagined I'd use it to run a few fast iterations locally and then still lean on the dedicated review bots for the real deep analysis. To my surprise, it's usually just as thorough as those bots, and sometimes better. It consistently catches things I'd expect only a careful human reviewer to notice. Here is everything you need to know.

## What it does

It reads the diff of your branch (or last commit, or last N commits - you choose the scope) and lists every behavioral change it can find. A typical output looks like this:

[screenshot goes here]

## Let's unpack the magic

The whole prompt is meticulously crafted around keeping the LLM focused on a well-grounded task. Every line is there for a reason:

> Do NOT run tests, typechecks, linters, or build commands.

Protects the agent's context window so the underlying LLM can invest its reasoning where it matters.

> Enumerate behavioral differences: this used to do X, now it does Y.

It poses a specific, concrete question: what are all the behavioral changes in this diff? This is a grounded task as opposed to open-ended "review my code" prompts.

> Do not judge whether the old or new behavior is correct - just surface the delta.

Plays to the strengths of LLMs.

> Do not flag pre-existing issues or suggest improvements

Shuts down a distraction course agents love pursuing.

> Add a "Cleared" section listing items that were reviewed and found to have no issues.

This sounds cosmetic but it's critical and load-bearing. When the agent inspects a code change, it has two outlets: either the change is safe (goes to "Cleared") or it's a behavioral difference (goes to "Regressions"). This symmetry forces an explicit decision on every change instead of quietly skipping things it's unsure about. It improved recall noticeably when I added it.

## How I actually use it

I work on something. When I think it's ready, I open a fresh Claude Code session - fresh, so the review isn't biased by the conversation that produced the code - and type something like:

```
flag all regressions in this branch
```

It comes back fast - often within a minute, depending on the scope, of course. Anyhow, even in more complex changes it is significantly faster than the push-to-GitHub-wait-for-bot loop. And it's right there in my terminal, no tab-switching.

Then I fix things in that same session. It is already loaded with high quality context about the branch, so fixes are fast. I apply judgment - some "regressions" are intentional behavior changes, and I skip those. Then I open yet another clean session and run it again. Repeat until it comes back clean, or until every flagged item is something I'm deliberately changing.


## The full skill

Here's the entire prompt - this is all there is:

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
