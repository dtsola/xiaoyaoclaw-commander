---
type: project
status: active
progress: 100
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

**全部完成（100%）**：开发 + GitHub 发布 + 九件套互链 + ClawHub 提交并公开。GitHub `dtsola/xiaoyaoclaw-commander`（public/main/MIT）；ClawHub `xiaoyaoclaw-commander` **v1.0.1 已公开**（OpenClaw Cross-Tool Commander，安全扫描 clean，topics×5 生效，description 定稿版「Drive OpenClaw with Claude Code/Codex/etc」）。九件套生态闭环完成。

## 进度日志

- 2026-09-01 14:41：**立项**——指挥官指令将任务 claude-code-openclaw-research 升级为项目，命名 xiaoyaoclaw-commander（九件套），结构参考 xiaoyaoclaw-memory-distill；已建 projects/xiaoyaoclaw-commander/（SKILL.md 基线 + docs/ 调研双文档）
- 2026-09-01 14:45：**多工具标识定案 + SKILL.md 更新**——指挥官纠正「不需要泛化，只需要标识」：技能正文命令通用，仅 frontmatter 标识兼容工具（Claude Code/Codex/OpenCode/Trae/DSH）；技能名统一 openclaw-commander → xiaoyaoclaw-commander；README.md / README.en.md 双语完成（含各工具技能目录安装表）
- 2026-09-01 14:47-14:49：**DESIGN + hero + LICENSE + GitHub 发布**——docs/DESIGN.md（定位/核心问题/兼容标识表/工作流 Step 0-3/红线/发布计划）；hero.svg 暗色终端风（Chrome headless + PIL 像素扫描 + AI 视觉三重校验通过）；MIT LICENSE；git init main → 建仓 dtsola/xiaoyaoclaw-commander → push（http 代理 22307 通）→ About 中英 description + 5 topics（openclaw/agent-skills/claude-code/multi-agent/cli）；坑：PowerShell 内联脚本传中文 JSON 被 GBK 破坏 → 改 Python 脚本文件执行修复
- 2026-09-01 14:53-14:54：**全局技能回滚**——指挥官定案：本技能非 OpenClaw 全局技能（给外部工具用），删除误同步的 state/skills/xiaoyaoclaw-commander/
- 2026-09-01 14:55-14:58：**README 对齐 + 通用化**——README 对齐 memory-distill 结构（hero/徽章/为什么需要它/特性/安装/快速上手/对比表/目录结构/定制广告/交流群/姊妹项目）；「为什么需要它」通用化（面向所有想用外部智能体操作 OpenClaw/小遥Claw 的用户）；community-qr.png 补齐
- 2026-09-01 15:00-15:03：**SKILL.md 安全格式对齐**——按 kb-retriever 格式新增「能力范围与写操作声明（权限透明）」（身份/涉及操作/边界承诺）+「禁止行为」5 条红线 + GitHub/小遥Claw 引用块；frontmatter YAML 校验通过 + 全项目敏感信息扫描 CLEAN
- 2026-09-01 15:04-15:08：**九件套 README 互链闭环**——8 仓库 16 个 README（中英）全部追加 commander 姊妹项目条目；「七件套/八件套/on top of the seven」标注统一为九件套/生态；全部 push 成功
- 2026-09-01 15:12-15:14：**显示名定案 + description 调整**——指挥官定名 **OpenClaw Cross-Tool Commander**（否决长名「For Claude Code / Codex etc」）；SKILL.md description 改为英文在前中文在后；README 双语 + hero.svg 同步；push f6fecff
- 2026-09-01 15:16：**ClawHub v1.0.0 提交**——categories automation/agents/productivity，topics multi-agent/orchestration/cli/agent-skills/command，source f6fecff，versionId k973d99vf7s5a2wqydq3z2mqbs8dka6x
- 2026-09-01 15:21-15:31：**description 直白化三轮迭代**——指挥官要求「非常直白」：版本 A 定稿「Drive OpenClaw with Claude Code, Codex, OpenCode, Trae, DSH etc. — command the gateway, its agents and channels, from any Agent Skills tool. 中文：用 Claude Code / Codex / OpenCode / Trae / DSH 等工具驱动 OpenClaw 干活——从任意支持 Agent Skills 的工具指挥网关、agent 和通道」；push 953bf2c
- 2026-09-01 15:32-15:33：**ClawHub v1.0.1 提交**（新描述版，versionId k97164bawvj5835kw1bv07h4zs8dk8ym）
- 2026-09-01 15:35：**ClawHub 公开 ✅（指挥官操作）**——线上验证通过：显示名/描述/topics×5/latest 1.0.1/安全扫描 clean（scanner.llm.clean）；**九件套全部公开，生态闭环完成**

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
