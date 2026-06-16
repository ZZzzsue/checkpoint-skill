<div align="center">

# 🔖 checkpoint-skill

**Cross-agent task context handoff — save progress, resume anywhere.**

*Works identically in Hermes, Claude Code, and Codex.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Works with Hermes](https://img.shields.io/badge/Hermes-✓-gold)](https://github.com/NousResearch/hermes-agent)
[![Works with Claude Code](https://img.shields.io/badge/Claude_Code-✓-orange)](https://docs.anthropic.com/claude-code)
[![Works with Codex](https://img.shields.io/badge/Codex-✓-green)](https://github.com/openai/codex)

</div>

---

## 🧩 What is this?

When your context window gets too long, you switch models, or you hand off a task between agents — you lose state. This skill distills your current task into a compact markdown file that any agent can load and resume from in under 30 seconds.

```
User → /checkpoint → mdw-myproject   →  ~/.hermes/checkpoints/myproject.md
                                              ↓
New session / different agent → mdr-myproject → resumes exactly where you left off
```

---

## ⚡ Quick Install

### Hermes
```bash
hermes skills install https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md
```
After install, `/checkpoint` is auto-registered as a slash command.

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
After install, use `/use checkpoint` to load.

---

## 🚀 Usage

### Step 1 — Load the skill

| Agent | Command |
|-------|---------|
| Hermes | `/checkpoint` |
| Codex | `/use checkpoint` |
| Claude Code | Automatically active after install |

### Step 2 — Use commands

> All commands are plain text — type them as a standalone message.

| Command | Action |
|---------|--------|
| `mdw-<name>` | Save current task progress to a checkpoint |
| `mdr-<name>` | Load a checkpoint and offer to resume |
| `mdl` | List all saved checkpoints |

**Aliases also work:**

| Chinese | English | Action |
|---------|---------|--------|
| `保存进度 <name>` | `checkpoint write <name>` | Write |
| `加载进度 <name>` | `checkpoint read <name>` | Read |
| `列出进度` | `checkpoint list` | List |

**No name? No problem** — `mdw-` without a name uses a timestamp: `20260617-0012.md`

---

## 📄 Checkpoint Format

Checkpoints are saved to `~/.hermes/checkpoints/<name>.md`.

```markdown
# myproject · checkpoint
_saved: 2026-06-17 00:12_

## Goal
[Original task in one sentence]

## Status
[Current phase / last completed action]

## Findings
- [High-signal discovery with evidence]

## Done
- [x] [Completed step]

## Next Steps
1. [Immediately actionable — this is the resume point]

## Context
- paths: ...
- tools: ...
- gotchas: ...
- env: [key NAMES only, never values]

## Blockers
[Omit if none]
```

### Compression Rules

| Section | Rule |
|---------|------|
| **HEAD** — Goal + Status | Preserve verbatim |
| **TAIL** — Next Steps + Context + Blockers | Preserve verbatim |
| **MIDDLE** — Findings + Done | Compress aggressively (max 5 + 8 items) |
| **Total** | Under 80 lines |

---

## 🔄 Cross-Agent Workflow

```
Hermes session (long context)
  └─ mdw-feature-auth          ← save checkpoint

Claude Code session
  └─ mdr-feature-auth          ← load checkpoint, continue

Codex session
  └─ mdr-feature-auth          ← same file, same resume point
```

The checkpoint file is plain markdown — any agent can read what any other agent wrote.

---

## 📋 Pitfalls

- **No secrets** — key names only, never actual values
- **No raw output** — extract the signal, not the full response
- **Stale warning** — checkpoints older than 24h trigger a warning on load
- **Compacted context** — if context was compressed, Status notes it
- **Internal consistency** — if a gotcha says "X is impossible", Next Steps must not say "do X"

---

## 📁 Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill file — install into Hermes or Codex |
| `README.md` | This file |
| `claude-setup.md` | Snippet to append to `~/.claude/CLAUDE.md` |

---

<div align="center">
<sub>Made for <a href="https://github.com/NousResearch/hermes-agent">Hermes Agent</a> · Compatible with Claude Code and Codex</sub>
</div>
