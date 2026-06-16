
## Checkpoint Skill

When the user types any of the following as a **standalone message** (whole message only),
follow the checkpoint skill conventions:

| Command | Action |
|---------|--------|
| `mdw-<name>` / `checkpoint write <name>` / `保存进度 <name>` | Write checkpoint |
| `mdr-<name>` / `checkpoint read <name>` / `加载进度 <name>` | Read checkpoint |
| `mdl` / `checkpoint list` / `列出进度` | List checkpoints |

**Write** → save to `~/.hermes/checkpoints/<name>.md` (under 80 lines, no secrets, no raw output).
**Read** → load file, summarize goal + status + next step, offer to resume.
**List** → run `ls -lt ~/.hermes/checkpoints/` and show results.

No name on write → use timestamp `YYYYMMDD-HHMM` as filename.
No name on read → ask "要加载哪个 checkpoint？用 `mdl` 查看列表。"

Format: HEAD (Goal+Status) + TAIL (NextSteps+Context+Blockers) preserved verbatim.
MIDDLE (Findings ≤5 + Done ≤8) compressed aggressively.
