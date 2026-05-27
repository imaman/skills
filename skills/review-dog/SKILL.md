---
name: review-dog
description: Review code changes for bugs, style, best practices, and adherence to project conventions.
---

This skill guides creation of distinctive, expert-level code reviews that avoid generic "AI slop" — boilerplate observations, vague suggestions, parroting code back, and rubber-stamped clearances. Produce reviews that read like they came from a staff+ engineer who focuses on finding real problems in the concrete code at hand, not reciting common wisdom that sounds applicable but isn't. Every finding must be grounded in the context of the code being changed — its purpose, its callers, its invariants — not in what diffs like this tend to get wrong in general.

Important:

- Do NOT run tests, typechecks, linters, or build commands. CI already handles those. Focus your context budget entirely on reasoning about the code changes.
- Read CLAUDE.md and AGENTS.md before reviewing to ground your feedback in the project's actual rules.
- NEVER flag commit messages or commit titles as issues. Commit message quality — whether a title is descriptive, conveys the change, will produce a good squash title, etc. — is out of scope. Review only the code changes themselves, never the prose of the commits that carry them.

CLAUDE.md compliance:

- Do not rationalize away CLAUDE.md violations. If the rule says "ALWAYS", "NEVER", "MUST", or "MUST NOT", flag any deviation regardless of context. There are no exceptions based on perceived intent or severity.

Review technique:

1. Read the diff carefully.
2. Classify the changed files into clusters based on what they are part of. For each cluster, note its nature — for example: a high-traffic production endpoint, a shared internal library, an internal CLI tool, a standalone PoC app, a test helper, a CI/CD script, etc. Output these clusters before proceeding.
3. For each cluster, based on its nature and the specific changes in the diff, compile a list of review aspects that are most relevant. Aspects to consider (pick the ones that apply, add others as needed):

   - single source of truth for each piece of data
   - locality (things that change together are placed close to each other)
   - failure modes / sad paths
   - happy-path correctness
   - contracts and invariants
   - coupling
   - cohesion
   - abstraction completeness (entities whose usage sites repeat logic that could live inside the entity) 
   - CLAUDE.md compliance
   - security
   - naming
   - concurrency
   - error propagation
   - testing gaps
   - [add your own based on the diff at hand]

   Weight aspects according to the cluster's nature — e.g., failure modes and contracts matter more for a shared library than for a one-off script; security matters more for a production endpoint than for a test helper.

4. Review the code through each selected aspect, one cluster at a time. For each aspect, reason about whether the diff introduces any issues visible through that lens.
5. Compile findings into the output format below.

Quality bar:

- Every issue must be concrete: point to a specific line, explain what's wrong, and why it matters. No vague "consider improving" or "might want to look at" hand-waving.
- Do not parrot back what the code does. The author already knows that. Say what's _wrong_ with it or say nothing.
- Do not pad the review with generic praise, filler, or observations that don't lead to an actionable finding. If an aspect has no issues, just mark it as cleared — don't narrate the absence of problems.
- If you're not sure whether something is an issue, think harder rather than hedging with "this might be fine but..."

Output format:

- List issues first, numbered. Each finding has three parts, in this order:
  1. **Title line**: a tight one-liner (≤ ~12 words) naming the specific problem. Strong verbs, concrete nouns. Optimized for skim/prioritization — a reader should be able to tell from the title alone whether to dive in. Avoid generic titles like "logging issue" or "potential bug"; say what is wrong (e.g., "Logs full Shopify payload at info — hundreds of KB per request").
  2. **Subtitle line** (italicized, immediately under the title): `severity · aspect — path/to/file.ts:line`. Severity is one of critical / warning / nit.
  3. **Body**: the detailed explanation. Quote the offending code or point at specific identifiers; explain why it matters and what to do.
- After the issues section, add a "Cleared" section listing aspects that were reviewed and found to have no issues. Prefix each cleared item with a checkmark emoji.

Example of a single finding:

```
1. Logs full shopifyContext at info — emits hundreds of KB per request
   _warning · log hygiene — generate-segments-endpoint.ts:194_

   `ret` carries up to 50 product objects plus 50 collection objects with
   merchant-authored rich-text descriptions... [rest of the detailed body]
```

Scope (pass as arguments):

- No arguments: review the most recent commit
- `main`: review all commits since the last merge from main
- `HEAD~3`: review the last 3 commits
- Any git revision range (e.g., `abc123..HEAD`, `main..HEAD`)
