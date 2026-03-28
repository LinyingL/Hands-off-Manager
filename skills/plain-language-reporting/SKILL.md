# Plain Language Reporting

## Purpose

The human you report to has basic code knowledge. They understand what files, functions, APIs, and databases are, but they do not read code daily. Every report, escalation, and status update must be written so that a non-developer product owner can read it and make decisions.

## This Skill Overrides All Other Report Formats

When any other skill (evidence-based-change-report, bounded-change-planning, escalation-thresholds, etc.) asks you to produce a report, you still follow that skill's structure—but you **translate every technical detail into plain language**. The technical details go in a collapsed section at the bottom for reference.

## Reporting Template

Every report to the human must follow this structure:

```
## 做了什么

[1-3 sentences in plain Chinese. Describe WHAT changed in terms of user-visible behavior or system behavior, not code structure.]

Example:
  ✅ "用户登录后，刷新页面不会再被踢出去了。之前是因为记住登录状态的机制在页面刷新时会失效。"
  ❌ "Modified auth.service.ts to add token refresh retry with exponential backoff in the interceptor chain."

## 改了哪里

[List files changed, but describe each in terms of its ROLE, not its code path.]

Example:
  ✅ "登录相关的后台逻辑（1个文件），以及它的测试（1个文件）"
  ❌ "src/services/auth.service.ts (+12 -3), src/services/auth.service.test.ts (+45 -0)"

## 有没有风险

[State risks in terms of what could go wrong for users or the product, not in code terms.]

Example:
  ✅ "这次改动只影响登录状态的刷新，不影响其他功能。风险很低。"
  ✅ "这个改动碰到了权限检查的核心逻辑，虽然测试都过了，但建议上线前手动测一下管理员和普通用户的权限是否正常。"
  ❌ "Risk: auth middleware is imported by 5 route handlers."

## 验证情况

[State what was tested in plain terms.]

Example:
  ✅ "自动测试全部通过（14个测试）。我还手动模拟了一次登录过期后刷新页面的情况，确认不会被踢出。"
  ❌ "npm test -- auth.service: 14 passed, 0 failed. tsc --noEmit clean."

---
<details>
<summary>技术细节（可跳过）</summary>

[Here put the full technical report as required by evidence-based-change-report, including exact file paths, diff stats, commands run, and raw output.]

</details>
```

## Escalation Template

When escalating a decision (per escalation-thresholds), translate it:

```
## 需要你决定

[One sentence describing the situation in plain language.]

### 背景
[Why this decision came up, in terms of what the product/feature needs.]

### 选项 A（推荐）：[plain description]
好处：[in user/product terms]
代价：[in user/product terms]

### 选项 B：[plain description]
好处：[in user/product terms]
代价：[in user/product terms]

### 如果选错了会怎样
[Describe impact in terms of: user experience, data safety, timeline, or maintenance cost. NOT in code terms.]

Example:
  ✅ "如果选 A，后续加新的用户角色会比较容易。如果选 B，现在快一天，但以后加角色要改的地方多。"
  ❌ "Option A uses a polymorphic role interface with O(1) lookup vs Option B's switch statement with O(n) maintenance cost."
```

## Size Budget Warning Template

When size-budget-check is triggered:

```
## 改动范围超出预期

这个任务比预想的要大。

已经完成的部分：
  - [plain description of what's done, in behavioral terms]

还没做的部分：
  - [plain description of what's left]

建议拆成两步：
  1. 先上线：[what can ship now and what it gives the user]
  2. 后续再做：[what comes next]

需要你确认这样拆可以。
```

## Translation Rules

### Always Translate These

| Technical term | Say instead |
|---|---|
| diff / changeset | 改动内容 |
| dependency | 需要额外安装的工具包 |
| migration | 数据库结构变更 |
| refactor | 重新整理代码（不影响功能） |
| type error / type check | 代码格式检查 |
| lint | 代码风格检查 |
| build | 打包编译 |
| endpoint | 接口 / API 接口 |
| schema | 数据结构 / 数据表结构 |
| abstract / abstraction | 公共模块 / 通用逻辑 |
| middleware | 中间处理层 |
| token | 登录凭证 |
| cache | 缓存（临时存储，加速用的） |
| ORM | 数据库操作工具 |
| CI/CD | 自动测试和部署流程 |
| mock | 模拟数据 |
| fixture | 测试用的假数据 |
| assertion | 测试验证点 |
| regression | 旧功能被新改动搞坏 |
| dead code | 已经没人用的旧代码 |
| barrel file / index.ts | 统一导出文件 |
| monorepo | 多个项目放在同一个代码库里 |
| generic / generics | 通用类型参数 |

### Keep These As-Is (The Human Knows Them)

- 文件名和路径（but add a parenthetical role description）
- API, 数据库, 前端, 后端
- 测试, bug, 功能
- 上线, 部署
- Git, 分支, 提交

### When Unsure

If a term might confuse the human, add a brief parenthetical:

> "这次改动涉及到 WebSocket（实时通信的连接方式）的重连逻辑。"

## Language

Default reporting language is **Chinese (简体中文)**. Use Chinese for all human-facing sections. Technical details section (in the collapsed block) can remain in English since it's for reference/debugging only.

## Anti-Patterns

- ❌ Dumping raw command output without explanation
- ❌ Using file paths as the primary way to describe what changed
- ❌ Saying "3 files modified, +45 -12 lines" without saying what those changes DO
- ❌ Describing risks in terms of code coupling ("imported by 5 modules") instead of user impact
- ❌ Asking the human to evaluate code-level tradeoffs ("should I use a Map or an Object?")
- ❌ Assuming the human will read the technical details section
