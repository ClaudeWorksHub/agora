# Agora

[English](README.md)

> Claude Code 多 AI 角色辩论插件——通过结构化多视角讨论，生成高质量提案。

Agora 召集多个 AI 角色独立探索你的代码库，然后通过多轮结构化辩论逐步精炼提案，直到全员达成共识。每个角色带来独特视角——最终产出是经过全方位审查的提案。

## 四个角色

| 角色 | 视角 | 仲裁优先权 |
|------|------|-----------|
| 🎭 **Momus（摩摩斯）** | 魔鬼代言人——挑战假设、发现安全风险、压力测试极端场景 | 安全冲突 |
| 🔮 **Cassandra（卡珊德拉）** | 细节洞察——捕捉被忽视的边界条件、逻辑漏洞和细微缺陷 | 边界与测试 |
| 🦉 **Athena（雅典娜）** | 战略架构——审视全局设计、可扩展性和长期影响 | 架构冲突 |
| ⚒️ **Hephaestus（赫菲斯托斯）** | 执行可行性——评估可实现性、资源和时间表风险 | 可行性冲突 |

## 工作原理

```
阶段 0  问答向导（或解析命令行参数）→ 用户确认
           ↓
阶段 1  Orchestrator 探索代码库 → 生成初始提案
           ↓
阶段 2  独立评估 — 各角色并行探索代码，从各自视角生成开发计划
           ↓
阶段 3  多轮辩论（最多 N 轮）
        ┌─→ 各角色审查提案（并行，须验证代码）
        │       ↓
        │   [交互模式] 用户补充意见
        │       ↓
        │   Orchestrator 逐条回应异议，更新提案
        │       ↓
        │   检查收敛条件
        └── 未收敛则继续
           ↓
阶段 4  质量门自检 → 输出最终提案
```

每个角色**必须阅读实际源码**后才能审查——不接受纸上谈兵。异议需要提供证据（文件路径、行号或引用片段）和严重程度标签（`[CRITICAL]`、`[MAJOR]`、`[MINOR]`）。

## 安装

### 从插件市场安装

```bash
# 1. 添加市场源
/plugin marketplace add ClaudeWorksHub/claude-plugins

# 2. 安装插件
/plugin install agora@claudeworkshub

# 3. 激活
/reload-plugins
```

更新到最新版：

```bash
/plugin marketplace update claudeworkshub
/plugin install agora@claudeworkshub
/reload-plugins
```

### 本地安装

将 `agora/` 目录复制到 `~/.claude/plugins/`，或在项目的 `.claude/settings.local.json` 中配置插件路径。

## 快速开始

**向导模式**——新用户推荐：

```
/agora:proposal
```

**直接模式**——跳过向导：

```
/agora:proposal 设计一个用户认证系统
```

**带参数：**

```
/agora:proposal --roles athena,momus --max-rounds 8 --interactive 设计 API 网关
```

## 参数

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `--roles` | 根据复杂度自动选择 | 逗号分隔的角色名 |
| `--exclude` | 无 | 排除指定角色（与 `--roles` 互斥） |
| `--max-rounds` | 5-10（按复杂度） | 最大讨论轮数（1-20） |
| `--interactive` | 关闭 | 每轮后暂停等待用户输入 |
| `--resume <file>` | 无 | 从中断的讨论文件恢复 |

### 复杂度自动选择

| 复杂度 | 适用场景 | 角色 | 默认轮数 |
|--------|---------|------|---------|
| **Light**（默认） | Bug 修复、功能开发、重构、分析 | 🎭 Momus + ⚒️ Hephaestus | 5 |
| **Medium** | 多模块重构、API 设计 | 🔮 Cassandra + 🦉 Athena + 🎭 Momus | 8 |
| **Heavy** | 系统架构、安全关键变更 | 全部 4 个角色 | 10 |

## 输出

每次运行在 `proposals/` 目录下生成两个文件：

| 文件 | 内容 |
|------|------|
| `proposal-<timestamp>.md` | 完整讨论过程——每轮、每个异议、每条回应 |
| `proposal-<timestamp>-final.md` | 最终提案 + 讨论摘要 |

## 收敛规则

- **第 1-3 轮**：鼓励深度审查，每个角色须从至少 3 个维度评估
- **第 4 轮起**：所有活跃角色连续 2 轮无异议即收敛（结构化规则，orchestrator 不可覆盖）
- **休眠机制**：连续 2 轮无异议的角色自动休眠节省 token，提案变更或 3 轮后自动唤醒
- **硬限制**：达到 `--max-rounds` 时输出当前最佳提案，列出未解决问题
- **交叉验证**：收敛前从讨论文件中提取原始 Status 行，与内存中的解析结果比对

## 交互模式

```
/agora:proposal --interactive 设计缓存系统
```

每轮角色反馈后，Agora 暂停让你选择：
- **继续** — 由 orchestrator 回应
- **补充意见** — 注入约束、优先级或方向指引，角色必须考虑

你的指令优先于角色建议——你是最终决策者。

## 中断恢复

```
/agora:proposal --resume proposals/proposal-20260328-160500.md
```

从中断点继续。Agora 检测中断阶段（评估、辩论或定稿），验证讨论文件与状态 JSON 的一致性，恢复后首轮所有角色强制 active。

## 成本控制

- 自动选择最轻量的角色配置
- 共识接近时角色休眠减少调用
- 上下文管理：orchestrator 仅保留最近 2 轮，agent 每轮全新启动
- 启动前展示成本预估：`~{N + N×max_rounds} opus calls (max)`

## 讨论语言

讨论跟随任务描述的语言。中文任务产生中文讨论，英文任务产生英文讨论。角色名保持希腊神话原名。

## 许可证

MIT
