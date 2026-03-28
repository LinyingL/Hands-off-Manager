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
Hands-off-Manager/
├── README.md
├── CLAUDE.md                          ← 复制到项目根目录
├── guardrails/
│   └── guardrails.md                  ← 6 条强制约束
└── skills/
    ├── P0/                            ← 核心，立即启用
    │   ├── default-decision-policy/     何时自主决策、何时问人
    │   ├── bounded-change-planning/     限制每次改动的范围
    │   ├── evidence-based-change-report/ 完成报告 + 审计清单
    │   ├── escalation-thresholds/       升级决策纯查表
    │   └── requirement-clarification/   模糊想法 → 可执行需求
    ├── P1/                            ← 第二批启用
    │   ├── regression-watchlist/        高风险模块检查清单 *
    │   ├── surgical-refactor/           修 bug 与重构分离
    │   └── product-qa/                  从用户角度验收功能
    ├── communication/                 ← 沟通层
    │   ├── plain-language-reporting/    所有输出翻译为中文产品语言
    │   └── session-handoff/             对话间的上下文交接
    └── maintenance/                   ← 代码健康
        ├── code-health-standards/       好代码的定义 *
        └── code-debt-cleanup/           定期健康扫描与债务报告

* 标记的 skill 是项目模板，需要根据你的项目自定义后才能生效。
```

## 安装

### 前提条件

- 已安装 [Claude Code](https://docs.anthropic.com/en/docs/claude-code)（VS Code 扩展或 CLI 版本均可）
- 或使用其他支持 `.claude/skills/` 目录的 AI 编程工具

### 方式一：全量安装

```bash
git clone https://github.com/LinyingL/Hands-off-Manager.git
cd Hands-off-Manager

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

### 安装后验证

```bash
# 确认 skills 已安装（应该看到 12 个 skill 文件夹）
ls ~/.claude/skills/

# 确认 guardrails 已安装
head -5 ~/.claude/rules/guardrails.md

# 确认 frontmatter 生效（应该以 --- 开头）
head -3 ~/.claude/skills/default-decision-policy/SKILL.md
```

在 Claude Code 里输入 `/` 确认 skill 出现在补全列表里。

### 安装后自定义

以下两个 skill 包含项目模板，需要根据你的实际项目修改后才能生效：

1. **regression-watchlist** — 打开 `~/.claude/skills/regression-watchlist/SKILL.md`，将模板中的示例路径和测试命令替换为你项目的实际模块。

2. **code-health-standards** — 打开 `~/.claude/skills/code-health-standards/SKILL.md`，将项目结构模板替换为你项目的实际目录布局和命名约定。

## 12 个 Skills 详解

### P0：核心（5 个）

**`default-decision-policy`** — 自主决策规则。Agent 遇到未指定的选择（命名、文件组织、代码风格等）时自动触发。核心原则：跟着项目现有惯例走，没有就选更简单的。

**`bounded-change-planning`** — 改动范围控制。写代码之前自动触发，要求先产出改动计划。预算上限：3 新文件 / 8 改动文件 / 300 行 diff / 1 新依赖 / 2 新抽象。超出则停下重新规划。

**`evidence-based-change-report`** — 完成报告。每次声称完成时触发，按固定格式汇报：改了什么、为什么、跟计划有无偏差、验证证据、合规自查表。

**`escalation-thresholds`** — 升级查表。一张三级表：永远不问 / 必须问 / 有条件问。减少无意义打断，保证关键决策不被跳过。

**`requirement-clarification`** — 需求澄清。收到模糊需求时触发，用场景化问题（不是技术术语）帮你理清需求，产出中文需求确认文档后才开始写代码。

### P1：扩展（3 个）

**`regression-watchlist`** — 高风险模块检查清单。列出每个区域的不变量和必跑测试。**需要自定义。**

**`surgical-refactor`** — 修 bug 与重构分离。五条硬规则：分开提交、一次一个概念、先删后抽象、先证明行为再改结构、不顺手改。

**`product-qa`** — 产品验收。从用户角度逐步验证：正常流程、异常情况、范围确认。产出中文验收报告。

### Communication：沟通层（2 个）

**`plain-language-reporting`** — 中文产品语言。所有面向人的输出自动翻译，附术语翻译表（diff → 改动内容、migration → 数据库结构变更等），技术细节折叠到底部。

**`session-handoff`** — 会话交接。对话结束时自动写交接备忘：进度、决策、已知问题、约定。下次新对话自动读取。

### Maintenance：代码健康（2 个）

**`code-health-standards`** — 代码质量标准。定义模块边界、命名规则、反债务规则、健康指标。**需要自定义。**

**`code-debt-cleanup`** — 定期健康扫描。轻量扫描（一行报告）+ 全量扫描（中文健康报告：红黄绿三级 + 趋势追踪）。

## 6 条 Guardrails

写入 `.claude/rules/` 目录，始终生效。Skills 是指导，Guardrails 是硬约束：

| # | 名称 | 规则 | 触发时机 |
|---|---|---|---|
| G1 | no-diff-no-done | 没改动不能说完成 | 声称完成代码改动时 |
| G2 | no-evidence-no-success | 没证据不能说成功 | 声称代码正确时 |
| G3 | size-budget-check | 超预算强制停下 | 新文件 >3 / 改动文件 >8 / diff >300 行 |
| G4 | understand-before-modify | 改前必须理解 | 修改任何现有文件前 |
| G5 | verify-after-each-file | 改一个验一个 | 任务涉及 2+ 个文件时 |
| G6 | revert-before-retry | 两次失败回退重来 | 修复尝试连续失败时 |

### Guardrails 落地路径

1. **Phase 1 — 文档约束**：把 guardrails.md 放入 `.claude/rules/`，Agent 自我遵守，你审查合规表
2. **Phase 2 — 验证脚本**：guardrails.md 底部附了 `verify-diff.sh` 和 `check-budget.sh` 示例脚本
3. **Phase 3 — CI 集成**：将脚本接入 pre-commit hook 或 CI pipeline

## 与 Superpowers 的关系

本仓库不替代 Superpowers，而是补充它没覆盖的部分：

| 维度 | Superpowers 提供的 | 本仓库补充的 |
|---|---|---|
| 干活范围 | Writing Plans | bounded-change-planning + G3 |
| 完成标准 | Verification Before Completion | evidence-based-change-report + G1/G2 |
| 调试方法 | Systematic Debugging | surgical-refactor + G5/G6 |
| 沟通方式 | *(假设使用者是开发者)* | plain-language-reporting |
| 决策边界 | *(无)* | default-decision-policy + escalation-thresholds |
| 输入质量 | *(无)* | requirement-clarification |
| 跨会话 | *(无)* | session-handoff |
| 验收 | *(无)* | product-qa |

## 设计原则

- **Superpowers 原版能力全部保留**，不重复造轮子
- **Skills 是指导，Guardrails 是约束**，两者配合使用
- **所有面向人的输出都是中文**，技术细节折叠备查
- **每个 SKILL.md 带 YAML frontmatter**，支持自动发现和触发
- **活文档**，Agent 可以提议更新（需人类批准）

## 适合谁

有基础编程能力但不是每天看代码的人——产品经理、创始人、项目负责人。所有报告、决策升级、健康检查都用中文产品语言。但三个环节仍需懂代码的人定期介入：需求澄清的技术细节、关键技术选型、独立代码审查。

## License

[MIT](LICENSE)
