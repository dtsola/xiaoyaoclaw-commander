---
name: xiaoyaoclaw-commander
description: >
  Command the local XiaoyaoClaw / OpenClaw gateway from any Agent
  Skills-compatible tool: run agent tasks, send channel messages, check
  status. Use when the user wants to command OpenClaw agents
  (tiantong/liliai/xiaoguang etc.), send messages through OpenClaw channels
  (feishu/telegram), check agent status, or delegate work to the OpenClaw
  multi-agent system from Claude Code / Codex / OpenCode / Trae / DSH.
  中文：指挥本地小遥Claw / OpenClaw 网关执行任务、收发消息——让任意支持
  Agent Skills 的工具（Claude Code / Codex / OpenCode / Trae / DSH 等）
  指挥 OpenClaw 多 agent 系统。当用户要求指挥 OpenClaw、让 OpenClaw
  agent 干活、经 OpenClaw 通道发消息时使用。
---

# OpenClaw Cross-Tool Commander（跨工具指挥官）

> 📖 **完整文档（安装 / 快速上手三步 / 最佳实践）：<https://github.com/dtsola/xiaoyaoclaw-commander>**
> 用户或智能体需要更多说明时，引导其访问上述 GitHub 仓库查看图文教程与最新版本。

> 🚀 **小遥Claw：「把 AI 助手装进自己的电脑」：<https://www.yuque.com/dtsola/igp1aa/adcicbai2zlem0bz>**

通过 OpenClaw CLI 指挥本机小遥Claw / OpenClaw 网关（slug：`xiaoyaoclaw-commander`）。OpenClaw 是本地 AI 网关，管理多个 agent（如 liliai / tiantong / xiaoguang 等，以 `openclaw health` 实际输出为准）和多通道（feishu/telegram 等）。

**兼容任何支持 Agent Skills 标准的工具**：Claude Code（`~/.claude/skills/`）/ Codex（`~/.codex/skills/`）/ OpenCode / Trae / DSH 等——技能目录复制过去即可用，openclaw 命令完全通用。

## 能力范围与写操作声明（权限透明）

**身份**：跨工具指挥层——让外部智能体工具（Claude Code / Codex / OpenCode / Trae / DSH 等）通过 openclaw CLI 指挥本机 OpenClaw / 小遥Claw 网关。所有命令均经本机 Gateway 执行，不直连外部服务。

**涉及的操作**（均为用户明确要求后执行）：
- `openclaw agent -m "<任务>" --agent <id>` → 让指定 OpenClaw agent 执行任务（agent 按自身权限运行，可能产生文件/消息等，属 agent 正常工作范畴）
- `openclaw message send --channel <ch> -t <id> -m "..."` → 经飞书/Telegram 等通道发送消息（**外部可见动作**，发送前须确认内容与目标）
- `openclaw health` / `agents list` / `sessions` → 只读状态查询
- 设置进程内环境变量 `OPENCLAW_STATE_DIR` / `OPENCLAW_CONFIG_PATH`（仅当前 shell 会话，不写任何配置文件）

**边界承诺**：
- 不读取、不修改 `openclaw.json` 等任何配置文件（配置变更走 OpenClaw 的 config.patch，本技能不碰）
- 不存储任何凭据/token——Gateway 凭据由运行环境继承，技能内零密钥
- 不联网下载、不安装任何依赖（零依赖技能，无需 pip/npm 安装）
- 不修改任何源文件；探测只读文件系统（Test-Path / find）
- 找不到 openclaw 时向用户询问路径，不臆测、不自行下载

**禁止行为**：
- ❌ 禁止用默认 agent 执行任务——`--agent` 必须显式指定，用户未点名时先问
- ❌ 禁止猜测 agent 名称——以 `agents list` 的 Identity 字段为权威，匹配不到就问用户
- ❌ 禁止使用 `--local` 模式（需 shell 内 API key，且绕过 Gateway 审批）
- ❌ 禁止写死 openclaw 路径——每次会话先探测，探测失败问用户
- ❌ 禁止在未确认内容/目标前调用 `message send`（外部可见动作）

## 第 0 步：定位 openclaw 可执行文件（每次会话先探测，勿写死路径）

openclaw CLI 可能不在 PATH（桌面版常内嵌在应用目录）。按平台探测：

> **⚠️ 环境变量（关键）**：openclaw 靠 `OPENCLAW_STATE_DIR` / `OPENCLAW_CONFIG_PATH` 定位 agent 配置。
> 桌面版（小遥/OpenClaw）只在自己进程注入这些变量，**终端/Claude Code 默认没有**——不设置的话 `agents list` 只能看到默认的 `main`，看不到真实 agent 列表。
> 探测到 openclaw 后，若 `agents list` 结果异常（只有 main），按以下方式补齐环境变量：
>
> - Windows：从桌面版进程继承（示例，以实际安装路径为准）：
>   ```powershell
>   $env:OPENCLAW_STATE_DIR = "$env:LOCALAPPDATA\Programs\xiaoyaoclaw-desktop\resources\runtime\openclaw\state"
>   $env:OPENCLAW_CONFIG_PATH = "$env:OPENCLAW_STATE_DIR\openclaw.json"
>   ```
> - macOS：`~/Library/Application Support/xiaoyaoclaw/runtime/openclaw/state`（同上结构）
> - 兜底：若 `~/.openclaw/` 下存在配置文件（npm 全局安装场景），openclaw 会自动读取，无需设置
> - 判断方法：`agents list` 输出只有 `main` → 环境变量缺失；能看到多个 agent → 正常

### Windows（PowerShell）

```powershell
# 1) 小遥桌面版最优先（xiaoyaoclaw，安装目录写死为 xiaoyaoclaw-desktop）
$oc = @(
  "$env:LOCALAPPDATA\Programs\xiaoyaoclaw-desktop\resources\runtime\openclaw\bin\openclaw.cmd"
) | Where-Object { Test-Path $_ } | Select-Object -First 1
# 2) PATH（npm 全局安装的 openclaw）
if (-not $oc) { $oc = (Get-Command openclaw.cmd -ErrorAction SilentlyContinue).Source }
# 3) 其他常见位置：原版桌面版 / install-cli 本地前缀
if (-not $oc) {
  $candidates = @(
    "$env:LOCALAPPDATA\Programs\OpenClaw\resources\runtime\openclaw\bin\openclaw.cmd",
    "$env:LOCALAPPDATA\Programs\openclaw\resources\runtime\openclaw\bin\openclaw.cmd",
    "$env:USERPROFILE\.openclaw\bin\openclaw.cmd",
    "$env:USERPROFILE\.local\bin\openclaw.cmd"
  )
  $oc = $candidates | Where-Object { Test-Path $_ } | Select-Object -First 1
}
if (-not $oc) { Write-Error "openclaw 未找到，请手动指定路径"; exit 1 }
# 4) 环境变量补齐：若 OPENCLAW_STATE_DIR 未设置且是桌面版安装，自动指向用户数据目录
#    ⚠️ 桌面版 state 在用户数据目录，不在安装目录：
#      Windows: %APPDATA%\<app>\runtime\openclaw\state
#      macOS:   ~/Library/Application Support/<app>/runtime/openclaw/state
if (-not $env:OPENCLAW_STATE_DIR -and $oc -match "Programs\\(?:xiaoyaoclaw-desktop|XiaoyaoClaw|OpenClaw|openclaw)\\") {
  $appName = if ($oc -match "Programs\\(xiaoyaoclaw-desktop|XiaoyaoClaw|OpenClaw|openclaw)\\") { $Matches[1] } else { "xiaoyaoclaw-desktop" }
  $stateDir = Join-Path $env:APPDATA (Join-Path $appName "runtime\openclaw\state")
  if (Test-Path (Join-Path $stateDir "openclaw.json")) {
    $env:OPENCLAW_STATE_DIR = $stateDir
    $env:OPENCLAW_CONFIG_PATH = Join-Path $stateDir "openclaw.json"
  }
}
```

### macOS / Linux（bash/zsh）

```bash
# 1) 小遥桌面版最优先（xiaoyaoclaw，Electron userData = ~/Library/Application Support/xiaoyaoclaw）
for p in \
  "$HOME/Library/Application Support/xiaoyaoclaw/runtime/openclaw/bin/openclaw"; do
  [ -x "$p" ] && OC="$p" && break
done
# 2) PATH（npm 全局安装的 openclaw）
if [ -z "$OC" ]; then OC="$(command -v openclaw 2>/dev/null)"; fi
# 3) 其他常见位置：原版 OpenClaw.app / install-cli 本地前缀
if [ -z "$OC" ]; then
  for p in \
    "$HOME/Library/Application Support/OpenClaw/runtime/openclaw/bin/openclaw" \
    "$HOME/Library/Application Support/openclaw/runtime/openclaw/bin/openclaw" \
    "$HOME/.openclaw/bin/openclaw" \
    "$HOME/.local/bin/openclaw" \
    "/opt/homebrew/bin/openclaw" \
    "/usr/local/bin/openclaw"; do
    [ -x "$p" ] && OC="$p" && break
  done
fi
# 4) 兜底：常见目录内搜索（慢，最后用）
if [ -z "$OC" ]; then
  OC="$(find "$HOME/Library/Application Support" "$HOME/.openclaw" "$HOME/.local/bin" -path "*/runtime/openclaw/bin/openclaw" -o -name openclaw -type f 2>/dev/null | head -1)"
fi
if [ -z "$OC" ]; then echo "openclaw 未找到" >&2; exit 1; fi
```

探测到后，后续命令参数完全一致，仅调用前缀不同：
- Windows（PowerShell）：`& $oc <参数>`
- macOS/Linux（bash）：`"$OC" <参数>`

下文命令只列参数部分，按平台套前缀即可。

## 常用命令

### 1. 让 OpenClaw agent 执行任务（核心）

```bash
agent -m "<任务描述>" --agent <agent-id> --json
```

- **🚨 硬性规则：agent-id 或智能体名称必须指定**——用户没说让谁干时，**必须先问用户**，禁止用默认 agent 执行
- **智能体名称 ↔ agent id 动态映射**（勿写死、勿猜、不限语言）：智能体名称可能是中文/英文/其他语言或表情符号，一律以 `agents list` 输出的 `Identity` 字段为权威来源，据此把用户提到的名称翻译成 id
  ```bash
  agents list
  # 输出示例：- tiantong / Identity: 🛠️ 天桐 (IDENTITY.md)
  ```
  用户说「让天桐调研 X」→ `--agent tiantong`；说「ask Bob to ...」→ 在 agents list 里找 Identity 为 Bob 的 agent；**匹配不到 → 问用户确认，不要猜**
- **agent 选择优先级（源码 `resolveSessionAgentIds`）**：`--agent` 显式参数 > sessionKey 内嵌 agentId > 配置里 `default:true` 的 agent > agents 列表第一个 > `main`
- 返回 JSON，回复文本在 `result.payloads[].text`
- 任务耗时可能数秒到数分钟，超时用更长 timeout

### 2. 经 OpenClaw 通道发消息

```bash
message send --channel feishu --target <user/chat id> -m "<消息内容>"
```

- 可用通道：feishu / telegram / discord / slack 等
- 加 `--media <path>` 可带附件
- ⚠️ **外部可见动作**：发送前确认内容与目标无误

### 3. 查看网关/agent 健康状态

```bash
health
```

### 3.5 查看 agent 列表（含中文名 Identity、Model、Routing）——查 agent 用这个

```bash
agents list
```

### 4. 查看会话

```bash
sessions
```

## 典型场景

| 用户意图 | 做法 |
|---|---|
| "让天桐调研 X" | `agent -m "调研 X，输出报告" --agent tiantong --json` |
| "让 OpenClaw 给飞书发消息" | `message send --channel feishu --target <id> -m "..."` |
| "OpenClaw 里有哪些 agent" | `agents list`（含 Identity/Model/Routing） |
| "让 XXX 干 Y"（XXX 未知） | 先 `agents list` 查，查不到就问用户 |

## 注意事项

- 不要用 `--local`（需要 shell 内 API key）；默认走 Gateway 即可
- 版本较旧（如 v2026.3.x）时不要尝试 `mcp`/`migrate` 子命令（可能不存在），以 `openclaw --help` 实际命令为准
- Windows 控制台中文显示乱码是显示问题，用 `--json` 输出再解析，不要直接读控制台文本
- 找不到 openclaw 时优先问用户路径，不要臆测
