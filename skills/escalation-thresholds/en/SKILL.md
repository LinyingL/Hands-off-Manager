---
name: escalation-thresholds
description: >
  Use before asking the human any question during implementation. A pure lookup
  table: Never Escalate / Always Escalate / Conditional. If the topic is not on
  this table, handle it per default-decision-policy. Pair with
  default-decision-policy for the full decision boundary.
---

# Escalation Thresholds

A reference table. Check here before asking the human anything.

## Never Escalate

Handle autonomously per `default-decision-policy`:

- Naming (variables, functions, files, tests)
- Internal function decomposition
- Test fixtures, mocks, test file organization
- Import ordering, barrel files
- Log messages and log levels
- Comments
- Equivalent TS constructs (`interface`/`type`, `enum`/`const`)
- Internal error message wording
- Code formatting (Prettier/ESLint)
- Commit message wording
- Inline TODOs (add them)
- Function order within a file
- Function vs method (follow local pattern)

## Always Escalate

| Category | Escalate when... |
|---|---|
| **Data & Schema** | Any schema change (table, column, index, migration), serialization format change, retention/deletion logic change |
| **API & Interfaces** | Any public API change (endpoints, request/response shapes, HTTP methods, status codes) |
| **Security & Auth** | Any change to auth flow, authorization rules, token handling, session management, security headers |
| **Business Logic** | Pricing/billing/payment logic, user-visible behavior, notifications, dashboard data |
| **Infrastructure** | New/removed dependency, env variable contracts, CI/CD config, deploy/build config |
| **Irreversible** | Deleting code paths in use, removing DB columns/tables, dropping format/API version support, non-reversible migrations |

## Conditional Escalation

| Decision | Escalate only if... |
|---|---|
| Cross-module refactor | Touches 4+ modules or changes a shared interface |
| Performance optimization | Changes algorithmic complexity or adds caching |
| New abstraction (> 20 lines) | Only 1 call site (no existing reuse need) |
| Error handling change | Affects user-visible error messages |
| Test strategy change | Switches approach (unit→integration, mock→real) |
| Moving files | Affects 3+ files' import paths |
| Shared type change | Type used in 5+ files |
| New env variable | Required (not optional with default) |
| Default config change | Affects production behavior |

## Escalation Format

```
DECISION NEEDED: [one-line summary]
Context: [what you're doing and why this came up]
Option A: [recommended] — [reason]
Option B: [alternative] — [tradeoff]
Risk if wrong: [low/medium/high]
```

Maximum 3 options. Always recommend one.
