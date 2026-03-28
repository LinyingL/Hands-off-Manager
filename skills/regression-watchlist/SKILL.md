# Regression Watchlist

## Purpose

A living reference document that catalogs the parts of the codebase most likely to break when something else changes. This is not a replacement for tests—it's a **checklist that tells you which tests to run and which areas to inspect** when working near high-risk code.

## When This Skill Activates

Before modifying any file listed in the watchlist below, or any file that imports from a watched module. Also activates during the evidence-gathering phase of `evidence-based-change-report` for changes that touch risk areas.

## How to Use This Skill

1. Before starting a change, check if any files in your change plan appear in the watchlist.
2. If yes, note the **required checks** listed for that area.
3. After completing the change, run those checks and include results in your change report.
4. If a check fails unexpectedly, **stop and escalate** before claiming done.

## Watchlist Template

> **You must customize this section for your project.** The categories below are common for full-stack TypeScript projects. Replace the example paths and checks with your actual modules.

### Authentication & Session Management

```
Watch: src/services/auth*, src/middleware/auth*, src/utils/token*
Invariants:
  - Expired tokens must always return 401, never silently succeed
  - Refresh flow must not create duplicate sessions
  - Role checks must happen after token validation, never before
Required checks:
  - npm test -- --grep "auth"
  - npm test -- --grep "session"
  - Manual: hit /api/me with expired token, confirm 401
```

### Database Access & ORM Layer

```
Watch: src/db/*, src/models/*, src/migrations/*, prisma/schema.prisma
Invariants:
  - All queries must use parameterized inputs (no string interpolation)
  - Migrations must be reversible or explicitly marked irreversible
  - Model changes must not silently drop columns
Required checks:
  - npx prisma validate (or equivalent)
  - npm test -- --grep "repository|model|migration"
  - Manual: review generated SQL for any new query
```

### Shared Types & Interfaces

```
Watch: src/types/*, src/shared/*, src/contracts/*
Invariants:
  - Changing a shared type must not break downstream consumers silently
  - Optional fields added must have sensible defaults at all usage sites
Required checks:
  - npx tsc --noEmit (full project type check)
  - Verify all files importing the changed type still compile
```

### API Route Handlers

```
Watch: src/routes/*, src/controllers/*, src/api/*
Invariants:
  - Response shapes must match documented/typed contracts
  - Error responses must use consistent format
  - No endpoint should return 500 for expected error cases
Required checks:
  - npm test -- --grep "api|route|endpoint"
  - Manual: test the modified endpoint with valid and invalid input
```

### State Management (Frontend)

```
Watch: src/store/*, src/context/*, src/hooks/use*State*
Invariants:
  - State updates must not trigger infinite re-render loops
  - Derived state must stay consistent with source state
  - Async state must handle loading, error, and success
Required checks:
  - npm test -- --grep "store|context|state"
  - Manual: verify no console warnings about state updates
```

### Shared Utilities

```
Watch: src/utils/*, src/lib/*, src/helpers/*
Invariants:
  - Utility functions must be pure (no side effects) unless explicitly documented
  - Changing a utility signature must update all call sites
Required checks:
  - npx tsc --noEmit
  - npm test -- --grep "util|helper|lib"
```

## How to Maintain This Watchlist

This document should be updated when:

- A bug is found that was caused by an unexpected side effect in a seemingly unrelated module (add that module to the watchlist)
- A new core module is added (auth provider, payment service, etc.)
- A previously stable module starts breaking frequently
- An entry hasn't been relevant for 6+ months (consider removing it)

The goal is to keep this list **short and high-signal**. If everything is on the watchlist, nothing is.

## Integration with Other Skills

- **bounded-change-planning**: The change plan should flag when watched files are in scope.
- **evidence-based-change-report**: Changes to watched modules must include the required checks in the evidence section.
- **escalation-thresholds**: Breaking a watchlist invariant during implementation is always an escalation trigger.
