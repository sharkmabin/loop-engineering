# Loop Engineering 中文说明

Loop Engineering 是一个面向 AI 编程代理的参考仓库。它关注的不是写更长的单次提示词，而是把“提示代理的人”替换成一套可重复运行的系统：这套系统会发现任务、维护状态、调用技能、执行验证，并在风险较高时交给人类确认。

这个仓库适合正在使用 Grok、Claude Code、Codex、Cursor 或其他 AI coding agent 的开发者，用来学习和落地“循环式代理工作流”。

## 这个项目解决什么问题

普通 AI 编程协作通常依赖人工不断输入下一条提示词。Loop Engineering 的目标是把这件事系统化：

- 用固定节奏自动发现和归类工作。
- 用 `STATE.md`、run log、budget 文件保存循环状态和成本约束。
- 用 skills、agents、MCP/connectors 固化项目知识和外部工具访问方式。
- 用 maker/checker 分工，让实现和验证分离。
- 用人工 gate 管住高风险动作，避免无人值守循环直接造成破坏。

简单说：你不再只设计 prompt，而是设计会持续提示、检查和交接的 loop。

## 核心概念

| 概念 | 含义 |
| --- | --- |
| Loop | 一个递归目标。代理按既定节奏反复运行，直到任务完成或升级给人类。 |
| L1 | 只报告，不自动修复。适合第一周验证价值和成本。 |
| L2 | 可辅助修复，但需要 verifier 和人工审查。 |
| L3 | 具备无人值守能力，但仍应有明确的预算、日志和人工 gate。 |
| State | 循环的外部记忆，通常是 `STATE.md` 或模式专用 state 文件。 |
| Budget | token、频率、运行范围和 kill switch 等成本控制。 |
| Verifier | 独立检查者，避免实现者自己给自己验收。 |

## 仓库里有什么

- `patterns/`：可复用的 loop 模式，例如 Daily Triage、PR Babysitter、CI Sweeper、Dependency Sweeper、Changelog Drafter。
- `starters/`：可以复制到项目中的 starter，覆盖 Grok、Claude Code、Codex 等工具。
- `skills/` 和 `templates/`：通用技能模板、预算检查、验证器和模式模板。
- `tools/loop-init/`：脚手架工具，用于把某个 loop 模式初始化到你的项目。
- `tools/loop-audit/`：Loop Readiness 评分工具，用于检查项目是否具备运行 loop 的基本条件。
- `tools/loop-cost/`：token 成本估算工具。
- `examples/`：按工具分类的运行示例，包括 GitHub Actions、Grok、Claude Code、Codex 和 MCP 配置。
- `docs/`：概念、原语矩阵、设计清单、安全、失败模式和多 loop 协调文档。
- `stories/`：真实使用故事，包括成功案例和失败复盘。

## 快速开始

如果只想在自己的项目中试一个最小 loop，可以直接使用已发布的 npm 工具：

```bash
# 1. 初始化一个 Daily Triage starter
npx @cobusgreyling/loop-init . --pattern daily-triage --tool grok

# 2. 估算 token 成本
npx @cobusgreyling/loop-cost --pattern daily-triage --level L1

# 3. 检查项目的 loop readiness
npx @cobusgreyling/loop-audit . --suggest
```

建议第一周从 L1 report-only 开始，只让 loop 生成报告和更新状态，不让它自动提交修复。

## 常见模式

| 模式 | 推荐节奏 | 第一周建议 | 适用场景 |
| --- | --- | --- | --- |
| Daily Triage | 1 天到 2 小时 | L1 报告 | 每天扫描项目状态、issue、待办和风险。 |
| PR Babysitter | 5 到 15 分钟 | L1 观察 | 盯 PR 状态、CI、review 反馈和需要跟进的事项。 |
| CI Sweeper | 5 到 15 分钟 | L2 谨慎修复 | 自动分析 CI 失败，必要时提出最小修复。 |
| Dependency Sweeper | 6 小时到 1 天 | L2 patch-only | 跟进依赖更新和小范围兼容性修复。 |
| Changelog Drafter | 1 天或 tag 后 | L1 草稿 | 根据变更生成 changelog 草稿。 |
| Post-Merge Cleanup | 1 天到 6 小时 | L1 离峰运行 | 合并后清理遗留事项、文档和状态。 |
| Issue Triage | 2 小时到 1 天 | L1 propose-only | 给 issue 分类、补充上下文和建议下一步。 |

不确定从哪个开始，可以先看 `docs/pattern-picker.md`。

## 从源码开发

这个仓库是 monorepo。常用命令如下：

```bash
# 构建所有工具
npm run build:tools

# 运行所有工具测试
npm run test:tools

# 校验 pattern registry
npm run validate:registry
```

也可以单独进入某个工具目录开发：

```bash
cd tools/loop-audit && npm ci && npm test && npm run build
cd tools/loop-init && npm ci && npm test && npm run build
cd tools/loop-cost && npm ci && npm test
```

## 安全建议

- 先跑 L1，只报告，不自动修复。
- 自动修复前必须有独立 verifier。
- 高风险动作需要人工 gate，例如合并 PR、修改安全配置、删除数据、扩大权限。
- 给每个 loop 设置 token 预算、运行频率、退出条件和 kill switch。
- 用 worktree 隔离无人值守或并行执行的代码变更。
- 定期阅读 loop 产物，避免理解债务快速累积。

## 进一步阅读

- 英文主说明：`README.md`
- Loop 设计清单：`docs/loop-design-checklist.md`
- 核心原语：`docs/primitives.md`
- 工具矩阵：`docs/primitives-matrix.md`
- 安全说明：`docs/safety.md`
- 运行和成本：`docs/operating-loops.md`
- 失败模式：`docs/failure-modes.md`

## 许可证

MIT
