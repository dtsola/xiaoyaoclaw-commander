# xiaoyaoclaw-commander 设计方案

> 状态：待指挥官确认 · 日期：2026-09-01
> 定位：小遥生态九件套第九件——**跨工具指挥层**，让任意支持 Agent Skills 标准的工具指挥本机小遥Claw / OpenClaw 多 agent 系统。

---

## 1. 项目定位

**一句话：** 一个 Agent Skills 标准技能，通过 openclaw CLI 指挥本机小遥Claw / OpenClaw 网关——执行 agent 任务、通道收发消息、状态查询。

**与小遥生态的关系（九件套收官）：**

| 件 | 项目 | 职责 | 类比 |
|---|---|---|---|
| 1 | xiaoyaoclaw-workspace-initializer | 家：目录规范 | 房子 |
| 2 | xiaoyaoclaw-memory-distill | 内容：记忆蒸馏 | 管家 |
| 3 | xiaoyaoclaw-task-progress-tracker | 状态：进度管理 | 账本 |
| 4 | xiaoyaoclaw-kb-retriever | 知识：本地检索 | 书房 |
| 5 | xiaoyaoclaw-workspace-auditor | 健康：工作区体检 | 体检医生 |
| 6 | xiaoyaoclaw-web-clipper | 输入：网页剪藏 | 采买 |
| 7 | xiaoyaoclaw-agent-orchestrator | 协作：多 agent 编排 | 调度室 |
| 8 | xiaoyaoclaw-usage-report | 度量：用量报告 | 财务 |
| 9 | **xiaoyaoclaw-commander** | **指挥：跨工具指挥层** | **对讲机** |

**继承的体系铁律：**
- 路径一律按 WORKSPACE.md 走（技能路径冲突仲裁原则）
- 配置修改只用 config.patch，禁 config.apply（多 agent 安全）
- ClawHub 全流程确认制（提交/重发/版本递增先经指挥官确认）
- 反馈至上：任务无论成败必须汇报

---

## 2. 核心问题（为什么需要它）

终端/外部工具直接调 openclaw 有三个障碍：

1. **二进制不在 PATH**：小遥Claw 桌面版把 openclaw 内嵌在应用目录（`resources/runtime/openclaw/bin/`），终端敲 `openclaw` 找不到命令
2. **环境变量缺失**：agent 配置靠 `OPENCLAW_STATE_DIR` / `OPENCLAW_CONFIG_PATH` 定位，桌面版只在自身进程注入 → 终端里 `agents list` 只剩 `main`，看不到 7 个 agent
3. **agent 选择陷阱**：不指定 `--agent` 时按 `default:true` > agents[0] > main 兜底，可能让任务跑到错误 agent 上

**方案：** 技能内动态探测可执行文件 + 自动补齐环境变量 + 强制显式指定 agent。

---

## 3. 兼容范围（标识，不泛化）

**关键决策（指挥官定案 2026-09-01）：不需要泛化，只需要标识。**

Agent Skills 是开放标准（SKILL.md + name/description frontmatter），所有支持工具加载机制同构；openclaw 命令本身通用。因此技能**不为每个工具写专属逻辑**，只在描述与 README 中标识支持范围：

| 工具 | 技能目录 | 说明 |
|---|---|---|
| Claude Code | `~/.claude/skills/` | 已实测通过（Bash 调 openclaw CLI，614ms 链路通） |
| Codex | `~/.codex/skills/` | Agent Skills 标准目录 |
| OpenCode | 工具文档 | 支持 Agent Skills |
| Trae | 工具文档 | 支持 Agent Skills |
| DSH | 工具文档 | 支持 Agent Skills |
| 其他 | 对应技能目录 | 任意 Agent Skills 兼容工具 |

> 技能正文只做「openclaw 可执行文件探测」（Windows/macOS/Linux 多级探测 + 环境变量补齐），这部分与宿主工具无关，天然通用。

---

## 4. 文件结构

```
xiaoyaoclaw-commander/
├── SKILL.md                    # 技能主体（探测 + 环境变量补齐 + 命令手册）
├── README.md / README.en.md    # 中英双语 README
├── LICENSE                     # MIT
├── assets/readme/
│   ├── hero.svg                # README 封面图
│   └── community-qr.png        # 交流群二维码（发布时补）
└── docs/
    ├── DESIGN.md               # 本方案文档
    ├── research-report-claude-code-openclaw.md   # 调研报告
    └── research-00-problem-decomposition.md      # 问题拆解
```

---

## 5. SKILL.md 工作流（Step 0-3）

### Step 0: 定位 openclaw 可执行文件（每次会话先探测，勿写死路径）
- **Windows（PowerShell）**：小遥桌面版（`%LOCALAPPDATA%\Programs\xiaoyaoclaw-desktop\`，安装目录源码写死）→ PATH（npm 全局）→ 原版桌面版 / install-cli 本地前缀
- **macOS / Linux（bash）**：`~/Library/Application Support/xiaoyaoclaw/`（Electron userData 源码 `app.setName("xiaoyaoclaw")` 强制）→ PATH → OpenClaw.app / `~/.openclaw` / `~/.local/bin` → find 兜底
- 探测后仅调用前缀不同（PowerShell `& $oc` / bash `"$OC"`），命令参数完全一致

### Step 0.5: 环境变量补齐（关键）
- openclaw 靠 `OPENCLAW_STATE_DIR` / `OPENCLAW_CONFIG_PATH` 定位 agent 配置
- 桌面版安装场景自动推导 state 目录（Windows: `%APPDATA%\<app>\runtime\openclaw\state`；macOS: `~/Library/Application Support/<app>/runtime/openclaw/state`）
- 判断方法：`agents list` 只有 `main` → 环境变量缺失；能看到多个 agent → 正常
- npm 全局安装场景（`~/.openclaw/` 有配置）自动读取，无需设置

### Step 1: 让 agent 执行任务（核心）
```bash
agent -m "<任务描述>" --agent <agent-id> --json
```
- **硬性规则：`--agent` 必须显式指定**——用户没说让谁干时先问，禁止默认
- **agent 名称 ↔ id 动态映射**：以 `agents list` 的 Identity 字段为权威（中文/英文/emoji 不限），匹配不到问用户不猜
- agent 选择优先级（源码 resolveSessionAgentIds）：`--agent` > sessionKey 内嵌 agentId > default:true > agents[0] > main
- 返回 JSON，回复文本在 `result.payloads[].text`

### Step 2: 经通道发消息
```bash
message send --channel feishu --target <id> -m "<内容>"
```
- 可用通道：feishu / telegram / discord / slack 等；`--media <path>` 带附件

### Step 3: 状态查询
```bash
health          # 网关/agent 健康
agents list     # agent 列表（Identity/Model/Routing）
sessions        # 会话列表
```

### 注意事项（写入 SKILL.md）
- 不用 `--local`（需 shell 内 API key），默认走 Gateway
- 旧版本（v2026.3.x）无 `mcp`/`migrate` 子命令，以 `--help` 实际输出为准
- Windows 控制台中文乱码是显示问题，用 `--json` 输出再解析
- 找不到 openclaw 优先问用户路径，不臆测

---

## 6. 安全红线

1. **禁止默认 agent**：用户未点名时先问，绝不静默 fallback（多 agent 任务串错人的代价高）
2. **不猜 agent 名称**：以 Identity 字段为权威，匹配不到就问
3. **不写死路径**：探测失败时问用户，不臆测安装位置
4. **不改 openclaw.json**：技能只读网关状态、下发任务，配置变更走 config.patch
5. **不泄露 token**：技能内不存储任何凭据（Gateway token 由环境继承）

---

## 7. 触发方式

| 方式 | 说明 |
|---|---|
| 宿主工具调用 | 技能被 Claude Code / Codex 等工具按 Agent Skills 标准自动加载，用户自然语言触发 |
| 手动复制 | 复制技能目录到任意 Agent Skills 工具的技能目录即可用 |

---

## 8. 开发与发布计划

1. ~~建项目目录 + 进度卡~~（2026-09-01 已完成）
2. ~~SKILL.md 命名统一 + 多工具标识~~（2026-09-01 已完成）
3. ~~README.md / README.en.md 双语~~（2026-09-01 已完成）
4. 写 docs/DESIGN.md（本文档）
5. 写 hero.svg（assets/readme/）+ LICENSE(MIT)
6. git init（分支 main）→ GitHub 建仓 `dtsola/xiaoyaoclaw-commander` → push（代理 22307）
7. GitHub About 设置（中英 description + topics）
8. 同步全局技能 `state/skills/xiaoyaoclaw-commander/`（当前工作区即可用）
9. 八件套 README 互链（九件套闭环）
10. ClawHub 提交（走确认制，指挥官确认后操作）
11. memory/ 日志记录

**发布安全（吸取教训）：**
- push 前 `git rev-parse --git-dir` 确认仓库边界，`git status --short` 确认 staged 范围
- PowerShell 捕获 curl 输出用 GBK 解码破坏 UTF-8 → `cmd /c "curl ... > file"` 原样落盘再解析；传 JSON 用 `--data-binary @file`
- PATCH 后 GET 确认生效；git init 后 `git branch -M main`
- 推送 GitHub 走代理 `http://127.0.0.1:22307`（抽风时 SOCKS5 兜底）

---

## 9. 风险与注意点

- **环境变量推导依赖安装路径匹配**：桌面版目录名变了（如未来版本改名）推导会失效 → 探测失败时保留手动设置入口，问用户
- **openclaw 版本差异**：命令输出格式随版本演进 → 用 `--json` 解析 + `--help` 兜底
- **多工具标识 ≠ 多工具测试**：目前仅 Claude Code 实测，其余工具按标准兼容但未逐一验证 → README 注明实测范围
- **不替代 orchestrator**：commander 是指挥入口（外部工具 → 网关），orchestrator 是内部编排（网关内多 agent 拆解分发），职责不重叠

---

*待确认项：① 本方案整体是否 OK；② hero.svg 风格（对齐 usage-report 暗色终端风）；③ 确认后按 §8 计划发布。*
