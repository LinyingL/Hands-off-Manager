# Escalation Thresholds

## Purpose

A fixed lookup table that answers one question: **"Do I need to ask the human about this, or can I just decide?"** This skill eliminates ambiguity about when to escalate. If it's not on the escalation list, handle it yourself per `default-decision-policy`.

## When This Skill Activates

Before asking the human any question during implementation. Check this table first. If the topic is in the "Never Escalate" section, do not ask. If it's in "Always Escalate," ask immediately. If it's in "Conditional," check the conditions.

## Never Escalate (Handle Autonomously)

Decide these yourself. Follow existing conventions. See `default-decision-policy` for how.

- Naming: variables, functions, files, test descriptions
- Internal function decomposition within a single module
- Test fixture data, mock structure, test file organization
- Import ordering, barrel file organization
- Log message wording and log levels
- Comment content and placement
- Choice between equivalent TS constructs (`interface`/`type`, `enum`/`const`, etc.)
- Error message wording for internal errors
- Code formatting (Prettier/ESLint handles this)
- Git commit message wording
- Whether to add inline TODOs (yes, add them)
- Order of function definitions within a file
- Whether a helper should be a function or a method (follow local pattern)

## Always Escalate

Stop and ask before proceeding. Use the escalation format from `default-decision-policy`.

**Data & Schema:**
- Any database schema change (new table, column, index, migration)
- Any change to data serialization format (JSON shape, protobuf, etc.)
- Any change to data retention or deletion logic

**API & Interfaces:**
- Any change to public API contracts (REST endpoints, GraphQL schema, RPC interfaces)
- Any change to request/response shapes that external consumers depend on
- Adding, removing, or renaming API endpoints
- Changing HTTP methods or status codes for existing endpoints

**Security & Auth:**
- Any change to authentication flow
- Any change to authorization rules or role definitions
- Any change to token handling, session management, or credential storage
- Adding or modifying CORS, CSP, or other security headers

**Business Logic:**
- Pricing, billing, or payment calculation logic
- User-visible behavior changes (UI text, feature flags, default settings)
- Email, notification, or messaging content/triggers
- Any logic that affects data users see in their dashboard/reports

**Infrastructure & Dependencies:**
- Adding a new external dependency (npm package)
- Removing or replacing an existing dependency
- Changing environment variable contracts
- Modifying CI/CD pipeline configuration
- Changing deployment, build, or bundling configuration

**Irreversible Actions:**
- Deleting code paths that might still be in use
- Removing database columns or tables
- Dropping support for a data format or API version
- Any migration that cannot be rolled back

## Conditional Escalation

Escalate only if the stated condition is true.

| Decision | Escalate if... |
|---|---|
| Cross-module refactor | Touches 4+ modules or changes a shared interface |
| Performance optimization | Changes algorithmic complexity or adds caching |
| New abstraction (class, interface > 20 lines) | No existing reuse need (only 1 call site) |
| Error handling strategy change | Affects user-visible error messages or error reporting |
| Test strategy change | Switches testing approach (unit→integration, mock→real) |
| Moving files between directories | Affects more than 3 files' import paths |
| Changing a shared type definition | Type is used in 5+ files |
| Adding a new environment variable | It's required (not optional with default) |
| Changing default configuration | Affects production behavior |

## Escalation Format

When you do escalate, be efficient. Use this format:

```
DECISION NEEDED: [one-line summary]
Context: [what you're doing and why this came up]
Option A: [recommended] — [reason]
Option B: [alternative] — [tradeoff]
Risk if wrong: [low/medium/high]
```

Maximum 3 options. Always recommend one. If you can't recommend, explain the core tradeoff in one sentence.

## Meta-Rule

If you're unsure whether something should be escalated, check if it's closer to the "Never" list or the "Always" list. If it's closer to "Never," handle it. If it's genuinely ambiguous, escalate—but this should be rare. The whole point of this table is to remove ambiguity.
