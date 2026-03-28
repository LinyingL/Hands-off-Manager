---
name: session-handoff
description: >
  Use at end of every session (or when conversation is winding down) to write
  a handoff memo to .claude/handoff.md. Use at start of every new session to
  read the existing handoff and resume context. Preserves progress, decisions,
  known issues, and project conventions across sessions.
  Note: ideally a lifecycle hook, implemented as skill until hook support exists.
---

# Session Handoff

## Purpose

Preserve context between sessions. Write a handoff memo before each session ends; read it when the next session starts.

## When This Skill Activates

- **Write handoff**: At the end of each session (when the human says "see you later," "continue next time," or "wrap up for today," or when you sense the conversation is concluding)
- **Read handoff**: At the start of a new session, if `.claude/handoff.md` exists in the project directory

## Handoff File Location

```
.claude/handoff.md
```

This file is overwritten each session (not appended). It represents the **current state**, not a history log.

## Writing the Handoff

At the end of a session, create or update `.claude/handoff.md` with this structure:

```markdown
# Session Handoff

Updated: [datetime]

## Current Progress

### Completed
- [Describe what was completed in product language, not code terms]
  - Files touched: [file list for navigation]

### In Progress (unfinished)
- [What is being worked on, and how far along is it]
  - Current state: [specifically how far along]
  - Next step: [what comes next]
  - Blockers: [if any]

### To Do (not started)
- [Planned tasks not yet started]

## Key Decisions

| Decision | Choice | Reason | Decided by |
|---|---|---|---|
| [decision description] | [which option] | [why chosen] | Human / Agent |

## Known Issues

- [Issues currently known but unfixed]
- [Potentially risky areas]

## Project Conventions

[Conventions established during this session that must continue to be followed]

- [Example: API responses always use { data, error, meta } format]
- [Example: All dates stored in UTC, converted to local timezone for display]

## File Map

[Key files touched during this session and their responsibilities, for quick navigation next time]

- `src/services/auth.service.ts` — login, logout, token refresh
- `src/middleware/auth.middleware.ts` — route-level permission checking
```

## Reading the Handoff

At the start of a new session:

1. Check if `.claude/handoff.md` exists.
2. If yes, read it **before doing anything else**.
3. Briefly acknowledge to the human: "I've read the previous handoff. Last time we finished [summary], and [summary] is still pending. Continue or new task?"
4. If the human wants to continue, pick up from the "In Progress" section.
5. If the human has a new task, keep the handoff context in mind (especially "Project Conventions" and "Known Issues").

## Rules

1. **Always write in English.** This file is for the human to read too, not just for the Agent.

2. **Product language, not code language.** The "Completed" section should say "Users no longer get logged out after refreshing the page" not "Added token refresh retry in auth.service.ts."

3. **File references go in sub-bullets.** The human doesn't need to see file paths in the main description, but the next session's Agent does need them for navigation. Put them in indented sub-bullets.

4. **Decision records are critical.** The single most common source of wasted work across sessions is re-debating a decision that was already made. Record every significant decision, who made it, and why.

5. **Overwrite, don't append.** Each session produces a fresh handoff. Old handoffs are not valuable — git history preserves them if needed.

6. **Don't ask permission to write the handoff.** Just write it at the end of the session. It's a background maintenance task, not something the human needs to approve.

7. **Keep it under 80 lines.** The handoff should be scannable in under 2 minutes. If the project has too much context to fit, focus on the "In Progress" and "Key Decisions" sections — those are what the next session needs most.

## When the Human Doesn't Explicitly End the Session

If the conversation seems to be ending naturally (long pause, task completed, human says "thanks" or similar), write the handoff proactively. Better to write one that isn't needed than to lose context.

## Anti-Patterns

- ❌ Not writing a handoff because "we finished everything" (there's always context worth preserving — conventions, decisions, known issues)
- ❌ Writing the handoff in code or technical jargon
- ❌ Appending to the previous handoff (creates a growing, unreadable log)
- ❌ Including raw code diffs in the handoff (that's what git is for)
- ❌ Asking the human "should I write a handoff?" (just do it)

## Integration with Other Skills

- **requirement-clarification**: Confirmed requirements go into the "Project Conventions" section.
- **evidence-based-change-report**: The completion report for the session informs the "Completed" section.
- **bounded-change-planning**: Unfinished plan items go into "In Progress" with their current state.
- **escalation-thresholds**: Pending decisions that were escalated but not yet answered go into "Blockers."
