---
name: code-health-standards
description: >
  Use when writing new modules, making architectural decisions, or when
  default-decision-policy says "follow existing conventions" — this document
  IS the conventions. Defines module boundaries, naming rules, anti-debt
  patterns, and health metrics for full-stack TypeScript projects.
  Note: this is a project template — customize module structure for your project.
---

# Code Health Standards

## Purpose

Define what "good code" looks like in this project, so that every task—whether feature, bug fix, or refactor—moves the codebase toward the same target instead of drifting. This is a **living reference** that the Agent maintains as the project grows.

## When This Skill Activates

- Before writing any new module, component, or service
- When making architectural decisions (which module does this belong in?)
- During `bounded-change-planning` to decide where new code should live
- When `default-decision-policy` says "follow existing conventions" — this document IS the conventions

## Part 1: Module Boundaries

> **You must fill this section in for your project.** Below is a template for a typical full-stack TypeScript project. The Agent should update this section whenever a new module is created or an existing boundary is clarified.

```
PROJECT STRUCTURE:

src/
  api/           → HTTP route handlers. No business logic here.
                   Only: parse request → call service → format response.

  services/      → Business logic. All domain rules live here.
                   Services call repositories, never the database directly.
                   Services never import from api/.

  repositories/  → Database access. Raw queries or ORM calls.
                   Returns plain data objects, not ORM entities.
                   Never imported by api/ directly.

  types/         → Shared type definitions. No runtime code.
                   If a type is only used in one file, keep it in that file.

  utils/         → Pure functions with zero side effects.
                   Must not import from services/, repositories/, or api/.
                   If a utility has more than 2 dependencies, it's not a utility.

  middleware/    → Request pipeline: auth, logging, error handling.
                   Must not contain business logic.

  config/        → Environment and app configuration.
                   Single source of truth. No scattered process.env reads.

Frontend:
  components/    → UI components. Presentational only when possible.
  hooks/         → Custom React hooks. State and side effect logic.
  pages/         → Route-level components. Compose components + hooks.
  store/         → Global state management.
```

### Boundary Rules

1. **Dependencies flow downward.** `api → services → repositories`. Never upward.
2. **No circular imports.** If A imports B and B imports A, one of them is in the wrong layer.
3. **Shared code goes in the lowest common ancestor.** If both `services/user.ts` and `services/order.ts` need a helper, it goes in `utils/`, not in a new `services/shared.ts`.
4. **One responsibility per file.** If a file has two unrelated exports, split it. If a file name needs "and" to describe it, split it.

## Part 2: Naming Conventions

### Files
- `kebab-case` for all file names: `user-service.ts`, not `UserService.ts`
- File name reflects the primary export: `user-service.ts` exports `UserService`
- Test files mirror source files: `user-service.ts` → `user-service.test.ts`

### Code
- `PascalCase` for classes, interfaces, type aliases, React components
- `camelCase` for functions, variables, methods, properties
- `SCREAMING_SNAKE_CASE` for true constants (compile-time known values only)
- Boolean variables start with `is`, `has`, `can`, `should`: `isActive`, `hasPermission`
- Event handlers start with `handle`: `handleSubmit`, `handleClick`
- Async functions that fetch data: `fetchUsers`, `loadOrders` (not `getUsers` — `get` implies synchronous)

### Patterns to Always Use
- **Early returns** over nested if/else: check error conditions first, return early, keep the happy path unindented
- **Explicit error handling**: every async call has a try/catch or .catch(). No unhandled promise rejections.
- **Const by default**: use `let` only when reassignment is genuinely needed
- **Named exports** over default exports (except React page components)

### Patterns to Never Use
- `any` type — use `unknown` and narrow, or define a proper type
- Nested ternaries — use if/else or early return
- Magic numbers/strings — extract to named constants
- `console.log` for debugging in committed code — use the project's logger
- Index signatures (`[key: string]: any`) without a TypeScript comment explaining why

## Part 3: Anti-Debt Rules

These rules prevent the most common sources of technical debt accumulation.

### 1. No Dead Code

- Commented-out code blocks must be deleted, not left "for reference." Git history is the reference.
- Unused imports, variables, and functions must be removed immediately.
- If ESLint/TypeScript flags something as unused, fix it in the same commit.

### 2. No TODO Without a Ticket

- `// TODO:` comments are allowed but must reference a task/issue: `// TODO(PROJ-123): migrate to new auth flow`
- `// HACK:` and `// FIXME:` require the same treatment.
- TODOs without a reference will be flagged during code review.

### 3. No Copy-Paste Duplication

When you're about to copy a block of code:

1. If the block is < 5 lines: duplication is OK. The cure is worse than the disease.
2. If the block is 5-15 lines: extract to a shared function in the same file.
3. If the block is > 15 lines or appears in 3+ files: extract to `utils/` or a shared module.

### 4. No Leaky Abstractions

Every abstraction must hide complexity, not just move it:

- **Bad**: A `BaseService` that every service extends but that provides nothing useful (just constructor boilerplate)
- **Good**: A `withRetry(fn, options)` utility that encapsulates retry logic so callers don't think about it
- **Test**: If using the abstraction requires understanding its internals, it's leaky. Inline it.

### 5. No Orphaned Patterns

If the project has two ways of doing the same thing (e.g., two HTTP client wrappers, two date formatting approaches), the newer pattern must include a migration note:

```
// NOTE: This replaces the old date formatting in utils/format-date.ts.
// Remaining usages of the old approach: src/services/report.ts, src/services/export.ts
// TODO(PROJ-456): migrate remaining usages to this new approach
```

## Part 4: Health Metrics (Agent Self-Check)

The Agent should periodically assess (when starting a new task in an area it hasn't touched recently):

| Metric | Healthy | Warning | Action |
|---|---|---|---|
| Files in a single directory | < 10 | 10-15 | Consider subdirectories |
| Imports in a single file | < 10 | 10-15 | File may have too many responsibilities |
| Lines in a single file | < 300 | 300-500 | Look for extraction opportunities |
| Exported symbols from a module | < 8 | 8-12 | Interface may be too wide |
| Depth of function nesting | < 3 levels | 3-4 | Refactor with early returns |
| Number of function parameters | < 4 | 4-5 | Use an options object |

These are guidelines, not guardrails. The Agent does not stop work for these—it notes them in the change report under "代码健康观察" and suggests follow-up tasks if needed.

## Part 5: Maintaining This Document

This document is **alive**. The Agent should propose updates when:

- A new module is created → add it to the project structure
- A new naming convention is established → add it to naming rules
- A recurring code smell is found across multiple tasks → add it to anti-debt rules
- A health metric threshold proves too strict or too lenient → adjust it

Updates to this document are proposed in the change report and require human approval before being committed. The Agent must never silently change the standards.
