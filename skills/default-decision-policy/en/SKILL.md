---
name: default-decision-policy
description: >
  Use when the Agent encounters an unspecified choice during implementation —
  naming, file organization, code style, internal decomposition, or any
  low-stakes decision. Prevents unnecessary interruptions by providing
  autonomous decision rules. Pair with escalation-thresholds for the
  full decision boundary.
---

# Default Decision Policy

## Purpose

Eliminate unnecessary interruptions by giving the Agent a clear framework for making low-stakes decisions autonomously. The goal is not to remove human judgment—it's to reserve human judgment for decisions that actually matter.

## When This Skill Activates

Any time during implementation when you encounter a choice that is **not explicitly specified** in the requirements, plan, or existing codebase conventions.

Examples: naming a helper function, choosing between `interface` vs `type`, picking a test fixture format, deciding where to put a utility, choosing log level, structuring a test file.

## Core Rule

**Default to the most boring, conservative, precedent-following option.** Do not innovate on things that don't matter.

## Decision Algorithm

When you face an unspecified choice:

1. **Check existing codebase conventions.** If the project already does it a certain way, do it that way. No discussion needed.
2. **If no convention exists, pick the simpler option.** Fewer files > more files. Fewer abstractions > more abstractions. Inline > extracted. Concrete > generic.
3. **If both options are equally simple, pick the one that touches fewer existing files.**
4. **If still tied, just pick one and move on.** Do not ask.

## What You Must NEVER Ask About

- Variable, function, or file naming (follow existing patterns; if none, use the most obvious descriptive name)
- Import organization or ordering
- Test data shape or fixture structure
- Internal function decomposition within a single file
- Log message wording or log levels (use `info` for happy path, `warn` for recoverable, `error` for unrecoverable)
- Comment style or placement
- Choice between equivalent TypeScript constructs (`interface` vs `type`, `enum` vs `const object`, `for...of` vs `.forEach`)
- Whether to add a `TODO` comment (yes, add it and move on)
- Barrel file (`index.ts`) organization

## What You MUST Escalate

See the companion skill **escalation-thresholds** for the complete threshold table. In summary, escalate when the decision affects:

- API contracts or public interfaces
- Database schema or migrations
- Authentication / authorization logic
- Data that crosses service boundaries
- Anything a user can see or that changes user-visible behavior
- Pricing, billing, or payment logic
- Deletion of existing code paths that might still be in use
- Introduction of a new external dependency

## How to Escalate

When you do need to escalate, format it as:

```
DECISION NEEDED:
Context: [1-2 sentences on what you're doing]
Option A: [description] — recommended because [reason]
Option B: [description] — tradeoff: [what you'd lose]
Risk if wrong: [low/medium/high + why]
```

Do not present more than 3 options. Always state which one you recommend and why. If you can't recommend one, say so and explain the tradeoff clearly.

## Anti-Patterns

- ❌ Asking "should I use X or Y?" when the codebase already uses X everywhere
- ❌ Presenting 4+ options with pros/cons tables for a naming choice
- ❌ Asking permission to create a test helper that only one test file uses
- ❌ Stopping work to ask about formatting when Prettier/ESLint is configured
- ❌ Asking whether to add error handling (yes, always add it)
