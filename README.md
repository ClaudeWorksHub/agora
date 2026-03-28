# Agora

> Multi-AI role debate plugin for Claude Code — generate high-quality proposals through structured multi-perspective discussion.

Agora assembles multiple AI roles to independently explore your codebase, then debate a proposal through structured rounds until consensus is reached. Each role brings a distinct perspective — the result is a proposal that has survived scrutiny from every angle.

## The Roles

| Role | Perspective | Arbitration Priority |
|------|-------------|---------------------|
| 🎭 **Momus** | Devil's advocate — challenges assumptions, finds security risks, stress-tests extreme scenarios | Security conflicts |
| 🔮 **Cassandra** | Detail insight — catches overlooked edge cases, boundary conditions, and subtle flaws | Edge cases & testing |
| 🦉 **Athena** | Strategic architecture — reviews global design, scalability, and long-term impact | Architecture conflicts |
| ⚒️ **Hephaestus** | Execution feasibility — evaluates implementability, resources, and schedule risks | Feasibility conflicts |

## How It Works

```
Phase 0  Q&A wizard (or parse CLI arguments) → user confirms
            ↓
Phase 1  Orchestrator explores codebase → generates initial proposal
            ↓
Phase 2  Independent Assessment — each role explores the code in parallel,
         produces a development plan from its own perspective
            ↓
Phase 3  Multi-round debate (up to N rounds)
         ┌─→ Each role reviews proposal (parallel, with code verification)
         │       ↓
         │   [Interactive mode] User provides additional input
         │       ↓
         │   Orchestrator responds point-by-point, updates proposal
         │       ↓
         │   Convergence check
         └── Loop until consensus or max-rounds
            ↓
Phase 4  Quality gate self-check → final proposal
```

Each role **must read the actual source code** before reviewing — no armchair critiques. Objections require evidence (file paths, line numbers, or quoted excerpts) and severity tags (`[CRITICAL]`, `[MAJOR]`, `[MINOR]`).

## Installation

### From marketplace

```bash
# 1. Add marketplace
/plugin marketplace add ClaudeWorksHub/claude-plugins

# 2. Install
/plugin install agora@claudeworkshub

# 3. Activate
/reload-plugins
```

Update to latest:

```bash
/plugin marketplace update claudeworkshub
/plugin install agora@claudeworkshub
/reload-plugins
```

### Local

Copy `agora/` to `~/.claude/plugins/`, or configure the path in `.claude/settings.local.json`.

## Quick Start

**Wizard mode** — guided setup for new users:

```
/agora:proposal
```

**Direct mode** — skip the wizard:

```
/agora:proposal Design a user authentication system
```

**With options:**

```
/agora:proposal --roles athena,momus --max-rounds 8 --interactive Design an API gateway
```

## Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `--roles` | Auto-selected by complexity | Comma-separated role names |
| `--exclude` | None | Exclude specific roles (mutually exclusive with `--roles`) |
| `--max-rounds` | 5-10 (by complexity) | Max discussion rounds (1-20) |
| `--interactive` | Off | Pause after each round for user input |
| `--resume <file>` | None | Resume an interrupted discussion |

### Complexity Auto-Selection

| Complexity | Criteria | Roles | Default Rounds |
|-----------|----------|-------|----------------|
| **Light** (default) | Bug fix, feature, refactor, analysis | 🎭 Momus + ⚒️ Hephaestus | 5 |
| **Medium** | Multi-module refactor, API design | 🔮 Cassandra + 🦉 Athena + 🎭 Momus | 8 |
| **Heavy** | System architecture, security-critical | All 4 roles | 10 |

## Output

Each run produces two files in `proposals/`:

| File | Content |
|------|---------|
| `proposal-<timestamp>.md` | Full discussion — every round, every objection, every response |
| `proposal-<timestamp>-final.md` | Final agreed proposal with discussion summary |

## Convergence

- **Rounds 1-3**: Deep review — each role must examine at least 3 dimensions
- **Round 4+**: Consensus when all active roles have no objection for 2 consecutive rounds
- **Hibernation**: Roles with no objections for 2+ rounds sleep to save cost; wake on proposal changes or after 3 rounds
- **Hard limit**: Best proposal at `--max-rounds` with unresolved issues listed
- **Cross-check**: Before convergence, raw status lines are verified against the discussion file

## Interactive Mode

```
/agora:proposal --interactive Design a caching system
```

After each round of role feedback, Agora pauses to let you:
- **Continue** — let the orchestrator respond
- **Add input** — inject constraints, priorities, or direction that roles must consider

Your directives take priority over role suggestions — you are the ultimate decision maker.

## Resume

```
/agora:proposal --resume proposals/proposal-20260328-160500.md
```

Picks up where you left off. Agora detects the interruption phase (assessment, debate, or finalize), verifies state consistency between the discussion file and state JSON, and resumes with all roles active for the first round back.

## Cost Control

- Auto-selects the lightest role configuration that fits your task
- Role hibernation reduces calls when consensus is near
- Context window managed: orchestrator keeps only last 2 rounds; agents launch fresh each round
- Cost estimate shown before you confirm: `~{N + N×max_rounds} opus calls (max)`

## Discussion Language

Discussions follow the language of your task description. English task → English discussion. Chinese task → Chinese discussion. Role names stay as their Greek mythology originals.

## License

MIT
