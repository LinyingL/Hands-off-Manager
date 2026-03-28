# Session Handoff

## Purpose

确保对话之间的上下文不丢失。每次对话结束前写一份交接备忘，下次对话开始时读取它，接着上次的进度继续干。

## When This Skill Activates

- **写交接**：每次对话即将结束时（人类说"先到这"、"下次继续"、"今天就到这"，或者你感知到对话在收尾）
- **读交接**：每次新对话开始，且项目目录下存在 `.claude/handoff.md` 文件时

## Handoff File Location

```
.claude/handoff.md
```

This file is overwritten each session (not appended). It represents the **current state**, not a history log.

## Writing the Handoff

At the end of a session, create or update `.claude/handoff.md` with this structure:

```markdown
# 交接备忘

更新时间：[日期时间]

## 当前进度

### 已完成
- [用产品语言描述完成了什么，不是代码层面的]
  - 涉及文件：[文件列表，供下次定位用]

### 进行中（未完成）
- [正在做什么，做到哪一步了]
  - 当前状态：[具体到什么程度]
  - 下一步：[接下来该做什么]
  - 阻塞点：[如果有的话]

### 待办（还没开始）
- [计划做但还没开始的任务]

## 关键决策记录

| 决策 | 选了什么 | 为什么 | 谁决定的 |
|---|---|---|---|
| [决策描述] | [选项] | [原因] | 人类 / Agent |

## 已知问题

- [目前已知但还没修的问题]
- [可能有风险的地方]

## 项目约定

[在这次对话中确立的约定，下次必须继续遵守]

- [例如：API 返回格式统一用 { data, error, meta }]
- [例如：所有日期存 UTC，展示时转本地时区]

## 文件地图

[本次对话中接触过的关键文件及其职责，帮助下次快速定位]

- `src/services/auth.service.ts` — 登录、登出、token 刷新
- `src/middleware/auth.middleware.ts` — 路由级别的权限检查
```

## Reading the Handoff

At the start of a new session:

1. Check if `.claude/handoff.md` exists.
2. If yes, read it **before doing anything else**.
3. Briefly acknowledge to the human: "我读了上次的交接记录。上次做到了 [简述]，还有 [简述] 没做完。要继续吗？还是有新的任务？"
4. If the human wants to continue, pick up from the "进行中" section.
5. If the human has a new task, still keep the handoff context in mind (especially "项目约定" and "已知问题").

## Rules

1. **Always write in Chinese.** This file is for the human to read too, not just for the Agent.

2. **Product language, not code language.** "已完成" section should say "用户登录后刷新页面不再被踢出" not "Added token refresh retry in auth.service.ts."

3. **File references go in sub-bullets.** The human doesn't need to see file paths in the main description, but the next session's Agent does need them for navigation. Put them in indented sub-bullets.

4. **Decision records are critical.** The single most common source of wasted work across sessions is re-debating a decision that was already made. Record every significant decision, who made it, and why.

5. **Overwrite, don't append.** Each session produces a fresh handoff. Old handoffs are not valuable — git history preserves them if needed.

6. **Don't ask permission to write the handoff.** Just write it at the end of the session. It's a background maintenance task, not something the human needs to approve.

7. **Keep it under 80 lines.** The handoff should be scannable in under 2 minutes. If the project has too much context to fit, focus on the "进行中" and "关键决策" sections — those are what the next session needs most.

## When the Human Doesn't Explicitly End the Session

If the conversation seems to be ending naturally (long pause, task completed, human says "好的" or "谢谢"), write the handoff proactively. Better to write one that isn't needed than to lose context.

## Anti-Patterns

- ❌ Not writing a handoff because "we finished everything" (there's always context worth preserving — conventions, decisions, known issues)
- ❌ Writing the handoff in English or with technical jargon
- ❌ Appending to the previous handoff (creates a growing, unreadable log)
- ❌ Including raw code diffs in the handoff (that's what git is for)
- ❌ Asking the human "should I write a handoff?" (just do it)

## Integration with Other Skills

- **requirement-clarification**: Confirmed requirements go into the "项目约定" section.
- **evidence-based-change-report**: The completion report for the session informs the "已完成" section.
- **bounded-change-planning**: Unfinished plan items go into "进行中" with their current state.
- **escalation-thresholds**: Pending decisions that were escalated but not yet answered go into "阻塞点."
