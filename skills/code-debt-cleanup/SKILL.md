# Code Debt Cleanup

## Purpose

Proactively find and report accumulating technical debt before it becomes a "shit mountain." This skill defines a periodic health scan that the Agent runs at natural breakpoints—not as a separate task, but woven into normal work.

## When This Skill Activates

Run a **lightweight scan** (Part 1 only) at the start of every task that touches a module the Agent hasn't worked on in the current session.

Run a **full scan** (Parts 1-3) when:
- A milestone or feature is completed
- The human explicitly asks for a health check
- The compliance checklist shows 3+ consecutive tasks with health warnings

## Part 1: Lightweight Scan (< 2 minutes)

When entering a module to work on a task, quickly check:

```bash
# 1. Dead exports: symbols exported but never imported elsewhere
# 2. Unused files: files not imported by anything
# 3. TODO count: how many TODOs are in this module
# 4. File sizes: any file over 300 lines
```

Report findings as a single line in the change report:

```
代码健康观察：这个模块里有 2 个没人用的导出函数，1 个文件超过 350 行，4 个 TODO 没有关联任务编号。
```

Do NOT fix these during the current task. Note them only. Per `surgical-refactor`, cleanup is a separate task.

## Part 2: Full Scan — Structural Health

Check the overall project structure against `code-health-standards`:

### 2.1 Dependency Direction

Verify that imports flow downward (api → services → repositories). Flag any upward imports:

```
⚠️ 依赖方向问题：
  - src/repositories/user.repo.ts 引入了 src/services/auth.service.ts
    这是底层模块引用了上层模块，会导致循环依赖风险。
```

### 2.2 Duplication Detection

Look for patterns that suggest copy-paste:

- Two files with suspiciously similar function names
- Identical error handling blocks across multiple files
- Same data transformation logic in different services

```
⚠️ 疑似重复逻辑：
  - src/services/user.service.ts 和 src/services/order.service.ts
    都有一段差不多的"验证权限后查数据"逻辑（约 20 行）。
    建议提取为共享的权限检查工具函数。
```

### 2.3 Orphaned Patterns

Check if the project has multiple ways of doing the same thing:

```
⚠️ 存在两种不同的做法：
  - HTTP 请求：src/utils/api-client.ts 和 src/utils/fetch-wrapper.ts
    两个文件都在做 HTTP 请求封装，12 个文件用前者，3 个文件用后者。
    建议统一到 api-client.ts 并删除 fetch-wrapper.ts。
```

### 2.4 Growing Files

Flag files that are approaching or exceeding healthy limits:

```
⚠️ 文件体积预警：
  - src/services/order.service.ts: 480 行（建议上限 300 行）
    最近 3 个任务都往这个文件加了逻辑。建议拆分为：
    - order-creation.service.ts（下单流程）
    - order-fulfillment.service.ts（履约流程）
```

## Part 3: Full Scan — Debt Inventory

Compile a debt summary in plain Chinese:

```
## 代码健康报告

上次扫描：[日期]
扫描范围：[全项目 / 某模块]

### 需要关注的问题（按优先级）

1. 🔴 高优先：order.service.ts 已经 480 行，再加功能就很难维护了
   建议：下个迭代安排一次拆分，预计 2-3 小时工作量

2. 🟡 中优先：项目里有两套 HTTP 请求封装，容易搞混
   建议：统一到一套，预计 1 小时工作量

3. 🟢 低优先：12 个 TODO 没有关联任务编号
   建议：下次碰到这些文件时顺手补上编号

### 好的趋势

- 最近 5 个任务都在预算内完成，改动范围控制得不错
- 没有新增 any 类型
- 测试覆盖率在这个模块里保持稳定

### 数字概览

| 指标 | 当前值 | 上次 | 趋势 |
|---|---|---|---|
| 总文件数 | 84 | 78 | ↑ 正常增长 |
| 超过 300 行的文件 | 3 | 2 | ⚠️ 关注 |
| 没人引用的导出 | 7 | 5 | ⚠️ 在积累 |
| 无编号的 TODO | 12 | 8 | ⚠️ 在积累 |
| 循环依赖 | 0 | 0 | ✅ 健康 |
```

## How This Connects to Your Workflow

The lightweight scan (Part 1) happens silently—you'll just see one line in the change report. No extra work for you.

The full scan (Parts 2-3) produces a Chinese-language health report. You read it like a physical checkup report: red items need scheduling, yellow items need awareness, green items can wait. The "趋势" column tells you whether things are getting better or worse.

**You never need to understand the code to act on this report.** Each item tells you: what the problem is in plain terms, what to do about it, and how much effort it takes. You can hand it to the Agent as a task ("处理那个高优先的 order.service 拆分") or to a developer for review.

## When to Run the Full Scan

Suggested cadence (adjust based on your project pace):

- **After every major feature ships**: Good checkpoint to see what debt the feature introduced
- **Every 2 weeks if in active development**: Catches slow accumulation
- **Before a release**: Ensures no surprise structural issues
- **When you feel something is "off"**: "最近改东西总是碰到各种问题" → run a full scan to see if there's structural rot

## Anti-Patterns

- ❌ Running a full scan after every small task (too noisy)
- ❌ Fixing debt items during feature work (per `surgical-refactor`, these are separate tasks)
- ❌ Ignoring the lightweight scan notes across 10+ tasks (debt is accumulating silently)
- ❌ Treating all debt as urgent (some debt is fine to live with)
