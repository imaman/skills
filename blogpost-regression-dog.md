---
title: I built a 21-line skill for Claude Code that distrupted my workflow.
published: false
tags: ai, agents, coding, codereview
---

## What it does

[regression-dog](https://github.com/imaman/skills) doesn't review code. It detects behavioral deltas. The entire prompt boils down to: *look at the code before, look at the code after, enumerate everything that changed in behavior.* Not "find bugs." Not "suggest improvements." Just: what used to do X that now does Y?

That reframe changes everything.

## Why the question matters

Most review bots ask a vague question: "what's wrong with this code?" That's a terrible prompt. It invites the model to free-associate about edge cases, naming conventions, and hypothetical issues. It's the code equivalent of asking someone "what do you think?" about a 200-page document.

regression-dog asks a *well-formed* question: given two snapshots, what behavioral differences exist between them? That's concrete. Bounded. The kind of question an LLM can actually nail.

## The trick that makes it work

The output format requires listing "Cleared" items — things that were reviewed and found to have no behavioral change. This looks like it's for the human reader, but it's actually for the model. By forcing the LLM to explicitly acknowledge what *didn't* change, it does a better job catching what *did*. It's a grounding mechanism disguised as formatting.

## Where it shines

I built it for refactoring. When you're restructuring code without intending to change behavior, knowing what *actually* changed is the whole game. Pure refactors should have zero regressions. When regression-dog finds one, it's almost always a real bug.

But I ended up using it just as much on feature work. Even when you *expect* behavioral changes, knowing the full list of deltas — including the ones you didn't intend — is a powerful signal. The planned changes confirm you did what you meant to. The unplanned ones are where the bugs hide.

## My workflow

When a chunk of work is ready:

1. Open a fresh Claude Code session
2. Type "flag all regressions in the branch"
3. Read the output — fix what needs fixing (often I copy-paste the findings into a separate Claude Code session to do the actual fix)
4. Repeat until every regression is either gone or intentional

That's it. No dashboard, no PR integration, no configuration. Just a question, an answer, and a loop.

## Try it

```bash
npx skills add https://github.com/imaman/skills --skill regression-dog
```

It's 21 lines. MIT licensed. The prompt is the product — read it, steal it, improve it.
