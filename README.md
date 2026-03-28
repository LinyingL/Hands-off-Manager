# Agent Skills & Guardrails

AI 编程 Agent 的行为约束与工作流规范，适用于 Superpowers / Claude Code / Cursor 等 AI 编程工具。

## 解决什么问题

1. **Agent 小事也来问你** → 用决策策略和升级阈值，让它自主处理不重要的选择
2. **越改越多 bug** → 用改动前理解、逐文件验证、失败回退三重机制切断级联故障
3. **说"完成了"但其实没改** → 用 guardrail 强制要求有 diff 才能说完成、有证据才能说成功
4. **代码库越来越乱** → 用健康标准和定期债务扫描，在问题还小的时候发现它
5. **看不懂技术报告** → 所有汇报强制翻译为中文产品语言
6. **需求说不清楚** → 用场景化提问帮你把模糊想法变成可执行需求
7. **对话之间上下文丢失** → 自动写交接备忘，下次接着干
8. **功能做完不知道对不对** → 从用户角度验收，不懂代码也能判断

## 目录结构

```
skills/
  P0/                              ← 核心，立即启用
    default-decision-policy/         何时自主决策、何时问人
    bounded-change-planning/         限制每次改动的范围
    evidence-based-change-report/    完成报告的格式与审计清单
    escalation-thresholds/           升级决策的查表
    requirement-clarification/       把模糊想法变成可执行需求

  P1/                              ← 第二批启用
    regression-watchlist/            高风险模块的检查清单（需自定义）
    surgical-refactor/               修 bug 与重构分离
    product-qa/                      从用户角度验收功能

  communication/                   ← 沟通层
    plain-language-reporting/        所有输出翻译为中文产品语言
    session-handoff/                 对话间的上下文交接

  maintenance/                     ← 代码健康
    code-health-standards/           好代码的定义与模块边界
    code-debt-cleanup/               定期健康扫描与债务报告

guardrails/
  guardrails.md                    ← 6 条强制约束（写入系统提示）
    G1: no-diff-no-done              没改动不能说完成
    G2: no-evidence-no-success       没证据不能说成功
    G3: size-budget-check            超预算强制停下
    G4: understand-before-modify     改之前必须理解
    G5: verify-after-each-file       每改一个文件验证一次
    G6: revert-before-retry          两次失败后回退重来
```

## 怎么用

### 第一步：把 guardrails 写入系统提示

将 `guardrails/guardrails.md` 的内容放入 `.claude/rules/` 目录。这 6 条规则是硬约束，Agent 必须始终遵守。

### 第二步：启用 P0 skills

将 `skills/` 下的文件夹放入你的 skill 目录：

```bash
cp -r skills/*/* ~/.claude/skills/
mkdir -p ~/.claude/rules && cp rules/guardrails.md ~/.claude/rules/
```

### 第三步：自定义 regression-watchlist

`skills/P1/regression-watchlist/SKILL.md` 里有模板，需要替换为你项目的实际模块路径和测试命令。

### 第四步：把 CLAUDE.md 放入项目根目录

将仓库根目录的 CLAUDE.md 复制到你的项目根目录，让 Agent 知道有哪些 skill 和 guardrail 可用。

## 设计原则

- **Superpowers 原版能力全部保留**，不重复造轮子
- **Skills 是指导，Guardrails 是约束**，两者配合使用
- **所有面向人的输出都是中文**，技术细节折叠备查
- **允许自我报告违规**，审计目的是透明度不是惩罚
- **活文档**，Agent 可以提议更新（需人类批准）

## License

MIT
