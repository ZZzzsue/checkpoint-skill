---
name: checkpoint
description: >
  Cross-agent context handoff: write/read compact task checkpoint files.
  Triggered by mdw-<name> (write), mdr-<name> (read), mdl (list).
  Also supports: checkpoint write/read/list, 保存进度, 加载进度, 列出进度.
  Works identically in Hermes, Claude Code, Codex, or any LLM agent.
triggers:
  - "上下文太长"
  - "切换模型"
  - "save progress"
  - "context handoff"
---

# Task Checkpoint Skill

## Cross-Agent Usage

This skill lives in `~/.hermes/skills/` and is auto-loaded by Hermes.
The file format is plain markdown — any agent can read a checkpoint written by any other agent.

### Agent support matrix

| Agent | Load skill | Then use |
|---|---|---|
| Hermes | `/checkpoint` (auto-registered via skill system) | bare-text commands |
| Claude Code | `/checkpoint` or tell agent to load it | bare-text commands |
| Codex | `/use checkpoint` | bare-text commands |

**All three agents use identical bare-text trigger commands.** Custom slash commands are intentionally NOT used — Codex does not support them, and bare-text is the only genuinely cross-agent approach.

### Setup per agent

**Hermes**: No setup needed. `/checkpoint` is auto-registered when the skill is installed.

**Claude Code**: Add to `~/.claude/CLAUDE.md` or project `CLAUDE.md`:
```
When the user types mdw-<name>, mdr-<name>, or mdl (as a standalone message),
load and follow the checkpoint skill conventions:
  mdw-<name>   write checkpoint → ~/.hermes/checkpoints/<name>.md
  mdr-<name>   read checkpoint, summarize, offer to resume
  mdl          list all checkpoints with timestamps
Keep files under 80 lines. No secrets, no raw outputs.
```

**Codex**: Add to `~/.codex/AGENTS.md`:
```
Checkpoint commands (plain text — Codex does not support custom slash commands):
  mdw-<name>   write checkpoint → ~/.hermes/checkpoints/<name>.md
  mdr-<name>   read checkpoint, summarize, offer to resume
  mdl          list all checkpoints with timestamps
Only trigger when the user's entire message is exactly the command.
Keep files under 80 lines. No secrets, no raw outputs.
```

---

## Storage

All checkpoint files live in `~/.hermes/checkpoints/`.
Create the directory on first use: `mkdir -p ~/.hermes/checkpoints/`

File naming: `mdw-foo` → `~/.hermes/checkpoints/foo.md`

---

## Trigger Conventions

| User input | Needs skill_view? | Needs thinking? | Action |
|---|---|---|---|
| `mdw-<name>` | yes (first call) | recommended for long sessions | Write/overwrite checkpoint |
| `mdw-<name> [note]` | yes (first call) | recommended | Write checkpoint + append note |
| `mdr-<name>` | no | no | Read file + summarize 2-3 lines |
| `mdl` | no | no | `ls -lt ~/.hermes/checkpoints/` |

- `mdr-` and `mdl`: execute directly, do NOT call skill_view, no thinking needed.
- `mdw-`: load skill only on first call per session; if already loaded, use cached template.
- Thinking mode for `mdw-` helps on sessions >30 messages or complex multi-phase tasks.

**Activation requirement**: All trigger commands only activate when this skill is already
loaded in the session (via `/checkpoint` in Hermes, `/use checkpoint` in Codex, or
explicit load in Claude Code). Do NOT trigger on bare keywords appearing in normal
conversation, code snippets, or explanations.

### Alias patterns (map to canonical commands)

All of the following are equivalent and trigger the same action:

| User says | Maps to | Action |
|---|---|---|
| `mdw-<name>` | write | Write checkpoint |
| `checkpoint write <name>` | write | Write checkpoint |
| `checkpoint 写 <name>` | write | Write checkpoint |
| `保存进度 <name>` | write | Write checkpoint |
| `mdr-<name>` | read | Read checkpoint |
| `checkpoint read <name>` | read | Read checkpoint |
| `checkpoint 读 <name>` | read | Read checkpoint |
| `加载进度 <name>` | read | Read checkpoint |
| `mdl` | list | List checkpoints |
| `checkpoint list` | list | List checkpoints |
| `checkpoint 列出` | list | List checkpoints |
| `列出进度` | list | List checkpoints |

Name extraction rule: the `<name>` is the last token in the message.
If no name is given for write, use timestamp as filename: `YYYYMMDD-HHMM` (e.g. `20260617-0012`).
If no name is given for read, ask the user: "要加载哪个 checkpoint？用 `mdl` 查看列表。"

---

## Writing a Checkpoint (`mdw-<name>`)

Scan the entire conversation history (or compressed context summary) and extract:

1. **Goal** — the original task in one sentence, exactly as the user stated it
2. **Status** — where execution is right now (phase, last completed action)
3. **Findings** — high-signal discoveries only: bugs confirmed, endpoints found, credentials format, unexpected behaviors. Skip things that can be re-discovered trivially.
4. **Done** — ordered list of completed steps (verbs, specific)
5. **Next Steps** — ordered, immediately actionable. First item = resume point.
6. **Context** — paths, env vars (key names not values), tool versions, gotchas, pitfalls already hit. Anything that took time to figure out.
7. **Blockers** — if any. Empty section = omit entirely.

### Output template

```markdown
# [name] · checkpoint
_saved: YYYY-MM-DD HH:MM_

## Goal
[one sentence, verbatim from user intent]

## Status
[current phase / last completed action]

## Findings
- [finding — be specific, include evidence like "returns 403 not 404 → endpoint exists"]
- [finding]

## Done
- [x] [completed step 1]
- [x] [completed step 2]

## Next Steps
1. [most immediate action — agent can execute this without asking]
2. [following action]
3. [...]

## Context
- paths: [relevant file paths]
- tools: [tools/commands used, any flags that mattered]
- gotchas: [things that burned time or behaved unexpectedly]
- env: [key names, NOT values]

## Blockers
[omit section if none]
```

### Size discipline

**Target: under 80 lines total.** Apply compression asymmetrically:

**HEAD — preserve verbatim (do NOT compress):**
- Goal: one sentence, exact user intent
- Status: current phase, last completed action

**TAIL — preserve verbatim (do NOT compress):**
- Next Steps: ordered, immediately actionable — this is the resume point
- Context: paths, tools, gotchas, env key names
- Blockers: if any (omit section if none)

**MIDDLE — extract key signals, compress aggressively:**
- Findings: max 5 bullets. Only facts that changed a decision or would take >5 min to re-discover. Drop anything trivially re-findable.
- Done: max 8 items. Collapse sub-steps into single lines. No timestamps, no minor details.

Rule: if removing a middle item doesn't change what the next agent does, drop it.

### Write the file

Use the write_file tool (Hermes) or file write capability (Claude Code / Codex):
```
path: ~/.hermes/checkpoints/<name>.md
content: [generated markdown]
```

Confirm to user: "Checkpoint saved → ~/.hermes/checkpoints/<name>.md ([N] lines)"

---

## Reading a Checkpoint (`mdr-<name>`)

1. Read `~/.hermes/checkpoints/<name>.md`
2. Announce to user: "Loaded checkpoint [name] (saved: [timestamp])"
3. Summarize in 2-3 sentences: goal + current status + immediate next step
4. Ask: "Resume from Next Steps item 1, or different starting point?"
5. Proceed without waiting if user says "go" / "continue" / "直接继续"

---

## Publishing to GitHub (for skill distribution)

To publish this skill for others to install via `hermes skills install`:

1. Create a new GitHub repo (e.g. `checkpoint-skill`)
2. Repo structure required:
   ```
   checkpoint-skill/
   ├── SKILL.md          ← the skill file (frontmatter + body)
   └── references/
       └── README.md     ← user-facing documentation
   ```
3. Push SKILL.md and references/ — strip any local machine paths, tokens, or personal config from the content
4. Users install via: `hermes skills install https://raw.githubusercontent.com/<user>/checkpoint-skill/main/SKILL.md`

**Sensitive content checklist before publishing:**
- No hardcoded paths referencing a specific machine (e.g. `/home/sue/`)
- No API keys, tokens, or credentials
- No references to "already configured on this machine"
- AGENTS.md / CLAUDE.md snippets in the skill body should be generic templates, not machine-specific configs

- Do NOT include sensitive values (API keys, passwords, tokens) in the file — key names only.
- Do NOT include full HTTP responses or large JSON — extract the signal, not the raw data.
- If the session was already heavily compressed, note that in Status: "context was compacted, some history may be missing"
- When reading, do NOT silently assume the checkpoint is current — check the timestamp and warn if >24h old.
- The goal of this file is to get a fresh agent up to speed in <30 seconds, not to document everything.
- **Internal consistency**: sections must not contradict each other. If Gotchas says "X is impossible", Next Steps must not say "do X". A checkpoint that forces the reader to infer relationships between sections is a bad checkpoint — make the connection explicit.
- **Next Steps must be standalone**: each step must be actionable without cross-referencing other sections. If a step depends on a constraint, state the constraint inline: "Write ~/.codex/AGENTS.md with bare-text trigger conventions (Codex does not support slash commands — this is the only viable path)."
