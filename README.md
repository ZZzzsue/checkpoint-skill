<div align="center">

# 🔖 checkpoint-skill

**跨 Agent 任务进度保存与恢复 · Cross-agent task context handoff**

*在 Hermes、Claude Code、Codex 中无缝切换，任务进度永不丢失*  
*Save progress, switch agents freely, resume anywhere*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Hermes](https://img.shields.io/badge/Hermes-✓-gold)](https://github.com/NousResearch/hermes-agent)
[![Claude Code](https://img.shields.io/badge/Claude_Code-✓-orange)](https://docs.anthropic.com/claude-code)
[![Codex](https://img.shields.io/badge/Codex-✓-green)](https://github.com/openai/codex)

</div>

---

## 🧩 这是什么 / What is this?

上下文太长、切换模型、跨 Agent 交接任务时，你会丢失所有状态。  
这个 skill 把当前任务蒸馏成一个精简 markdown 文件，任何 Agent 读入后 30 秒内接续。

When your context window gets too long, you switch models, or hand off between agents — you lose all state.  
This skill distills your current task into a compact markdown file that any agent can load and resume from in under 30 seconds.

```
当前 session (上下文过长)          新 session / 换了 Agent
mdw-myproject  →  保存进度    →    mdr-myproject  →  无缝继续
```

---

## ⚡ 安装 / Installation

### Hermes

```bash
hermes skills install https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md
```

安装后 `/checkpoint` 自动注册为 slash 命令，无需额外配置。  
After install, `/checkpoint` is auto-registered as a slash command — no extra setup needed.

### Claude Code

```bash
curl -sL https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/claude-setup.md >> ~/.claude/CLAUDE.md
```

### Codex

```bash
mkdir -p ~/.codex/skills/checkpoint && \
curl -sL https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md \
  > ~/.codex/skills/checkpoint/SKILL.md
```

安装后用 `/use checkpoint` 加载。  
After install, use `/use checkpoint` to load the skill.

---

## 🚀 使用方法 / Usage

### 第一步：加载 skill / Step 1 — Load the skill

| Agent | 命令 / Command |
|-------|--------------|
| Hermes | `/checkpoint` |
| Codex | `/use checkpoint` |
| Claude Code | 安装后自动生效 / Active after install |

### 第二步：使用命令 / Step 2 — Use commands

> 命令以独立消息发送（整条消息就是命令本身）  
> Type commands as a standalone message — the entire message is the command.

| 命令 / Command | 说明 / Action |
|----------------|--------------|
| `mdw-<name>` | 保存当前任务进度 / Save current task progress |
| `mdr-<name>` | 加载检查点并恢复 / Load checkpoint and resume |
| `mdl` | 列出所有检查点 / List all checkpoints |

**别名也支持 / Aliases also work:**

| 中文 / Chinese | 英文 / English | 说明 / Action |
|----------------|----------------|--------------|
| `保存进度 <name>` | `checkpoint write <name>` | 写入 / Write |
| `加载进度 <name>` | `checkpoint read <name>` | 读取 / Read |
| `列出进度` | `checkpoint list` | 列出 / List |

**不写名字 / No name?** — `mdw-` 不带名称时自动以时间命名：`20260617-0012.md`

---

## 📄 检查点格式 / Checkpoint Format

文件保存在 `~/.hermes/checkpoints/<name>.md`  
Files are saved to `~/.hermes/checkpoints/<name>.md`

```markdown
# myproject · checkpoint
_saved: 2026-06-17 00:12_

## Goal
原始任务一句话 / Original task in one sentence

## Status
当前阶段 / Current phase + last completed action

## Findings
- 高价值发现，附证据 / High-signal finding with evidence

## Done
- [x] 已完成步骤 / Completed step

## Next Steps
1. 立即可执行的下一步（恢复点）/ Immediately actionable — resume point

## Context
- paths: 相关路径 / relevant paths
- tools: 工具和关键参数 / tools and key flags
- gotchas: 踩过的坑 / things that burned time
- env: 环境变量名（不含值）/ key NAMES only, never values

## Blockers
卡住的问题（无则省略此节）/ Omit if none
```

### 压缩规则 / Compression Rules

| 区域 / Section | 规则 / Rule |
|----------------|------------|
| **HEAD** — Goal + Status | 原文保留 / Preserve verbatim |
| **TAIL** — Next Steps + Context + Blockers | 原文保留 / Preserve verbatim |
| **MIDDLE** — Findings + Done | 最大压缩，上限 5+8 条 / Max compress, limit 5+8 items |
| **总计 / Total** | 不超过 80 行 / Under 80 lines |

---

## 🔄 跨 Agent 工作流 / Cross-Agent Workflow

```
Hermes session (上下文过长 / long context)
  └─ mdw-feature-auth          ← 保存检查点 / save checkpoint

Claude Code session
  └─ mdr-feature-auth          ← 加载并继续 / load and resume

Codex session
  └─ mdr-feature-auth          ← 同一个文件，同一个恢复点 / same file, same resume point
```

检查点是纯 markdown — 任何 Agent 都能读取其他 Agent 写入的内容。  
Checkpoints are plain markdown — any agent can read what any other agent wrote.

---

## ⚠️ 注意事项 / Pitfalls

- 🔒 **不写密钥** — 只记变量名，不记实际值 / No secrets — key names only, never values
- 📵 **不写原始输出** — 提取信号，不写完整响应 / No raw output — extract the signal
- ⏰ **过期提醒** — 超过 24 小时加载时会提示 / Stale warning after 24h
- 🗜️ **压缩标记** — 上下文被压缩时 Status 中注明 / Compacted context is noted in Status
- ✅ **内部一致** — Gotchas 说"X 不可能"，Next Steps 就不能写"做 X" / No internal contradictions

---

## 📁 文件说明 / Files

| 文件 / File | 说明 / Purpose |
|-------------|---------------|
| `SKILL.md` | Skill 主文件，安装到 Hermes 或 Codex / Main skill file |
| `README.md` | 本文件 / This file |
| `claude-setup.md` | 追加到 `~/.claude/CLAUDE.md` 的片段 / Snippet for Claude Code |

---

<div align="center">
<sub>
为 <a href="https://github.com/NousResearch/hermes-agent">Hermes Agent</a> 构建 · 兼容 Claude Code 和 Codex<br>
Built for <a href="https://github.com/NousResearch/hermes-agent">Hermes Agent</a> · Compatible with Claude Code and Codex
</sub>
</div>
