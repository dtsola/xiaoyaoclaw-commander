# xiaoyaoclaw-commander（小遥指挥官）

> 第九件套：**跨工具指挥层**——让任意支持 Agent Skills 的工具指挥小遥Claw / OpenClaw 多 agent 系统。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 这是什么

一个 **Agent Skills 标准技能**，通过 openclaw CLI 指挥本机小遥Claw / OpenClaw 网关：

- **执行任务**：`openclaw agent -m "<任务>" --agent <id> --json` → 让任意 agent（天桐/小光/小智…）干活
- **收发消息**：`openclaw message send --channel feishu -t <id> -m "..."` → 经飞书/Telegram 等通道发消息
- **状态查询**：`openclaw health` / `openclaw agents list` / `openclaw sessions`

**兼容任何支持 Agent Skills 标准的工具**：Claude Code / Codex / OpenCode / Trae / DSH 等——openclaw 命令完全通用，无需按工具定制。

## 为什么需要它

小遥Claw 桌面版把 openclaw 内嵌在应用目录（不在 PATH），且 agent 配置靠 `OPENCLAW_STATE_DIR` / `OPENCLAW_CONFIG_PATH` 环境变量定位（桌面版只在自身进程注入）。终端里直接敲 openclaw 会找不到命令、`agents list` 只剩 `main`。

本技能解决三件事：

1. **动态探测** openclaw 可执行文件（双平台，不写死路径）
2. **补齐环境变量**，让终端/外部工具看到真实的 7 个 agent
3. **agent 名称 ↔ id 动态映射**，强制显式指定 agent（禁止默认）

## 安装

把 `SKILL.md`（所在目录）复制到你的工具的技能目录：

| 工具 | 技能目录 |
|------|---------|
| Claude Code | `~/.claude/skills/xiaoyaoclaw-commander/` |
| Codex | `~/.codex/skills/xiaoyaoclaw-commander/` |
| OpenCode | 见其 Agent Skills 文档 |
| Trae | 见其 Agent Skills 文档 |
| DSH | 见其 Agent Skills 文档 |
| 其他支持 Agent Skills 的工具 | 对应技能目录 |

## 快速上手

```bash
# 查看有哪些 agent（含中文名 Identity）
openclaw agents list

# 让天桐调研 X（agent 必须显式指定！）
openclaw agent -m "调研 X，输出报告" --agent tiantong --json

# 经飞书发消息
openclaw message send --channel feishu --target <user/chat id> -m "你好"
```

> ⚠️ **硬性规则**：`--agent` 必须指定——用户没说让谁干时先问，禁止用默认 agent。
> 智能体名称（中文/英文/emoji）以 `agents list` 的 Identity 字段为权威，匹配不到就问用户，不要猜。

## 项目结构

```
xiaoyaoclaw-commander/
├── SKILL.md          # 技能主体（兼容任意 Agent Skills 工具）
├── docs/
│   ├── research-report-claude-code-openclaw.md   # 调研报告（MCP/ACP/CLI 三路径）
│   └── research-00-problem-decomposition.md      # 问题拆解
└── PROGRESS.md       # 项目进度卡
```

## 设计要点

- **路径不写死**：Windows / macOS / Linux 多级探测（小遥桌面版 → PATH → 原版桌面版 → 本地前缀）
- **环境变量自动补齐**：桌面版安装场景自动推导 state 目录
- **命令通用**：探测后仅调用前缀不同（PowerShell `& $oc` / bash `"$OC"`），参数完全一致
- **不依赖版本**：较旧 openclaw（v2026.3.x）无 `mcp`/`migrate` 子命令时自动规避，以 `--help` 为准

## 小遥生态（九件套）

本技能是「度量」后的第九件：**跨工具指挥层**，让外部编码工具（Claude Code 等）能指挥整个小遥多 agent 系统。

| # | 技能 | 定位 |
|---|------|------|
| 1 | xiaoyaoclaw-workspace-initializer | 家：目录规范 |
| 2 | xiaoyaoclaw-memory-distill | 内容：记忆蒸馏 |
| 3 | xiaoyaoclaw-task-progress-tracker | 状态：任务/项目进度 |
| 4 | xiaoyaoclaw-kb-retriever | 知识：本地知识库检索 |
| 5 | xiaoyaoclaw-workspace-auditor | 健康：工作区体检 |
| 6 | xiaoyaoclaw-web-clipper | 输入：网页剪藏 |
| 7 | xiaoyaoclaw-agent-orchestrator | 协作：多 agent 编排 |
| 8 | xiaoyaoclaw-usage-report | 度量：用量报告 |
| 9 | **xiaoyaoclaw-commander** | **指挥：跨工具指挥层** |

## License

MIT
