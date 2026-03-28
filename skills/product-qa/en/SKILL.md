---
name: product-qa
description: >
  Use after a feature or bug fix is complete, before reporting "done". Verifies
  the result from the user's perspective — not code quality, but whether the
  feature works as specified. Walks through normal flow, edge cases, and scope
  confirmation. Outputs an English-language acceptance report.
---

# Product QA

## Purpose

Verify features from the product and user perspective, not the code perspective. This skill answers: "Does what we built match what the user wanted?" — not "Is the code correct?" Code-level verification is handled by `evidence-based-change-report` and guardrails.

## When This Skill Activates

- A feature is complete, before reporting "done"
- The human says "test this," "does it work," or "verify it"
- A bug fix is complete and we need to verify the user experience is restored

## The QA Process

### Step 1: Pull the Requirement

Find the requirement summary document (produced by `requirement-clarification` skill). If no requirement document exists, ask the human: "After this feature ships, users should be able to [action]. Should I test it that way?"

### Step 2: Walk Through the User Flow

Follow the "normal flow" from the requirement and verify each step:

```
## Product Acceptance

### Normal Flow Verification

Step 1: User [action]
  Expected: [what they should see]
  Actual: ✅ As expected / ❌ Not as expected (actually saw [description])

Step 2: User [action]
  Expected: [what they should see]
  Actual: ✅ As expected / ❌ Not as expected

...
```

### Step 3: Test Edge Cases

Verify each edge case from the requirement:

```
### Edge Case Verification

Scenario: User enters a blank username
  Expected: Shows "Username is required"
  Actual: ✅ / ❌

Scenario: Network drops while submitting form
  Expected: Shows "Connection failed, please retry"
  Actual: ✅ / ❌

Scenario: User rapidly clicks submit button twice
  Expected: Only submits once
  Actual: ✅ / ❌
```

### Step 4: Check "Not Included" Items

Verify that things explicitly marked "not included" in the requirement were not implemented (prevents scope creep):

```
### Scope Confirmation

- This update does not include user deletion: ✅ Confirmed not implemented
- This update does not include batch operations: ✅ Confirmed not implemented
```

### Step 5: Produce the QA Report

Write an acceptance report in English:

```
## Product Acceptance Report

Feature: [feature name]
Date: [date]
Status: ✅ Pass / ❌ Fail / ⚠️ Partial Pass

### Summary

Normal flow: X/X steps passed
Edge cases: X/X passed
Scope check: ✅ Nothing extra, nothing missing

### Failed Items (if any)

1. [scenario description]
   Expected: [what]
   Actual: [what]
   Severity: 🔴 Blocks release / 🟡 Ship but fix soon / 🟢 Minor polish

### Overall Assessment

[One sentence: Can this be shipped to users? If not, what's the critical issue?]
```

## What This Skill Does NOT Check

These are handled by other skills and guardrails:

- Code quality → `code-health-standards`
- Test coverage → `evidence-based-change-report`
- Change scope reasonableness → `bounded-change-planning`
- Regression risk → `regression-watchlist`

Product QA only cares about: **what the user sees, what the user does, what the user experiences.**

## Common QA Scenarios to Always Check

Even if the requirement doesn't mention them, always test these scenarios:

### Form-Type Features

- Empty submission (submit without filling anything)
- Very long input (paste a large block of text)
- Special characters (quotes, angle brackets, emoji)
- Rapid repeat submission (quickly click submit button multiple times)

### List-Type Features

- Empty list (what displays when there's no data)
- Single item only
- Many items (does pagination/scrolling work correctly)

### State-Type Features

- Does state persist after page refresh
- Browser back button behavior
- Multiple tabs open simultaneously

### Error Scenarios

- Network disconnection
- Page loading state (loading spinners, etc.)
- Can the user retry after a failure

## Rules

1. **All output in English.** Per `plain-language-reporting`.
2. **Test as a user, not as a developer.** Don't examine source code. Check what the user sees and does.
3. **Be specific about failures.** "Not as expected" is not enough. Say what you actually saw.
4. **Severity must be practical.** 🔴 means "users cannot use this feature." 🟡 means "users can use it but will notice something off." 🟢 means "minor polish issue."
5. **Reference the original requirement.** Every test item traces back to the requirement summary. If something fails that wasn't in the requirement, note it as a "newly discovered issue" rather than a test failure.

## Anti-Patterns

- ❌ Only checking the happy path
- ❌ Saying "feature works" without listing what was tested
- ❌ Testing code paths instead of user actions
- ❌ Skipping QA because "all tests pass" (tests check code, QA checks product)
- ❌ Marking something as 🔴 when it's really 🟢 (inflating severity erodes trust)

## Integration with Other Skills

- **requirement-clarification**: The requirement summary is the test plan for QA. No requirement summary = no structured QA possible.
- **evidence-based-change-report**: Code-level evidence (tests, type check) appears in the change report. Product-level evidence (QA results) appears in the QA report. Both are needed for "done."
- **session-handoff**: QA failures go into the handoff's "Known Issues" section if not fixed in this session.
- **plain-language-reporting**: QA report is in English and uses product terms.
