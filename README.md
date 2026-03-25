# Symposion

> Multi-AI role debate plugin for Claude Code — generate high-quality proposals through structured multi-perspective discussion.

## Overview / 概述

华山论剑让多个AI角色从不同视角审视你的方案，通过多轮讨论打磨出更完善的 Proposal。

- 🎭 **Momus** — Devil's advocate: challenges assumptions, finds security risks and extreme scenarios
- 🔮 **Cassandra** — Detail insight: catches overlooked edge cases, boundary conditions and subtle flaws
- 🦉 **Athena** — Strategic architecture: reviews global design, scalability and long-term impact
- ⚒️ **Hephaestus** — Execution feasibility: evaluates implementability, resources and schedule risks

> *"Symposion 召集了四种不可替代的视角——批评之眼、先知之声、战略之断、匠人之手——各司其职，只问能力，不问出身。"*

讨论自动进行，直到所有角色达成共识（最多 N 轮），全过程写入文件供查阅。

## Installation / 安装

### From marketplace / 市场安装

In Claude Code, run the following commands:

在 Claude Code 中依次执行：

```bash
# 1. Add marketplace / 添加市场
/plugin marketplace add ClaudeWorksHub/claude-plugins

# 2. Install plugin / 安装插件
/plugin install huashanlunjian@claudeworkshub

# 3. Activate / 激活
/reload-plugins
```

To update to the latest version / 更新到最新版本：

```bash
/plugin marketplace update claudeworkshub
/plugin install huashanlunjian@claudeworkshub
/reload-plugins
```

### Local development / 本地安装

Copy the `huashanlunjian/` directory to `~/.claude/plugins/`:

```bash
cp -r huashanlunjian ~/.claude/plugins/
```

Or configure the plugin path in your project's `.claude/settings.local.json`.

## Usage / 使用方法

### Basic / 基本用法

```
/proposal 设计一个用户登录系统
```

### Select roles / 选择角色

```
/proposal --roles athena,momus 设计一个缓存系统
/proposal --roles athena,momus Design a caching system
```

### Exclude roles / 排除角色

```
/proposal --exclude hephaestus 设计一个通知系统
```

### Set max rounds / 设置最大轮数

```
/proposal --max-rounds 5 设计一个通知系统
```

### Combined / 组合使用

```
/proposal --roles cassandra,athena --max-rounds 8 设计一个API网关
```

## Parameters / 参数

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--roles` | All 4 roles | Comma-separated role names (or legacy aliases) |
| `--exclude` | None | Exclude specific roles |
| `--max-rounds` | 10 | Max discussion rounds (1-20) |

### Role names / 角色名

| Name | Legacy alias | Perspective |
|------|-------------|-------------|
| `momus` | `sunwukong` | Devil's advocate |
| `cassandra` | `lindaiyu` | Detail & edge cases |
| `athena` | `zhugeliang` | Strategy & architecture |
| `hephaestus` | `wangxifeng` | Execution & feasibility |

## Output / 输出文件

Each run generates two files in your project's `proposals/` directory:

每次运行在项目的 `proposals/` 目录下生成两个文件：

| File | Content |
|------|---------|
| `proposal-<timestamp>.md` | Full discussion process / 完整讨论过程 |
| `proposal-<timestamp>-final.md` | Final proposal / 最终方案 |

## How it works / 工作原理

```
Phase 0: Parse arguments, confirm with user
         解析参数，确认配置
              ↓
Phase 1: Generate initial proposal
         生成初始方案
              ↓
Phase 2: Multi-round discussion (max N rounds)
         多轮讨论（最多 N 轮）
         ┌─→ Each role reviews proposal (parallel recommended, serial fallback)
         │   各角色审视方案（推荐并行，串行兜底）
         │        ↓
         │   Main Claude responds to objections
         │   主 Claude 逐一回应异议
         │        ↓
         │   Check convergence
         │   检查是否收敛
         └── Continue if not converged
              ↓
Phase 3: Final proposal with self-review
         精修并输出最终 Proposal
```

### Convergence rules / 收敛规则

- Rounds 1-3: Deep review encouraged — each role reviews from at least 3 dimensions (can be no-objection with dimension list)
- Round 4+: Converge when all active roles have no objections for 2 consecutive rounds (structural rule, main Claude cannot override)
- Grep cross-check: Before convergence, verify status from file matches parsed results
- Hard limit: Stop at `--max-rounds` with best current proposal + unresolved issues listed

## Git handling / Git 处理建议

The plugin does not modify your `.gitignore`. Suggestions:

- Discussion files can be large — consider adding `proposals/proposal-*.md` to `.gitignore`
- Final proposals (`*-final.md`) are worth committing as design decision documents
- Or keep everything for full audit trail

## Extensibility / 可扩展性

v1 ships with 4 built-in roles. The format is open for future custom roles:

- Create a role file following the same format in your project's `.claude/agents/`
- Use `--roles` to include it in discussions
- The role must follow the same status/summary protocol

## Discussion language / 讨论语言

Discussions follow the language of your task description:
- Chinese task → Chinese discussion
- English task → English discussion
- Role names are Greek mythology characters (Momus/Cassandra/Athena/Hephaestus); legacy Chinese aliases (sunwukong/lindaiyu/zhugeliang/wangxifeng) remain supported

## License

MIT
