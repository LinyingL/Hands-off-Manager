---
name: requirement-clarification
description: >
  Use when the human gives a product-level request that hasn't been broken down
  into specifics — "Build me an XX feature", "I want XX", "XX has a problem,
  fix it". Translates vague ideas into actionable specs using scenario-based
  questions. Outputs a confirmed requirement summary before any code is written.
---

# Requirement Clarification

## Purpose

Translate vague product-level requests into actionable technical specs. Before writing any code, ensure both sides are fully aligned on "what to build." This skill manages Agent input quality—if the input is unclear, all downstream skills are wasted.

## When This Skill Activates

When the human gives a task that involves building, changing, or fixing functionality. Specifically:

- "Build me an XX feature"
- "I want XX"
- "XX has a problem, fix it"
- "Can we make XX work like YY"
- Any product-level request that hasn't been broken down into technical specifics

Do NOT activate for:

- Purely technical tasks with clear specs ("change this function's return type from string to number")
- Questions or discussions ("how is XX implemented?")
- Maintenance tasks that don't change behavior ("upgrade dependencies", "run tests")

## The Clarification Process

### Step 1: Restate in Plain English

Before asking any questions, restate what you understood in one sentence:

```
Here's what I understand you want: [one sentence, no technical jargon]
```

If you can't restate it clearly, that's a signal you need more context. Say so.

### Step 2: Ask the Four Essential Questions

Every feature or change needs answers to these four questions. Ask them using **concrete scenarios**, not abstract concepts:

**1. Who uses this? In what scenario?**

Don't ask: "Who is the target user?"
Do ask: "Is this feature for all users or just admins?" or "When would a user actually need this? Every day, or just occasionally?"

**2. What's the normal flow?**

Don't ask: "Describe the user journey."
Do ask: "Walk me through it: user opens the page, what's step one? Then what? What do they see at the end?"

**3. What happens when something goes wrong?**

Don't ask: "What's the error handling strategy?"
Do ask: "If the user enters incorrect info, what happens? Like if an email format is wrong, do we show an error message or prevent submission?" or "If the network drops while the user is working, will they lose their data?"

**4. What's NOT included?**

Don't ask: "What's the scope of this feature?"
Do ask: "You mentioned user management—is that just viewing and editing user info, or does it also include deleting users, banning users?" or "How complete should the first version be?"

### Step 3: Output a Requirement Summary

After clarification, produce a summary **in English** that the human can confirm:

```
## Requirement Confirmation

### What we're building
[One sentence description]

### Usage scenario
- Who: [role/user type]
- When: [scenario]
- Frequency: [how often]

### Normal flow
1. User [action] → sees [result]
2. User [action] → sees [result]
3. ...

### Error cases
- If [situation]: [how we handle it]
- If [situation]: [how we handle it]

### Not included (out of scope)
- [Explicitly excluded items]

### Does this match your expectations? Anything to change?
```

Wait for human confirmation before proceeding to planning.

## Rules

1. **Never assume.** If the human says "build a login feature", don't assume it includes registration, password reset, social login. Ask.

2. **Use examples, not abstractions.** Instead of "describe the business rules," say "For example, if a user wants to change their email, do we need to verify the old email? Or can they just change it directly?"

3. **Maximum 4 questions per round.** If you need more clarification, get answers first, then ask follow-ups. Don't dump 10 questions at once.

4. **Offer defaults for small decisions.** "For message timestamps, I'd default to showing 'a few minutes ago' style relative times—does that work for you?" This reduces the human's decision load.

5. **Flag hidden complexity early.** If a seemingly simple request actually involves significant work, say so upfront: "This sounds simple, but it actually involves real-time sync and conflict handling, which is a lot of work. Do you really need real-time in v1, or can we start with a version where users refresh to see the latest?"

6. **All communication in English.** Everything the human sees should be in clear, plain English.

## Anti-Patterns

- ❌ Receiving a vague request and immediately starting to write code
- ❌ Asking "What are your requirements?" (too vague, puts burden on human)
- ❌ Asking technical questions: "Do you need WebSocket or polling?", "REST or GraphQL?"
- ❌ Asking 8 questions at once
- ❌ Assuming "user management" means CRUD + auth + roles + permissions
- ❌ Skipping clarification because "the request seems clear enough"

## Integration with Other Skills

- **bounded-change-planning**: The requirement summary feeds directly into the change plan. The plan's scope must not exceed what was confirmed.
- **escalation-thresholds**: If clarification reveals the feature touches security, payments, or data—flag it during clarification, not after you've started coding.
- **plain-language-reporting**: This skill already outputs in English. The completion report should reference the original requirement summary.
- **product-qa**: The requirement summary becomes the test checklist for product QA.
