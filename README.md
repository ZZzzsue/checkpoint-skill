<div align="center">

# 🔖 checkpoint-skill

**Cross-agent task context handoff — save progress, resume anywhere.**

*Works identically in Hermes, Claude Code, and Codex.*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Hermes](https://img.shields.io/badge/Hermes-✓-gold)](https://github.com/NousResearch/hermes-agent)
[![Claude Code](https://img.shields.io/badge/Claude_Code-✓-orange)](https://docs.anthropic.com/claude-code)
[![Codex](https://img.shields.io/badge/Codex-✓-green)](https://github.com/openai/codex)

**[🇨🇳 中文说明](README_ZH.md)**

</div>

---

## 🧩 What is this?

When your context window gets too long, you switch models, or hand off between agents — you lose all state.
This skill distills your current task into a compact markdown file that any agent can load and resume from in under 30 seconds.

```
Long session → mdw-myproject → checkpoint saved → new session → mdr-myproject → resume instantly
```

---

## ⚡ Installation

### Hermes
```bash
hermes skills install https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md
```
After install, `/checkpoint` is auto-registered as a slash command — no extra setup needed.

### Claude Code
```bash
mkdir -p ~/.claude/skills/checkpoint && \
curl -sL https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md \
  > ~/.claude/skills/checkpoint/SKILL.md
```
After install, use `/use checkpoint` to load.

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
| Claude Code | `/use checkpoint` |
| Codex | `/use checkpoint` |

### Step 2 — Use commands

> Type as a **standalone message** — the entire message is the command.

| Command | Action |
|---------|--------|
| `mdw-<name>` | Save current task progress |
| `mdr-<name>` | Load checkpoint and resume |
| `mdl` | List all checkpoints |

**Aliases also work:**

| Command | Action |
|---------|--------|
| `checkpoint write <name>` | Write |
| `checkpoint read <name>` | Read |
| `checkpoint list` | List |

**No name?** `mdw-` without a name auto-generates a timestamp: `20260617-0012.md`

---

## 📄 Checkpoint Format

Saved to `~/.hermes/checkpoints/<name>.md`

```markdown
# myproject · checkpoint
_saved: 2026-06-17 00:12_

## Goal
Original task in one sentence

## Status
Current phase + last completed action

## Findings
- High-signal finding with evidence

## Done
- [x] Completed step

## Next Steps
1. Immediately actionable — this is the resume point

## Context
- paths: ...
- tools: ...
- gotchas: ...
- env: KEY NAMES only, never values

## Blockers
Omit if none
```

**Compression rules:**

| Section | Rule |
|---------|------|
| HEAD — Goal + Status | Preserve verbatim |
| TAIL — Next Steps + Context + Blockers | Preserve verbatim |
| MIDDLE — Findings + Done | Max compress (limit 5 + 8 items) |
| Total | Under 80 lines |

---

## 🔄 Cross-Agent Workflow

```
Hermes (long context)    →  mdw-feature-auth  →  checkpoint saved
Claude Code              →  mdr-feature-auth  →  resumed
Codex                    →  mdr-feature-auth  →  same file, same resume point
```

Plain markdown format — any agent can read what any other agent wrote.

---

## ⚠️ Pitfalls

- 🔒 No secrets — key names only, never actual values
- 📵 No raw output — extract the signal, not the full response
- ⏰ Stale warning triggered after 24h
- 🗜️ Compacted context noted in Status field
- ✅ No internal contradictions between sections

---

<div align="center">
<sub>Built for <a href="https://github.com/NousResearch/hermes-agent">Hermes Agent</a> · Compatible with Claude Code and Codex</sub>
</div>
