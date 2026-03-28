# Bounded Change Planning

## Purpose

Prevent code changes from sprawling across the codebase. Every task starts with a **change plan** that explicitly bounds what will be touched. If the plan grows beyond budget, stop and re-scope before writing more code.

## When This Skill Activates

Before writing any code for a task that:

- Touches more than 1 file
- Might require creating new files
- Involves modifying shared modules, utilities, or types
- Is a bug fix in unfamiliar code
- Requires introducing a new pattern, abstraction, or dependency

## The Change Plan

Before coding, produce a **change plan** in this format:

```
CHANGE PLAN: [task title]

Files to modify:
  - src/services/user.service.ts — add email validation call
  - src/utils/validation.ts — extract shared email regex

Files to create:
  - (none)

Files to delete:
  - (none)

Estimated diff: ~40 lines

Dependencies added: none

Risk areas: validation.ts is imported by 3 other services
```

Rules for the plan:

1. **List every file you expect to touch.** No "and possibly others."
2. **Justify each new file.** If you're creating a new file, explain why an existing file won't work.
3. **State the estimated diff size.** You don't need to be exact, but you need a number.
4. **Flag risk areas.** Any shared module, any file imported by 3+ other files, any file that touches auth/data/API.

## Budget Limits

These are the default thresholds. If a task exceeds any of these, **stop and re-plan** before continuing:

| Metric | Limit | What to do when exceeded |
|---|---|---|
| New files created | 3 | Re-scope. Ask: can this reuse existing files? |
| Files modified | 8 | Split into subtasks. Ship the first chunk. |
| Estimated diff lines | 300 | Break into independent, shippable increments. |
| New dependencies | 1 | Justify in the plan. Get human approval. |
| New abstractions (class/interface/type > 20 lines) | 2 | Prove reuse need. No speculative abstractions. |

When you hit a limit mid-implementation (not caught in planning):

1. Stop writing code immediately.
2. Output a revised change plan showing the current state.
3. Propose how to split the remaining work.
4. Wait for approval before continuing.

## Principles

**Prefer modifying over creating.** Adding a function to an existing utility file is almost always better than creating a new file. A new file means a new import path, a new mental model, a new thing to maintain.

**Prefer deleting over abstracting.** If you see dead code while working, delete it. But do not create a new abstraction "to clean things up" unless the task specifically calls for it.

**Prefer narrow over general.** If a helper is only used in one place, keep it in that file. Extract only when you have 3+ concrete call sites.

**One concern per task.** If you discover a second problem while fixing the first, note it as a TODO and file it separately. Do not expand scope mid-task.

## Anti-Patterns

- ❌ "While I'm in here, let me also refactor..."
- ❌ Creating `types/`, `utils/`, `helpers/` directories for a single file each
- ❌ Introducing a `BaseService` or `AbstractHandler` with only one implementation
- ❌ Moving code to a new file "for better organization" when it's only used in one place
- ❌ Adding a config file or constant file for a single value
- ❌ "This might be useful later" as justification for anything

## Integration with Other Skills

- **escalation-thresholds**: New dependencies always require escalation.
- **evidence-based-change-report**: The completion report must reference the original change plan and explain any deviations.
- **surgical-refactor**: If a bug fix triggers refactoring urges, those go to a separate task.
