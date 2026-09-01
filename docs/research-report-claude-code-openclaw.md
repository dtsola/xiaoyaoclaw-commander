# Claude Code × OpenClaw 技能连接调研报告

> 调研日期：2026-08-31 | 调研人：天桐 | 类型：知识梳理 + 落地决策支持
> 时效窗口：2026 年内有效（AI 领域 3-6 个月）

## 一、核心结论（TL;DR）

**OpenClaw 技能与 Claude Code 技能是同一套开放标准（Agent Skills / SKILL.md），格式完全同构，技能文件可直接双向复制使用。**

官方已提供 3 条连接路径（迁移 / MCP / ACP），其中：
- ✅ **技能互通最简单方式 = 复制目录**（零依赖、无配置）
- ✅ **官方迁移工具** `openclaw migrate claude` 支持 Claude → OpenClaw 单向批量导入
- ✅ **深度集成**（OpenClaw 会话里直接跑 Claude Code）走 ACP Agents（acpx 插件）

---

## 二、技能机制对比（两边都是 SKILL.md）

| 维度 | OpenClaw | Claude Code |
|---|---|---|
| 技能格式 | 目录 + `SKILL.md`（YAML frontmatter + Markdown） | 目录 + `SKILL.md`（同上） |
| 遵循标准 | Agent Skills 开放标准（agentskills.io），ClawHub 明确声明 portable Agent Skills | Agent Skills 开放标准（官方文档明示） |
| 必填字段 | `name` + `description` | `name` + `description` |
| 存放位置 | `skills/`（workspace）等加载优先级链 | `~/.claude/skills/<name>/`（个人）、`.claude/skills/`（项目） |
| 调用方式 | 自动触发（description 匹配）+ `/skill-name` | 自动触发 + `/skill-name` |
| 扩展字段 | `metadata.openclaw`（requires.env/bins、primaryEnv、os、install）、`user-invocable`、`disable-model-invocation`、`command-dispatch` | invocation control、subagent execution、dynamic context injection（!`cmd`） |
| 注册中心 | ClawHub（clawhub.ai） | 无官方中心（社区分散） |

**关键结论**：两边 frontmatter 核心（name/description）与目录结构 100% 兼容。扩展字段互不识别但**不会报错**（未知字段被忽略）。跨系统使用时仅需注意技能正文里引用的工具差异。

---

## 二.5、指挥官澄清后的核心需求：Claude Code 直连 OpenClaw

> 需求原文：「通过 claude code 直接连接 openclaw」→「走acp方式的，claude code 有没有相关的技能」
> 即：想用 ACP 方式让 Claude Code 连上 OpenClaw，问 Claude Code 侧需要什么技能/配置。

### ⚠️ ACP 角色澄清（关键）

**ACP 协议角色**（官方定义）：`代码编辑器/IDE（ACP client）↔ 编码 agent（ACP server/harness）`

| 端 | 角色 | 典型代表 |
|---|---|---|
| ACP client | 编辑器/IDE，发起会话 | Zed、VS Code 插件 |
| ACP server | 编码 agent，被拉起干活 | Claude Code、Codex、Gemini CLI |

**结论：Claude Code 是 ACP 的 server/harness 端，不是 client 端。**

- 实测本机 Claude Code v2.1.143 全部 CLI 参数：**无任何 ACP 选项**（无 `--acp`、无 ACP client 模式）
- Claude Code 官方文档索引：只有 MCP 和 LLM Gateway 协议，**没有 ACP 文档**
- 因此 Claude Code **不能主动作为 ACP client 去连接 OpenClaw 的 `openclaw acp` bridge**（那个 bridge 是给 IDE/编辑器用的）

### 正确的 ACP 姿势：OpenClaw 拉起 Claude Code（方向相反）

走 ACP 的正确方向是 **OpenClaw 作为宿主（acpx 插件），拉起 Claude Code 作为 harness 干活**：

```bash
openclaw plugins install @openclaw/acpx
openclaw config set plugins.entries.acpx.enabled true
# 然后：/acp spawn claude（聊天里）或 sessions_spawn({ runtime: "acp", agentId: "claude" })
```

- 此时**技能在 OpenClaw 侧加载**（Claude Code 的编码能力被 OpenClaw 指挥）
- Claude Code 侧**不需要任何技能/配置**——acpx 自动下载 Claude ACP adapter，只需本机已有 Claude Code auth
- 本机 acpx 插件未装（需先安装）

### 若坚持「Claude Code 主动连 OpenClaw」：只有 MCP 一条路

- `openclaw mcp serve`（OpenClaw 作 MCP server，Claude Code 作 MCP client）——文档明示支持 Claude Code
- 但本机内嵌 openclaw 是 **v2026.3.7，没有 `mcp` 子命令**（实测完整命令列表确认）；官方文档对应最新 stable v2026.8.1
- 需升级 runtime 后才可用

---

## 二.6、Claude Code 侧「技能」与 ACP 的关系（回答指挥官问题）

- Claude Code 的「技能」= Agent Skills（`~/.claude/skills/*/SKILL.md`），是**给 Claude Code 自己用的**，与 ACP 连接**无直接关系**
- 走 ACP 方式连 OpenClaw：**Claude Code 侧不需要任何技能**——它只是被拉起的 harness，技能全部在 OpenClaw 侧
- 想要 Claude Code 在 ACP 会话里用 OpenClaw 的技能？不需要——OpenClaw agent（如天桐）负责读技能、指挥 Claude Code 执行
- 现有 `~/.claude/skills/` 的 5 个技能（better-icons/find-skills/frontend-design/grill-me/skill-creator）照常可用，与 ACP 无关

---

## 二.7、最终落地：Claude Code 指挥 OpenClaw（不升级 runtime）✅ 已实现

指挥官需求：「Claude Code 指挥 OpenClaw，但不升级 runtime」——不走 MCP（本机 v2026.3.7 无 mcp 命令），直接用**本机现成的 openclaw CLI 命令**。

### 原理

Claude Code 自带 Bash 工具 → 直接调用 `openclaw` CLI（走 Gateway，非 --local）→ 指挥任意 agent / 收发消息。

**已实测验证（2026-08-31 19:43）：**
```
openclaw agent -m "回复OK" --agent tiantong --json
→ 成功返回 "链路通"（614ms，deepseek-v4-flash，session agent:tiantong:main）
```
- `openclaw health` 正常：Feishu ok，7 个 agent 在线（liliai 默认/tiantong/xiaogang/xiaoguang/xiaoxia/xiaoyao/xiaozhi）

### 已创建技能：`openclaw-commander`（Claude Code 侧）

位置：`~/.claude/skills/openclaw-commander/SKILL.md`（Agent Skills 标准格式，已验证加载）

技能内容：
| 命令 | 用途 |
|---|---|
| `openclaw agent -m "<任务>" --agent <id> --json` | 让指定 OpenClaw agent 执行任务（核心） |
| `openclaw message send --channel feishu -t <id> -m "..."` | 经 OpenClaw 通道发消息 |
| `openclaw health` | 查看网关/agent 状态 |
| `openclaw sessions` | 查看会话 |

openclaw 路径：`C:\Users\Administrator\appdata\local\programs\xiaoyaoclaw-desktop\resources\runtime\openclaw\bin\openclaw.cmd`

### 效果

Claude Code 里直接说「让天桐调研 X」「给飞书发消息」→ Claude Code 自动加载 openclaw-commander 技能 → 调 CLI 指挥 OpenClaw。零升级、零插件、纯 CLI。

---

## 三、官方连接路径（3 条）

### 路径 1：`openclaw migrate claude` —— 技能批量迁移（Claude → OpenClaw）

OpenClaw 内置 Claude 迁移 provider，官方文档：[Migrating from Claude](https://docs.openclaw.ai/install/migrating-claude)

**导入内容：**
- ✅ Claude 技能（含 `SKILL.md` 的目录）→ 复制进 OpenClaw workspace skills 目录
- ✅ Claude commands（`.claude/commands/*.md`）→ 转成 OpenClaw 技能（`disable-model-invocation: true`）
- ✅ `CLAUDE.md` / `.claude/CLAUDE.md` → `AGENTS.md`；`~/.claude/CLAUDE.md` → `USER.md`
- ✅ MCP server 定义（`.mcp.json`、`~/.claude.json`、Claude Desktop 配置）
- ⚠️ 不导入：hooks、权限 allowlist、subagents（`.claude/agents/`）、历史记录（仅归档）

**用法：**
```bash
openclaw migrate claude --dry-run    # 预览计划（安全，不改动）
openclaw migrate apply claude --yes  # 应用（自动备份）
openclaw doctor                      # 迁移后体检
```

### 路径 2：`openclaw mcp serve` —— OpenClaw 作为 MCP server（Claude Code 当 MCP client）

官方文档：[MCP](https://docs.openclaw.ai/mcp)

- Claude Code 通过 MCP 直连 OpenClaw 的**通道会话**（conversations_list / messages_read / messages_send / events_wait 等）
- 有专门的 **Claude Code client mode**（`--claude-channel-mode on`，含 Claude 专属推送）
- **注意**：这条路径暴露的是「会话/消息」，**不是技能**——适合让 Claude Code 收发 OpenClaw 通道消息，不适合技能共享

```bash
openclaw mcp serve   # stdio MCP server，Claude Code 在 .mcp.json 里注册
```

### 路径 3：ACP Agents（acpx 插件）—— OpenClaw 会话里直接跑 Claude Code harness

官方文档：[ACP agents](https://docs.openclaw.ai/tools/acp-agents) + [setup](https://docs.openclaw.ai/tools/acp-agents-setup)

- OpenClaw 通过 ACP（Agent Client Protocol）**拉起外部编码 harness**：Claude Code / Codex / Gemini CLI / Cursor / Copilot 等
- 技能在 OpenClaw 侧加载，harness 干编码活 → **OpenClaw 的技能体系直接对 Claude Code 生效**
- 安装：`openclaw plugins install @openclaw/acpx` + `openclaw config set plugins.entries.acpx.enabled true`
- 使用：`/acp spawn claude`（聊天里）或 `sessions_spawn({ runtime: "acp", agentId: "claude" })`（代码里）
- 前提：本机已装 Claude Code 且有 auth；`acp.allowedAgents` 需含 `claude`

---

## 四、技能互通实操（核心答案）

### 方向 A：OpenClaw 技能 → Claude Code 使用

**最简单：直接复制目录。**

```powershell
# 把 OpenClaw 全局技能复制为 Claude Code 个人技能
Copy-Item -Recurse "C:\...\state\skills\grill-me" "$env:USERPROFILE\.claude\skills\grill-me"
```

**已验证**：本机 `~/.claude/skills/grill-me` 与 OpenClaw 全局 `skills/grill-me` 同源同内容（frontmatter 完全一致），说明该路径已被实际使用且 Claude Code 正常加载。

**注意事项：**
1. 技能正文若引用 OpenClaw 特有工具（`exec`、`read`、`{baseDir}` 占位符、recognize.ps1 等），在 Claude Code 里需微调（Claude Code 有 Bash/Read 但无 `{baseDir}` 宏）
2. ClawHub 技能天然兼容——`clawhub install` 后复制到 `~/.claude/skills/` 即可被 Claude Code 用
3. 我们的八件套技能（workspace-initializer 等）正文大量依赖 OpenClaw 工作区约定，跨到 Claude Code 时正文需适配，但 frontmatter 不用动

### 方向 B：Claude Code 技能 → OpenClaw 使用

**官方推荐：`openclaw migrate claude`**（见路径 1），或手动复制目录到 `skills/`。

本机 `~/.claude/skills/` 现有 5 个技能：better-icons、find-skills、frontend-design、grill-me、skill-creator——其中 frontend-design、grill-me、skill-creator 与我们的全局技能同源，说明指挥官此前已做过双向同步。

---

## 五、本机环境现状（实测）

| 项目 | 状态 |
|---|---|
| Claude Code | ✅ v2.1.143（`D:\MySoft\nodejs\node_global\claude`） |
| `~/.claude/skills/` | ✅ 存在，5 个技能 |
| `~/.claude.json` | ✅ 存在（MCP 配置载体） |
| `~/.claude/CLAUDE.md` | ✅ 存在 |
| openclaw CLI | ⚠️ 不在 PATH（桌面版内嵌 runtime，命令不可直接调用） |
| acpx 插件 | ❌ 未安装（未走 ACP 路径） |

---

## 六、决策建议

1. **想要 Claude Code 用上我们的技能**：复制 + 微调正文即可，**无需任何配置**。建议按需挑选（grill-me / skill-creator / frontend-design 已就位）
2. **想要 OpenClaw 用 Claude Code 的技能**：`openclaw migrate claude --dry-run` 先预览（桌面版需找到 openclaw 可执行文件路径）
3. **想要 OpenClaw 会话里直接跑 Claude Code**（深度集成）：装 `@openclaw/acpx` 插件，适合「聊天指挥 + Claude Code 写代码」工作流
4. **不推荐**：为技能共享单独上 MCP serve（它是会话桥，不是技能桥）

---

## 七、信息来源（全部官方，2026-08-31 抓取）

- OpenClaw: [Skills](https://docs.openclaw.ai/tools/skills) / [Creating skills](https://docs.openclaw.ai/tools/creating-skills) / [Migrate CLI](https://docs.openclaw.ai/cli/migrate) / [Migrating from Claude](https://docs.openclaw.ai/install/migrating-claude) / [MCP](https://docs.openclaw.ai/mcp) / [ACP agents](https://docs.openclaw.ai/tools/acp-agents) / [ACP setup](https://docs.openclaw.ai/tools/acp-agents-setup) / [ClawHub](https://docs.openclaw.ai/clawhub) / [Skill format](https://docs.openclaw.ai/clawhub/skill-format)
- Claude Code: [Skills](https://code.claude.com/docs/en/skills) / [Settings reference](https://code.claude.com/docs/en/settings-reference)
- 标准: [agentskills.io](https://agentskills.io)（Specification：name/description 必填、目录结构、字段约束）
