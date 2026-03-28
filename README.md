# Hands-off Manager

AI 编程 Agent 的行为约束与工作流规范，适用于 Superpowers / Claude Code / Cursor 等 AI 编程工具。

## 解决什么问题

1. **Agent 小事也来问你** → 决策策略 + 升级查表，让它自主处理不重要的选择
2. **越改越多 bug** → 改动前理解、逐文件验证、失败回退三重机制切断级联故障
3. **说"完成了"但其实没改** → 有 diff 才能说完成、有证据才能说成功
4. **代码库越来越乱** → 健康标准 + 定期债务扫描，在问题还小的时候发现它
5. **看不懂技术报告** → 所有汇报强制翻译为中文产品语言
6. **需求说不清楚** → 场景化提问，把模糊想法变成可执行需求
7. **对话之间上下文丢失** → 自动写交接备忘，下次接着干
8. **功能做完不知道对不对** → 从用户角度验收，不懂代码也能判断

## 目录结构

```
skills/
  P0/                              ← 核心，立即启用
    default-decision-policy/         何时自主决策、何时问人
    bounded-change-planning/         限制每次改动的范围
    evidence-based-change-report/    完成报告 + 审计清单
    escalation-thresholds/           升级决策纯查表
    requirement-clarification/       模糊想法 → 可执行需求

  P1/                              ← 第二批启用
    regression-watchlist/            高风险模块检查清单（需自定义）
    surgical-refactor/               修 bug 与重构分离
    product-qa/                      从用户角度验收功能

  communication/                   ← 沟通层
    plain-language-reporting/        所有输出翻译为中文产品语言
    session-handoff/                 对话间的上下文交接

  maintenance/                     ← 代码健康
    code-health-standards/           好代码的定义（需自定义）
    code-debt-cleanup/               定期健康扫描与债务报告

guardrails/
  guardrails.md                    ← 6 条强制约束
```

## 安装

### 方式一：一键复制

```bash
# 复制所有 skills
cp -r skills/P0/* ~/.claude/skills/
cp -r skills/P1/* ~/.claude/skills/
cp -r skills/communication/* ~/.claude/skills/
cp -r skills/maintenance/* ~/.claude/skills/

# 复制 guardrails
mkdir -p ~/.claude/rules
cp guardrails/guardrails.md ~/.claude/rules/

# 复制 CLAUDE.md 到你的项目根目录
cp CLAUDE.md /your/project/root/CLAUDE.md
```

### 方式二：只装核心

只安装 P0 + communication（解决最痛的 5 个问题）：

```bash
cp -r skills/P0/* ~/.claude/skills/
cp -r skills/communication/* ~/.claude/skills/
mkdir -p ~/.claude/rules
cp guardrails/guardrails.md ~/.claude/rules/
cp CLAUDE.md /your/project/root/CLAUDE.md
```

### 安装后

1. 打开 Claude Code，输入 `/` 确认 skill 出现在补全列表里
2. 自定义 `regression-watchlist` 和 `code-health-standards` 里的项目路径（如启用）
3. 开始工作

## 6 条 Guardrails

写入 `.claude/rules/` 目录，始终生效：

| # | 规则 | 含义 |
|---|---|---|
| G1 | no-diff-no-done | 没改动不能说完成 |
| G2 | no-evidence-no-success | 没证据不能说成功 |
| G3 | size-budget-check | 超预算强制停下重新计划 |
| G4 | understand-before-modify | 改之前必须理解文件职责和依赖 |
| G5 | verify-after-each-file | 每改一个文件验证一次 |
| G6 | revert-before-retry | 两次失败后回退重来 |

## 设计原则

- **Superpowers 原版能力全部保留**，不重复造轮子
- **Skills 是指导，Guardrails 是约束**，两者配合使用
- **所有面向人的输出都是中文**，技术细节折叠备查
- **每个 SKILL.md 带 YAML frontmatter**，支持自动发现和触发
- **活文档**，Agent 可以提议更新（需人类批准）

## 适合谁

有基础编程能力但不是每天看代码的人。所有报告、决策升级、健康检查都用中文产品语言。但三个环节仍需懂代码的人定期介入：需求澄清的技术细节、关键技术选型、独立代码审查。

## License

MIT
