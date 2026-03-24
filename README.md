# 华山论剑 / Huashan Lunjian

> Multi-AI role debate plugin for Claude Code — generate high-quality proposals through structured multi-perspective discussion.

多AI角色讨论插件 — 通过多角色结构化辩论，生成高质量 Proposal。

## Overview / 概述

华山论剑让多个AI角色从不同视角审视你的方案，通过多轮讨论打磨出更完善的 Proposal。

- 🌸 **林黛玉** — 细节洞察：发现被忽略的边界条件和潜在隐患
- 🌟 **诸葛亮** — 战略架构：从全局和长远角度审视设计
- 🐒 **孙悟空** — 魔鬼代言人：挑战假设，找出安全风险和极端场景
- 💰 **王熙凤** — 落地执行：关注可行性、资源和进度风险

讨论自动进行，直到所有角色达成共识（最多 N 轮），全过程写入文件供查阅。

## Installation / 安装

### From marketplace / 市场安装

```bash
claude plugins install huashanlunjian
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
/proposal --roles 诸葛亮,孙悟空 设计一个缓存系统
/proposal --roles zhugeliang,sunwukong Design a caching system
```

### Exclude roles / 排除角色

```
/proposal --exclude 王熙凤 设计一个通知系统
```

### Set max rounds / 设置最大轮数

```
/proposal --max-rounds 5 设计一个通知系统
```

### Combined / 组合使用

```
/proposal --roles 林黛玉,诸葛亮 --max-rounds 8 设计一个API网关
```

## Parameters / 参数

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--roles` | All 4 roles | Comma-separated role names (Chinese or pinyin) |
| `--exclude` | None | Exclude specific roles |
| `--max-rounds` | 10 | Max discussion rounds (1-20) |

### Role name mapping / 角色名映射

| Chinese | Pinyin | Perspective |
|---------|--------|-------------|
| 林黛玉 | lindaiyu | Detail & edge cases |
| 诸葛亮 | zhugeliang | Strategy & architecture |
| 孙悟空 | sunwukong | Devil's advocate |
| 王熙凤 | wangxifeng | Execution & feasibility |

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
         ┌─→ Each role reviews proposal sequentially
         │   各角色依次审视方案
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

- Rounds 1-3: Forced deep review — each role must raise at least 1 objection
- Round 4+: Converge when all roles + main Claude agree
- Hard limit: Stop at `--max-rounds` with best current proposal

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
- Role names and emojis remain in Chinese (they're character names from Chinese literature)

## License

MIT
