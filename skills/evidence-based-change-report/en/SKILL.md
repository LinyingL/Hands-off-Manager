---
name: evidence-based-change-report
description: >
  Use when reporting that a task, subtask, bug fix, or code change is complete.
  Defines the completion report format: file list with diffs, verification
  evidence, deviation notes, and compliance checklist. Enforced by guardrails
  no-diff-no-done and no-evidence-no-success.
---

# Evidence-Based Change Report

## Purpose

Ensure that every claim of "done" is backed by concrete, verifiable evidence. No diff = no "done." No test = no "works." This skill defines the **report format**; the guardrails (`no-diff-no-done`, `no-evidence-no-success`) enforce it mechanically.

## When This Skill Activates

Every time you are about to report that a task, subtask, bug fix, or code change is complete.

## The Completion Report

Every completion claim must include a **change report** in this format:

```
CHANGE REPORT: [task title]

Status: DONE | PARTIAL | BLOCKED

Files changed:
  - src/services/auth.service.ts (+12 -3)
    Why: Added token refresh retry logic
  - src/services/auth.service.test.ts (+45 -0)
    Why: Added tests for retry logic, timeout, and failure cases

Files created:
  - (none)

Files deleted:
  - (none)

Deviation from plan:
  - (none, or: "Originally planned to modify api-client.ts but the retry
    logic fit entirely in auth.service.ts")

Evidence:
  - Tests: 14 passed, 0 failed (npm test -- --grep "auth")
  - TypeCheck: tsc --noEmit passed
  - Lint: eslint passed
  - Manual verification: [describe what you checked, e.g., "called
    /api/refresh with expired token, confirmed retry succeeded"]
```

## Required Evidence by Task Type

| Task type | Required evidence |
|---|---|
| Bug fix | Failing test before fix, passing after. Reproduction steps. |
| New feature | New tests covering happy path + at least 1 error case. Type check passes. |
| Refactor | All existing tests still pass. No new tests needed unless behavior changed. |
| Dependency update | Full test suite passes. Build succeeds. |
| Config change | Relevant service starts without error. |
| Type/interface change | `tsc --noEmit` passes across the project. |

## Rules

1. **No diff, no "done."** If you did not produce a code change (verified by actual file modifications), you cannot say "I've completed the code changes." You must instead say "No code changes were made" and explain why.

2. **No evidence, no "works."** If you did not run tests, type checking, or linting, you cannot say "it works" or "the fix is correct." You must instead say "Changes written but not yet verified."

3. **Partial is fine.** If you completed part of a task, report status as PARTIAL, list what's done, what's remaining, and what's blocking you.

4. **Deviations must be explained.** If the actual changes differ from the change plan (see `bounded-change-planning`), the report must note what changed and why.

5. **Evidence must be reproducible.** "I checked and it looks right" is not evidence. Show the command you ran and its output.

## Forbidden Phrases (Without Evidence)

These phrases trigger the guardrails if used without the corresponding evidence:

- "I've fixed the bug" → requires: before/after test results
- "The code has been updated" → requires: file list with line counts
- "This should work now" → requires: test or runtime evidence
- "I've implemented the feature" → requires: full change report
- "The changes are complete" → requires: full change report
- "I've refactored X" → requires: test suite still passes

## Example: Good vs Bad

**Bad:**
> I've updated the auth service to handle token refresh retries. The changes look correct and should fix the timeout issue.

**Good:**
> CHANGE REPORT: Fix auth token refresh timeout
>
> Status: DONE
>
> Files changed:
>   - src/services/auth.service.ts (+12 -3) — Added retry with exponential backoff
>   - src/services/auth.service.test.ts (+45 -0) — Tests for retry, timeout, max retries
>
> Evidence:
>   - Tests: `npm test -- auth.service` → 14 passed, 0 failed
>   - TypeCheck: `npx tsc --noEmit` → clean
>   - Before fix: test "should retry on 401" failed with timeout
>   - After fix: same test passes in 340ms

## Compliance Checklist (Required at End of Every Report)

Every change report must end with the following checklist. Mark each item that was applicable to this task. If a guardrail was triggered, state what happened. If it was not applicable, mark N/A.

```
## Compliance Status

| Rule | Triggered? | Result |
|---|---|---|
| Understand before modify (G4) | ✅ Triggered / ⬜ N/A | [Brief: which files were understood, what dependencies discovered] |
| Verify after each file (G5) | ✅ Triggered / ⬜ N/A | [Brief: how many files modified, did verification pass each time] |
| Revert before retry (G6) | ✅ Triggered / ⬜ N/A | [Brief: did you hit failures, did you revert, which attempt succeeded] |
| Changes within budget (G3) | ✅ Within budget / ⚠️ Exceeded | New files: X/3  Modified files: X/8  Diff lines: X/300 |
| Actual diff before claiming done (G1) | ✅ Has changes / ⬜ No changes | — |
| Evidence before claiming success (G2) | ✅ Verified / ⬜ Unverified | [List which verifications were run] |
| Low-stakes decisions handled autonomously (S1) | [number] autonomous decisions | [Brief: what was the largest decision] |
| Decisions escalated (S4) | [number] escalations | [Brief: what was escalated] |
```

### Rules for the Checklist

1. **Every applicable guardrail must have a concrete result**, not just a checkmark. "Triggered" alone is not enough—say what happened.
2. **If a guardrail was violated and then corrected**, report both: "First forgot to verify before moving to next file, caught the mistake and reverted to re-verify."
3. **The budget row must always show numbers.** This lets the human spot trends across tasks (e.g., tasks keep hitting 280/300 lines = budget might be too tight, or scope creep is still happening).
4. **Self-reported violations are expected and good.** The purpose is transparency, not punishment. An honest "Skipped G5 because I only modified comments in one file" is better than a fake checkmark.

### What the Human Should Watch For

Patterns across multiple task reports that signal a problem:

- **G6 (Revert) keeps triggering**: The Agent is struggling with this area of code. Consider getting a developer to review the architecture.
- **G3 (Budget) keeps hitting 7/8 or 290/300**: Tasks are consistently scoped too large, or the codebase is getting tangled. Consider breaking features into smaller increments.
- **G4 (Understand) keeps showing "discovered 8 dependent files"**: The module being changed is a high-coupling hotspot. Flag it for future refactoring.
- **S1 (Autonomous decisions) count is 0 across many tasks**: The Agent might be over-escalating. Check if `escalation-thresholds` needs tuning.
- **S4 (Escalations) count is 0 across many tasks**: Either the work is simple, or the Agent is under-escalating and making decisions it shouldn't.

## Integration with Other Skills

- **bounded-change-planning**: The change report references the original plan and explains deviations.
- **escalation-thresholds**: If evidence reveals unexpected breakage in unrelated areas, escalate before claiming done.
- **regression-watchlist**: For changes in high-risk areas, the report must include results from the watchlist's required tests.
- **plain-language-reporting**: The compliance checklist uses plain English terms. Technical command output goes in a collapsed section.
