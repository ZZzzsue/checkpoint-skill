<div align="center">

# 🔖 checkpoint-skill

**跨 Agent 任务进度保存与恢复**

*在 Hermes、Claude Code、Codex 中无缝切换，任务进度永不丢失*

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Hermes](https://img.shields.io/badge/Hermes-✓-gold)](https://github.com/NousResearch/hermes-agent)
[![Claude Code](https://img.shields.io/badge/Claude_Code-✓-orange)](https://docs.anthropic.com/claude-code)
[![Codex](https://img.shields.io/badge/Codex-✓-green)](https://github.com/openai/codex)

**[🇺🇸 English](README.md)**

</div>

---

## 🧩 这是什么？

上下文太长、切换模型、跨 Agent 交接任务时，你会丢失所有状态。
这个 skill 把当前任务蒸馏成一个精简 markdown 文件，任何 Agent 读入后 30 秒内接续。

```
上下文过长 → mdw-myproject → 保存进度 → 新 session → mdr-myproject → 无缝继续
```

---

## ⚡ 安装

### Hermes
```bash
hermes skills install https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md
```
安装后 `/checkpoint` 自动注册为 slash 命令，无需额外配置。

### Claude Code
```bash
mkdir -p ~/.claude/skills/checkpoint && \
curl -sL https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md \
  > ~/.claude/skills/checkpoint/SKILL.md
```
安装后用 `/use checkpoint` 加载。

### Codex
```bash
mkdir -p ~/.codex/skills/checkpoint && \
curl -sL https://raw.githubusercontent.com/ZZzzsue/checkpoint-skill/main/SKILL.md \
  > ~/.codex/skills/checkpoint/SKILL.md
```
安装后用 `/use checkpoint` 加载。

---

## 🚀 使用方法

### 第一步：加载 skill

| Agent | 命令 |
|-------|------|
| Hermes | `/checkpoint` |
| Claude Code | `/use checkpoint` |
| Codex | `/use checkpoint` |

### 第二步：使用命令

> 命令以**独立消息**发送（整条消息就是命令本身）

| 命令 | 说明 |
|------|------|
| `mdw-<name>` | 保存当前任务进度 |
| `mdr-<name>` | 加载检查点并恢复 |
| `mdl` | 列出所有检查点 |

**别名也支持：**

| 命令 | 说明 |
|------|------|
| `checkpoint write <name>` / `保存进度 <name>` | 写入 |
| `checkpoint read <name>` / `加载进度 <name>` | 读取 |
| `checkpoint list` / `列出进度` | 列出 |

**不写名字？** `mdw-` 不带名称时自动以时间命名：`20260617-0012.md`

---

## 📄 检查点格式

文件保存在 `~/.hermes/checkpoints/<name>.md`

```markdown
# myproject · checkpoint
_saved: 2026-06-17 00:12_

## Goal
原始任务一句话

## Status
当前阶段 + 最后完成的操作

## Findings
- 高价值发现，附证据

## Done
- [x] 已完成步骤

## Next Steps
1. 立即可执行的下一步（恢复点）

## Context
- paths: 相关路径
- tools: 工具和关键参数
- gotchas: 踩过的坑
- env: 环境变量名（不含值）

## Blockers
卡住的问题（无则省略此节）
```

**压缩规则：**

| 区域 | 规则 |
|------|------|
| HEAD — Goal + Status | 原文保留 |
| TAIL — Next Steps + Context + Blockers | 原文保留 |
| MIDDLE — Findings + Done | 最大压缩，上限 5+8 条 |
| 总计 | 不超过 80 行 |

---

## 🔄 跨 Agent 工作流

```
Hermes（上下文过长）  →  mdw-feature-auth  →  保存检查点
Claude Code          →  mdr-feature-auth  →  加载并继续
Codex                →  mdr-feature-auth  →  同一文件，同一恢复点
```

检查点是纯 markdown — 任何 Agent 都能读取其他 Agent 写入的内容。

---

## ⚠️ 注意事项

- 🔒 **不写密钥** — 只记变量名，不记实际值
- 📵 **不写原始输出** — 提取信号，不写完整响应
- ⏰ **过期提醒** — 超过 24 小时加载时会提示
- 🗜️ **压缩标记** — 上下文被压缩时 Status 中注明
- ✅ **内部一致** — Gotchas 说"X 不可能"，Next Steps 就不能写"做 X"

---

<div align="center">
<sub>为 <a href="https://github.com/NousResearch/hermes-agent">Hermes Agent</a> 构建 · 兼容 Claude Code 和 Codex</sub>
</div>
