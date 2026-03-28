# Product QA

## Purpose

从产品和用户的角度验证功能是否正确，而不是从代码角度。这个 skill 回答的问题是："做出来的东西是不是用户想要的？"而不是"代码写得对不对？"代码层面的验证由 `evidence-based-change-report` 和 guardrails 负责。

## When This Skill Activates

- 一个功能开发完成后，在报告"完成"之前
- 人类说"测一下""看看对不对""验收一下"
- 一个 bug 修复完成后，验证用户体验是否恢复正常

## The QA Process

### Step 1: Pull the Requirement

找到这个功能的需求确认文档（由 `requirement-clarification` skill 产出的需求摘要）。如果没有需求文档，先问人类："这个功能做好之后，用户应该能 [做什么]？我按这个来验证，对吗？"

### Step 2: Walk Through the User Flow

按照需求里的"正常流程"逐步验证：

```
## 产品验收

### 正常流程验证

第 1 步：用户 [动作]
  预期：[应该看到什么]
  实际：✅ 符合预期 / ❌ 不符合（实际看到的是 [描述]）

第 2 步：用户 [动作]
  预期：[应该看到什么]
  实际：✅ 符合预期 / ❌ 不符合

...
```

### Step 3: Test Edge Cases

按照需求里的"异常情况"逐一验证：

```
### 异常情况验证

场景：用户输入了空白的用户名
  预期：提示"请输入用户名"
  实际：✅ / ❌

场景：网络中断时提交表单
  预期：提示"网络连接失败，请重试"
  实际：✅ / ❌

场景：用户快速连续点击提交按钮
  预期：只提交一次
  实际：✅ / ❌
```

### Step 4: Check "Not Included" Items

验证需求里明确说"不做"的东西确实没做（防止 Agent 过度开发）：

```
### 范围确认

- 本次不包含删除用户功能：✅ 确认没有实现
- 本次不包含批量操作：✅ 确认没有实现
```

### Step 5: Produce the QA Report

用中文输出一份验收报告：

```
## 产品验收报告

功能：[功能名称]
日期：[日期]
状态：✅ 通过 / ❌ 未通过 / ⚠️ 部分通过

### 结果摘要

正常流程：X/X 步通过
异常情况：X/X 项通过
范围确认：✅ 没有多做也没有少做

### 未通过的项目（如有）

1. [场景描述]
   预期：[什么]
   实际：[什么]
   严重程度：🔴 阻塞上线 / 🟡 可以先上但需要修 / 🟢 体验问题，不影响使用

### 总体评价

[一句话总结：这个功能能不能交付给用户用？如果不能，最关键的问题是什么？]
```

## What This Skill Does NOT Check

这些由其他 skill 和 guardrail 负责：

- 代码质量 → `code-health-standards`
- 测试覆盖率 → `evidence-based-change-report`
- 改动范围是否合理 → `bounded-change-planning`
- 是否有回归风险 → `regression-watchlist`

Product QA 只关心：**用户看到的、用户操作的、用户体验到的。**

## Common QA Scenarios to Always Check

无论需求有没有提到，以下场景默认都要验证：

### 表单类功能
- 空提交（什么都不填就提交）
- 超长输入（粘贴一大段文字）
- 特殊字符（引号、尖括号、emoji）
- 重复提交（连续快速点击）

### 列表类功能
- 空列表（没有数据时显示什么）
- 只有一条数据
- 很多数据（翻页/滚动是否正常）

### 状态类功能
- 刷新页面后状态是否保持
- 浏览器后退按钮的行为
- 同时打开两个标签页

### 错误类场景
- 网络断开
- 页面加载中的状态（loading）
- 操作失败后能否重试

## Rules

1. **All output in Chinese.** Per `plain-language-reporting`.
2. **Test as a user, not as a developer.** Don't check source code. Check what the user sees and does.
3. **Be specific about failures.** "不符合预期" is not enough. Say what you actually saw.
4. **Severity must be practical.** 🔴 means "users can't use this feature." 🟡 means "users can use it but will notice something off." 🟢 means "minor polish."
5. **Reference the original requirement.** Every test item traces back to the requirement summary. If something fails that wasn't in the requirement, note it as a "新发现的问题" rather than a test failure.

## Anti-Patterns

- ❌ Only checking the happy path
- ❌ Saying "功能正常" without listing what was tested
- ❌ Testing code paths instead of user actions
- ❌ Skipping QA because "all tests pass" (tests check code, QA checks product)
- ❌ Marking something as 🔴 when it's really 🟢 (inflating severity erodes trust)

## Integration with Other Skills

- **requirement-clarification**: The requirement summary is the test plan for QA. No requirement summary = no structured QA possible.
- **evidence-based-change-report**: Code-level evidence (tests, type check) appears in the change report. Product-level evidence (QA results) appears in the QA report. Both are needed for a complete "done."
- **session-handoff**: QA failures go into the handoff's "已知问题" section if not fixed in this session.
- **plain-language-reporting**: QA report is already in Chinese and uses product terms.
