# Agent Workflow

## Skills

Available custom skills (in `.claude/skills/`):

### Core (P0)
- `/default-decision-policy` — 自主决策规则：什么不该问、什么必须问
- `/bounded-change-planning` — 改动范围控制：每次任务先列计划、超预算则停
- `/evidence-based-change-report` — 完成报告：有 diff 才能说完成、有证据才能说成功
- `/escalation-thresholds` — 升级查表：何时自主决策、何时必须问人
- `/requirement-clarification` — 需求澄清：把模糊想法变成可执行的中文需求文档

### Extended (P1)
- `/regression-watchlist` — 高风险模块检查清单
- `/surgical-refactor` — 修 bug 与重构严格分离
- `/product-qa` — 产品验收：从用户角度验证功能是否符合需求

### Communication
- `/plain-language-reporting` — 所有面向人的输出必须使用中文产品语言，技术细节折叠
- `/session-handoff` — 会话交接：每次对话结束自动写交接备忘，下次接着干

### Maintenance
- `/code-health-standards` — 代码质量标准：模块边界、命名规则、反债务规则
- `/code-debt-cleanup` — 定期健康扫描与债务报告

## Guardrails (Always Active)

These rules are in `.claude/rules/guardrails.md` and apply at all times:

1. **no-diff-no-done** — 没有实际文件改动，不能说"完成"
2. **no-evidence-no-success** — 没跑验证，不能说"成功"
3. **size-budget-check** — 新文件 >3 / 改动文件 >8 / diff >300 行，强制停下重新计划
4. **understand-before-modify** — 改文件前必须先说明该文件的职责和依赖
5. **verify-after-each-file** — 多文件改动时，每改一个就验证一次
6. **revert-before-retry** — 修两次还没好，回退到改动前的状态重新来

## Design System

Always read DESIGN.md before making any visual or UI decisions.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.
In QA mode, flag any code that doesn't match DESIGN.md.
