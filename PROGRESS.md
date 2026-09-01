---
type: project
status: active
progress: 25
created: 2026-09-01
updated: 2026-09-01
docs:
  - path: SKILL.md
    desc: 技能主体（xiaoyaoclaw-commander，多工具标识版）
  - path: README.md / README.en.md
    desc: 中英双语 README（含各工具技能目录安装表）
  - path: docs/research-report-claude-code-openclaw.md
    desc: 调研报告（Claude Code 连接 OpenClaw 全路径：MCP/ACP/CLI）
  - path: docs/research-00-problem-decomposition.md
    desc: 调研问题拆解（Q1-Q4 边界界定）
---

# xiaoyaoclaw-commander（跨工具指挥官）

## 目标 / 背景

**九件套第九件：跨工具指挥层**。由任务 `claude-code-openclaw-research` 升级立项——核心产物 `~/.claude/skills/openclaw-commander`（Claude Code 侧技能）**指挥官已实测通过**（Claude Code Bash 调 openclaw CLI 走 Gateway，614ms 链路通）。

- **定位**：让任意支持 Agent Skills 标准的工具（Claude Code / Codex / OpenCode / Trae / DSH 等）通过 openclaw CLI 指挥本机小遥Claw / OpenClaw 网关——执行 agent 任务、通道发消息、状态查询
- **核心能力**：openclaw 可执行文件双平台动态探测（不写死路径）+ 环境变量补齐（OPENCLAW_STATE_DIR / OPENCLAW_CONFIG_PATH）+ agent 名称↔id 动态映射 + 强制显式指定 agent（禁默认）
- **命名**：`xiaoyaoclaw-commander`（与八件套命名体系对齐）；本地已测试技能暂名 `openclaw-commander`，发布版统一为 `xiaoyaoclaw-commander`
- **项目结构**：参考 xiaoyaoclaw-memory-distill（PROGRESS.md + SKILL.md + README 中英双语 + docs/DESIGN.md + templates/）

## 当前状态

开发推进中（25%）：项目目录 + 进度卡已建，SKILL.md 已统一命名并加多工具标识（指挥官定案：**不需要泛化，只需标识**——Agent Skills 是开放标准，openclaw 命令通用，各工具仅技能目录不同），README 中英双语已写。待办：docs/DESIGN.md、hero.svg、GitHub + ClawHub 发布（按 ClawHub 确认制走流程）。

## 进度日志

- 2026-09-01 14:41：**立项**——指挥官指令将任务 claude-code-openclaw-research 升级为项目，命名 xiaoyaoclaw-commander（九件套），结构参考 xiaoyaoclaw-memory-distill；已建 projects/xiaoyaoclaw-commander/（SKILL.md 基线 + docs/ 调研双文档）
- 2026-09-01 14:45：**多工具标识定案 + SKILL.md 更新**——指挥官纠正「不需要泛化，只需要标识」：技能正文命令通用，仅 frontmatter 标识兼容工具（Claude Code/Codex/OpenCode/Trae/DSH）；技能名统一 openclaw-commander → xiaoyaoclaw-commander；README.md / README.en.md 双语完成（含各工具技能目录安装表）

## 文档索引

| 文档 | 说明 | 更新 |
|------|------|------|
| SKILL.md | 技能主体（openclaw-commander 已测试版基线） | 2026-09-01 |
| docs/research-report-claude-code-openclaw.md | 调研报告（MCP/ACP/CLI 三路径 + 最终 CLI 方案） | 2026-08-31 |
| docs/research-00-problem-decomposition.md | 调研问题拆解 | 2026-08-31 |

<!--
使用说明（agent 维护，用户可忽略）：
- status: active | paused | archived
- progress: 0-100，时刻维护（每次更新进度日志时同步调整）
- 进度日志只追加不删除
- 重要文档：移入 docs/ 或记录路径，追加到 docs 数组（机器可读）+ 本表格（人可读）
- 项目完结：status 改 archived + 关键结论记入 MEMORY.md（供 memory-distill 蒸馏）
-->
