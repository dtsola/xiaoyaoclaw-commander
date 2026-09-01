---
type: project
status: active
progress: 60
created: 2026-09-01
updated: 2026-09-01
docs:
  - path: SKILL.md
    desc: 技能主体（xiaoyaoclaw-commander，多工具标识版）
  - path: README.md / README.en.md
    desc: 中英双语 README（含各工具技能目录安装表）
  - path: docs/DESIGN.md
    desc: 设计文档（定位 + 核心问题 + 兼容标识 + 工作流 + 红线 + 发布计划）
  - path: assets/readme/hero.svg
    desc: README 封面图（暗色终端风，三重校验通过）
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
- **⚠️ 非 OpenClaw 全局技能**（指挥官定案 2026-09-01）：本技能给**外部工具**（Claude Code/Codex/OpenCode/Trae/DSH）用，**不**同步到 OpenClaw 的 state/skills/ 全局技能目录（已删除误同步的副本）

## 当前状态

开发 + GitHub 发布完成（60%）：DESIGN.md、hero.svg（三重校验）、LICENSE、README 双语齐备；GitHub `dtsola/xiaoyaoclaw-commander` 已公开（public/main/MIT/5 topics/中英 description）。待办：ClawHub 提交（走确认制）、八件套 README 互链、community-qr.png。

## 进度日志

- 2026-09-01 14:41：**立项**——指挥官指令将任务 claude-code-openclaw-research 升级为项目，命名 xiaoyaoclaw-commander（九件套），结构参考 xiaoyaoclaw-memory-distill；已建 projects/xiaoyaoclaw-commander/（SKILL.md 基线 + docs/ 调研双文档）
- 2026-09-01 14:45：**多工具标识定案 + SKILL.md 更新**——指挥官纠正「不需要泛化，只需要标识」：技能正文命令通用，仅 frontmatter 标识兼容工具（Claude Code/Codex/OpenCode/Trae/DSH）；技能名统一 openclaw-commander → xiaoyaoclaw-commander；README.md / README.en.md 双语完成（含各工具技能目录安装表）
- 2026-09-01 14:47-14:49：**DESIGN + hero + LICENSE + GitHub 发布**——docs/DESIGN.md（定位/核心问题/兼容标识表/工作流 Step 0-3/红线/发布计划）；hero.svg 暗色终端风（Chrome headless + PIL 像素扫描 + AI 视觉三重校验通过）；MIT LICENSE；git init main → 建仓 dtsola/xiaoyaoclaw-commander → push（http 代理 22307 通）→ About 中英 description + 5 topics（openclaw/agent-skills/claude-code/multi-agent/cli）；坑：PowerShell 内联脚本传中文 JSON 被 GBK 破坏 → 改 Python 脚本文件执行修复
- 2026-09-01 14:53-14:54：**全局技能回滚**——指挥官定案：本技能非 OpenClaw 全局技能（给外部工具用），删除误同步的 state/skills/xiaoyaoclaw-commander/

## 文档索引

| 文档 | 说明 | 更新 |
|------|------|------|
| SKILL.md | 技能主体（多工具标识版） | 2026-09-01 |
| README.md / README.en.md | 中英双语 README | 2026-09-01 |
| docs/DESIGN.md | 设计文档 | 2026-09-01 |
| assets/readme/hero.svg | 封面图 | 2026-09-01 |
| docs/research-report-claude-code-openclaw.md | 调研报告 | 2026-08-31 |
| docs/research-00-problem-decomposition.md | 问题拆解 | 2026-08-31 |

<!--
使用说明（agent 维护，用户可忽略）：
- status: active | paused | archived
- progress: 0-100，时刻维护（每次更新进度日志时同步调整）
- 进度日志只追加不删除
- 重要文档：移入 docs/ 或记录路径，追加到 docs 数组（机器可读）+ 本表格（人可读）
- 项目完结：status 改 archived + 关键结论记入 MEMORY.md（供 memory-distill 蒸馏）
-->
