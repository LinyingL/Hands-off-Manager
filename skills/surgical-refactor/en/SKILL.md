---
name: surgical-refactor
description: >
  Use when fixing a bug near messy code, or when tempted to refactor while
  doing unrelated work. Enforces strict separation: bug fixes and refactors
  never share a commit. One new concept per task. Delete before abstract.
  Prove behavior before changing structure.
---

# Surgical Refactor

## Purpose

Prevent "refactoring drift"—the tendency to expand scope when touching messy code. Bug fixes fix bugs. Refactors refactor. They do not happen in the same commit, the same PR, or the same task.

## When This Skill Activates

- You're fixing a bug and notice the surrounding code is messy
- You're reading code to understand it and feel the urge to "clean it up"
- You want to rename, restructure, or extract something while doing unrelated work
- Someone says "while you're in there, can you also..."

## Core Rules

### Rule 1: Separate the Fix from the Refactor

A bug fix commit contains **only** the minimal change needed to fix the bug. If the surrounding code is ugly, that's a separate task.

```
✅ Commit 1: "Fix off-by-one in pagination query"  (3 lines changed)
   Commit 2: "Refactor pagination module for clarity" (40 lines changed)

❌ Commit 1: "Fix pagination bug and clean up pagination module" (43 lines changed)
```

Why: If the refactor introduces a new bug, you can't tell whether the original fix was correct. Reviewers can't tell what's a behavioral change vs. a cosmetic one.

### Rule 2: One New Concept Per Task

Each refactoring task introduces **at most one** new concept:

- One new abstraction (class, interface, pattern)
- One new organizational principle (new directory, new module boundary)
- One new convention (error handling pattern, naming scheme)

If your refactor requires introducing two new concepts, split it into two tasks.

### Rule 3: Delete Before You Abstract

When you see duplicate or messy code:

1. **First, delete dead code.** Remove anything provably unused. This is cheap, safe, and immediately clarifying.
2. **Then, inline over-abstractions.** If a function/class has only 1 call site and the abstraction doesn't carry its weight, inline it.
3. **Only then, consider adding a new abstraction.** And only if you have 3+ concrete, existing call sites.

```
Refactor priority:
  Delete dead code    > Inline unnecessary abstraction
  Inline unnecessary  > Extract shared logic
  Extract shared      > Introduce new pattern
```

### Rule 4: Prove Behavior First, Then Change Structure

Before changing how code is organized:

1. Ensure existing tests cover the current behavior.
2. If tests are missing, **add tests first in a separate commit**.
3. Then refactor, verifying tests still pass after each step.

```
✅ Commit 1: "Add missing tests for user validation" (+60 lines, tests only)
   Commit 2: "Extract validation into dedicated module" (+20 -40 lines, refactor only)

❌ Commit 1: "Refactor validation and add tests" (+80 -40 lines, mixed)
```

### Rule 5: Resist "While I'm Here"

If you discover something worth improving while doing another task:

1. Add a `// TODO: [description]` comment at the site.
2. Note it in your change report under a "Future Work" section.
3. Do not act on it now.

The only exception: deleting a line of obviously dead code (unreachable branch, unused import) that you encounter directly in the file you're already editing. This is low-risk enough to do inline.

## Refactor Checklist

Before starting a refactoring task, answer these:

- [ ] Is this refactor separated from any bug fix or feature work?
- [ ] Do tests exist for the code I'm about to restructure?
- [ ] Am I introducing at most one new concept?
- [ ] Do I have 3+ concrete call sites for any new abstraction?
- [ ] Is my change plan within the `bounded-change-planning` budget?

If any answer is "no," address that first.

## Anti-Patterns

- ❌ "Let me just clean this up while I'm fixing the bug"
- ❌ Extracting a `BaseService` for one service
- ❌ Creating a `utils/helpers.ts` for a single helper function
- ❌ Renaming everything in a module while fixing a typo in one place
- ❌ "This whole module needs a rewrite" (it probably just needs 3 small, targeted fixes)
- ❌ Changing code style in files you're not functionally modifying
- ❌ Adding generics or type parameters "for future flexibility"

## Integration with Other Skills

- **bounded-change-planning**: Refactor tasks get their own change plan with their own budget.
- **evidence-based-change-report**: Refactor reports must show "all existing tests still pass" as primary evidence.
- **regression-watchlist**: Refactoring watched modules requires running the full check suite for that area.
