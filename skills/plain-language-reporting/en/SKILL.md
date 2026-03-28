---
name: plain-language-reporting
description: >
  Use for ALL human-facing output — reports, escalations, status updates,
  health checks. Translates technical details into plain English product
  language. Technical specifics go in a collapsed section. Includes a
  tech-to-plain-English translation table. Overrides all other report formats.
---

# Plain Language Reporting

## Purpose

The human you report to has basic code knowledge. They understand what files, functions, APIs, and databases are, but they do not read code daily. Every report, escalation, and status update must be written so that a non-developer product owner can read it and make decisions.

## This Skill Overrides All Other Report Formats

When any other skill (evidence-based-change-report, bounded-change-planning, escalation-thresholds, etc.) asks you to produce a report, you still follow that skill's structure—but you **translate every technical detail into plain language**. The technical details go in a collapsed section at the bottom for reference.

## Reporting Template

Every report to the human must follow this structure:

```
## What was done

[1-3 sentences in plain English. Describe WHAT changed in terms of user-visible behavior or system behavior, not code structure.]

Example:
  ✅ "Users no longer get logged out when they refresh the page after signing in. Previously, the login state wasn't being preserved during a page refresh."
  ❌ "Modified auth.service.ts to add token refresh retry with exponential backoff in the interceptor chain."

## What changed

[List files changed, but describe each in terms of its ROLE, not its code path.]

Example:
  ✅ "Login-related backend logic (1 file) and its tests (1 file)"
  ❌ "src/services/auth.service.ts (+12 -3), src/services/auth.service.test.ts (+45 -0)"

## Any risks?

[State risks in terms of what could go wrong for users or the product, not in code terms.]

Example:
  ✅ "This change only affects login state refresh, not other features. Risk is very low."
  ✅ "This change touches core permission-checking logic. Although tests pass, recommend manual verification of admin vs. regular user permissions before release."
  ❌ "Risk: auth middleware is imported by 5 route handlers."

## Verification

[State what was tested in plain terms.]

Example:
  ✅ "All automated tests passed (14 tests). I manually simulated a scenario where a login expires and the user refreshes the page, confirming they don't get logged out."
  ❌ "npm test -- auth.service: 14 passed, 0 failed. tsc --noEmit clean."

---
<details>
<summary>Technical details (skip if not needed)</summary>

[Here put the full technical report as required by evidence-based-change-report, including exact file paths, diff stats, commands run, and raw output.]

</details>
```

## Escalation Template

When escalating a decision (per escalation-thresholds), translate it:

```
## Need your decision

[One sentence describing the situation in plain language.]

### Background
[Why this decision came up, in terms of what the product/feature needs.]

### Option A (recommended): [plain description]
Benefits: [in user/product terms]
Cost: [in user/product terms]

### Option B: [plain description]
Benefits: [in user/product terms]
Cost: [in user/product terms]

### If we choose wrong
[Describe impact in terms of: user experience, data safety, timeline, or maintenance cost. NOT in code terms.]

Example:
  ✅ "If we go with A, adding new user roles later will be easier. If we go with B, we ship one day faster now, but adding roles later requires changes in many places."
  ❌ "Option A uses a polymorphic role interface with O(1) lookup vs Option B's switch statement with O(n) maintenance cost."
```

## Size Budget Warning Template

When size-budget-check is triggered:

```
## Change scope larger than expected

This task turned out bigger than planned.

Already completed:
  - [plain description of what's done, in behavioral terms]

Still to do:
  - [plain description of what's left]

Suggested approach:
  1. Release now: [what can ship now and what it gives the user]
  2. Follow-up: [what comes next]

Need your confirmation that this split works.
```

## Translation Rules

### Always Translate These

| Technical term | Say instead |
|---|---|
| diff / changeset | changes made |
| dependency | external library/package |
| migration | database structure change |
| refactor | code reorganization (no behavior change) |
| type error / type check | type mismatch |
| lint | code style check |
| build | compile and package |
| endpoint | API endpoint |
| schema | data structure |
| abstract / abstraction | shared module |
| middleware | request processing layer |
| token | login credential |
| cache | temporary storage for speed |
| ORM | database access tool |
| CI/CD | automated testing and deployment |
| mock | simulated data |
| fixture | test data |
| assertion | test verification point |
| regression | old feature broken by new changes |
| dead code | unused old code |
| barrel file / index.ts | unified export file |
| monorepo | multiple projects in one codebase |
| generic / generics | generic type parameter |

### Keep These As-Is (The Human Knows Them)

- File names and paths (but add a parenthetical role description)
- API, database, frontend, backend
- Test, bug, feature
- Release, deploy
- Git, branch, commit

### When Unsure

If a term might confuse the human, add a brief parenthetical:

> "This change involves WebSocket (real-time communication connection) reconnection logic."

## Language

Default reporting language is **English**. Use English for all human-facing sections. Technical details section (in the collapsed block) can be in English or the project's default language since it's for reference/debugging only.

## Anti-Patterns

- ❌ Dumping raw command output without explanation
- ❌ Using file paths as the primary way to describe what changed
- ❌ Saying "3 files modified, +45 -12 lines" without saying what those changes DO
- ❌ Describing risks in terms of code coupling ("imported by 5 modules") instead of user impact
- ❌ Asking the human to evaluate code-level tradeoffs ("should I use a Map or an Object?")
- ❌ Assuming the human will read the technical details section
