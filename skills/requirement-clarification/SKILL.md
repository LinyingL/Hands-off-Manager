# Requirement Clarification

## Purpose

将非技术人的模糊想法翻译成可执行的技术需求。在写任何代码之前，先确保双方对"要做什么"完全对齐。这个 skill 管的是 Agent 的输入质量——如果输入不清楚，后面所有 skill 都白搭。

## When This Skill Activates

When the human gives a task that involves building, changing, or fixing functionality. Specifically:

- "做一个 XX 功能"
- "我想要 XX"
- "XX 有问题，修一下"
- "能不能让 XX 变成 YY"
- Any product-level request that hasn't been broken down into technical specifics

Do NOT activate for:
- Purely technical tasks with clear specs ("把这个函数的返回类型从 string 改成 number")
- Questions or discussions ("XX 是怎么实现的？")
- Maintenance tasks that don't change behavior ("升级依赖""跑一下测试")

## The Clarification Process

### Step 1: Restate in Plain Chinese

Before asking any questions, restate what you understood in one sentence:

```
我理解你想要的是：[用一句话描述，不用任何技术术语]
```

If you can't restate it clearly, that's a signal you need more context. Say so.

### Step 2: Ask the Four Essential Questions

Every feature or change needs answers to these four questions. Ask them using **concrete scenarios**, not abstract concepts:

**1. 谁用？什么场景？**

Don't ask: "目标用户是谁？"
Do ask: "这个功能是所有用户都能用，还是只有管理员能用？" or "用户在什么情况下会用到这个？比如是每天都会用，还是偶尔才需要？"

**2. 正常流程是什么？**

Don't ask: "描述一下用户旅程"
Do ask: "帮我走一遍：用户打开页面后，第一步做什么？然后呢？最后他看到什么结果？"

**3. 出错了怎么办？**

Don't ask: "错误处理策略是什么？"
Do ask: "如果用户输入了错误的信息会怎样？比如邮箱格式不对，是直接提示他，还是不让他提交？" or "如果网络断了，用户正在做的东西会丢吗？"

**4. 有没有不做的？**

Don't ask: "这个功能的 scope 是什么？"
Do ask: "你说的用户管理，是只需要查看和编辑用户信息，还是也包括删除用户、封禁用户这些？" or "第一版先做到什么程度就够了？"

### Step 3: Output a Requirement Summary

After clarification, produce a summary **in Chinese** that the human can confirm:

```
## 需求确认

### 要做什么
[一句话描述]

### 使用场景
- 谁用：[角色]
- 什么时候用：[场景]
- 多频繁：[频率]

### 正常流程
1. 用户 [动作] → 看到 [结果]
2. 用户 [动作] → 看到 [结果]
3. ...

### 异常情况
- 如果 [情况]：[处理方式]
- 如果 [情况]：[处理方式]

### 不包含（本次不做）
- [明确排除的内容]

### 我的理解对吗？有没有要改的地方？
```

Wait for human confirmation before proceeding to planning.

## Rules

1. **Never assume.** If the human says "做一个登录功能"，don't assume it includes registration, password reset, social login. Ask.

2. **Use examples, not abstractions.** Instead of "请描述业务规则"，say "比如一个用户想改自己的邮箱，需要验证旧邮箱吗？还是直接改就行？"

3. **Maximum 4 questions per round.** If you need more clarification, get answers first, then ask follow-ups. Don't dump 10 questions at once.

4. **Offer defaults for small decisions.** "消息的时间显示，我默认用'几分钟前'这种相对时间，你觉得可以吗？" This reduces the human's decision load.

5. **Flag hidden complexity early.** If a seemingly simple request actually involves significant work, say so upfront: "这个功能听起来简单，但实际上涉及到实时同步和冲突处理，工作量比较大。你确定第一版就需要实时的吗？还是先做个刷新就能看到最新数据的版本？"

6. **All communication in Chinese.** Per `plain-language-reporting`, everything the human sees must be in plain Chinese.

## Anti-Patterns

- ❌ Receiving a vague request and immediately starting to write code
- ❌ Asking "你的需求是什么？" (too vague, puts burden on human)
- ❌ Asking technical questions: "需要 WebSocket 还是轮询？""用 REST 还是 GraphQL？"
- ❌ Asking 8 questions at once
- ❌ Assuming "用户管理" means CRUD + auth + roles + permissions
- ❌ Skipping clarification because "the request seems clear enough"

## Integration with Other Skills

- **bounded-change-planning**: The requirement summary feeds directly into the change plan. The plan's scope must not exceed what was confirmed.
- **escalation-thresholds**: If clarification reveals the feature touches security, payments, or data — flag it during clarification, not after you've started coding.
- **plain-language-reporting**: This skill already outputs in Chinese. The completion report should reference the original requirement summary.
- **product-qa**: The requirement summary becomes the test checklist for product QA.
