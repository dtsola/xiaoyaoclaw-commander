# xiaoyaoclaw-commander

> 9th suite skill: **cross-tool command layer** — command your local XiaoyaoClaw / OpenClaw multi-agent system from any Agent Skills-compatible tool.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## What is this

An **Agent Skills standard skill** that drives your local XiaoyaoClaw / OpenClaw gateway through the openclaw CLI:

- **Run tasks**: `openclaw agent -m "<task>" --agent <id> --json` — let any agent (tiantong / xiaoguang / xiaozhi ...) do the work
- **Send messages**: `openclaw message send --channel feishu -t <id> -m "..."` — via Feishu / Telegram / etc.
- **Check status**: `openclaw health` / `openclaw agents list` / `openclaw sessions`

**Works in any tool supporting Agent Skills**: Claude Code / Codex / OpenCode / Trae / DSH — the openclaw commands are fully universal, no per-tool customization needed.

## Why

The XiaoyaoClaw desktop app embeds openclaw in its app directory (not on PATH), and agent config is located via `OPENCLAW_STATE_DIR` / `OPENCLAW_CONFIG_PATH` env vars (injected only in the desktop app's own process). Running openclaw from a terminal fails to find the binary, and `agents list` shows only `main`.

This skill solves three things:

1. **Dynamically locates** the openclaw binary (cross-platform, no hardcoded paths)
2. **Fills in env vars** so terminals / external tools see all real agents
3. **Maps agent names ↔ ids**, enforcing explicit `--agent` (no default agent)

## Install

Copy the `SKILL.md` folder into your tool's skills directory:

| Tool | Skills directory |
|------|-----------------|
| Claude Code | `~/.claude/skills/xiaoyaoclaw-commander/` |
| Codex | `~/.codex/skills/xiaoyaoclaw-commander/` |
| OpenCode | see its Agent Skills docs |
| Trae | see its Agent Skills docs |
| DSH | see its Agent Skills docs |
| Any other Agent Skills tool | its skills directory |

## Quick start

```bash
# List agents (with Chinese identities)
openclaw agents list

# Ask tiantong to research X (agent must be explicit!)
openclaw agent -m "Research X, output a report" --agent tiantong --json

# Send a Feishu message
openclaw message send --channel feishu --target <user/chat id> -m "Hello"
```

> ⚠️ **Hard rule**: `--agent` is mandatory — if the user doesn't say who should do it, ask first. Never fall back to a default agent. Agent names (any language/emoji) are resolved via the `Identity` field of `agents list`; if no match, ask the user — never guess.

## Project layout

```
xiaoyaoclaw-commander/
├── SKILL.md          # Skill body (any Agent Skills tool)
├── docs/
│   ├── research-report-claude-code-openclaw.md   # Research (MCP/ACP/CLI paths)
│   └── research-00-problem-decomposition.md      # Problem decomposition
└── PROGRESS.md       # Project progress card
```

## Design highlights

- **No hardcoded paths**: multi-level probing on Windows / macOS / Linux (XiaoyaoClaw desktop → PATH → vanilla desktop → local prefixes)
- **Auto env var filling**: derives state dir for desktop installs
- **Universal commands**: only the invocation prefix differs (PowerShell `& $oc` / bash `"$OC"`), arguments identical
- **Version-agnostic**: gracefully avoids `mcp`/`migrate` on older openclaw (v2026.3.x), defers to `--help`

## Xiaoyao ecosystem (9-skill suite)

| # | Skill | Role |
|---|-------|------|
| 1 | xiaoyaoclaw-workspace-initializer | Home: directory standards |
| 2 | xiaoyaoclaw-memory-distill | Content: memory distillation |
| 3 | xiaoyaoclaw-task-progress-tracker | Status: task/project tracking |
| 4 | xiaoyaoclaw-kb-retriever | Knowledge: local KB retrieval |
| 5 | xiaoyaoclaw-workspace-auditor | Health: workspace audit |
| 6 | xiaoyaoclaw-web-clipper | Input: web clipping |
| 7 | xiaoyaoclaw-agent-orchestrator | Collaboration: multi-agent orchestration |
| 8 | xiaoyaoclaw-usage-report | Metrics: usage reporting |
| 9 | **xiaoyaoclaw-commander** | **Command: cross-tool command layer** |

## License

MIT
