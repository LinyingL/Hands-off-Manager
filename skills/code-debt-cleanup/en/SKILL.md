---
name: code-debt-cleanup
description: >
  Use when entering a module not recently touched (lightweight scan), or after
  a milestone/feature ships (full scan). Detects dead exports, orphaned
  patterns, growing files, dependency direction violations, and duplication.
  Produces an English-language health report with severity and trend tracking.
---

# Code Debt Cleanup

## Purpose

Proactively find and report accumulating technical debt before it becomes a problem. This skill defines a periodic health scan that the Agent runs at natural breakpoints—not as a separate task, but woven into normal work.

## When This Skill Activates

Run a **lightweight scan** (Part 1 only) at the start of every task that touches a module the Agent hasn't worked on in the current session.

Run a **full scan** (Parts 1-3) when:
- A milestone or feature is completed
- The human explicitly asks for a health check
- The compliance checklist shows 3+ consecutive tasks with health warnings

## Part 1: Lightweight Scan (< 2 minutes)

When entering a module to work on a task, quickly check:

```bash
# 1. Dead exports: symbols exported but never imported elsewhere
# 2. Unused files: files not imported by anything
# 3. TODO count: how many TODOs are in this module
# 4. File sizes: any file over 300 lines
```

Report findings as a single line in the change report:

```
Code health note: This module has 2 unused exported functions, 1 file over 350 lines, 4 TODOs without ticket references.
```

Do NOT fix these during the current task. Note them only. Per `surgical-refactor`, cleanup is a separate task.

## Part 2: Full Scan — Structural Health

Check the overall project structure against `code-health-standards`:

### 2.1 Dependency Direction

Verify that imports flow downward (api → services → repositories). Flag any upward imports:

```
⚠️ Dependency direction issue:
  - src/repositories/user.repo.ts imports src/services/auth.service.ts
    This is a lower-level module importing from an upper-level module, creating circular dependency risk.
```

### 2.2 Duplication Detection

Look for patterns that suggest copy-paste:

- Two files with suspiciously similar function names
- Identical error handling blocks across multiple files
- Same data transformation logic in different services

```
⚠️ Suspected duplication:
  - src/services/user.service.ts and src/services/order.service.ts
    Both have similar "verify permission then fetch data" logic (about 20 lines).
    Consider extracting as a shared permission-checking utility function.
```

### 2.3 Orphaned Patterns

Check if the project has multiple ways of doing the same thing:

```
⚠️ Two different approaches exist:
  - HTTP requests: src/utils/api-client.ts and src/utils/fetch-wrapper.ts
    Both handle HTTP requests. 12 files use the former, 3 files use the latter.
    Recommend consolidating to api-client.ts and removing fetch-wrapper.ts.
```

### 2.4 Growing Files

Flag files that are approaching or exceeding healthy limits:

```
⚠️ File size warning:
  - src/services/order.service.ts: 480 lines (recommended max 300 lines)
    Last 3 tasks all added logic here. Consider splitting into:
    - order-creation.service.ts (order placement flow)
    - order-fulfillment.service.ts (fulfillment flow)
```

## Part 3: Full Scan — Debt Inventory

Compile a debt summary in English:

```
## Code Health Report

Last scan: [date]
Scope: [full project / specific module]

### Issues to Address (by priority)

1. 🔴 High: order.service.ts at 480 lines, becoming hard to maintain
   Recommendation: Schedule a split in the next iteration, estimated 2-3 hours of work

2. 🟡 Medium: Project has two HTTP request wrappers, easy to confuse
   Recommendation: Consolidate to one, estimated 1 hour of work

3. 🟢 Low: 12 TODOs without ticket references
   Recommendation: Add ticket references when touching these files next time

### Positive Trends

- Last 5 tasks all completed within budget, scope control is good
- No new `any` types introduced
- Test coverage remains stable in this module

### Numbers Overview

| Metric | Current | Previous | Trend |
|---|---|---|---|
| Total files | 84 | 78 | ↑ Normal growth |
| Files over 300 lines | 3 | 2 | ⚠️ Watch |
| Unreferenced exports | 7 | 5 | ⚠️ Accumulating |
| TODOs without tickets | 12 | 8 | ⚠️ Accumulating |
| Circular dependencies | 0 | 0 | ✅ Healthy |
```

## How This Connects to Your Workflow

The lightweight scan (Part 1) happens silently—you'll just see one line in the change report. No extra work for you.

The full scan (Parts 2-3) produces an English-language health report. You read it like a physical checkup report: red items need scheduling, yellow items need awareness, green items can wait. The "Trend" column tells you whether things are getting better or worse.

**You never need to understand the code to act on this report.** Each item tells you: what the problem is in plain terms, what to do about it, and how much effort it takes. You can hand it to the Agent as a task ("fix that high-priority order.service split") or to a developer for review.

## When to Run the Full Scan

Suggested cadence (adjust based on your project pace):

- **After every major feature ships**: Good checkpoint to see what debt the feature introduced
- **Every 2 weeks if in active development**: Catches slow accumulation
- **Before a release**: Ensures no surprise structural issues
- **When you feel something is "off"**: "Modifying things lately always hits problems" → run a full scan to see if there's structural rot

## Anti-Patterns

- ❌ Running a full scan after every small task (too noisy)
- ❌ Fixing debt items during feature work (per `surgical-refactor`, these are separate tasks)
- ❌ Ignoring the lightweight scan notes across 10+ tasks (debt is accumulating silently)
- ❌ Treating all debt as urgent (some debt is fine to live with)
