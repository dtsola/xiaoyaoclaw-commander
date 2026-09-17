# ClawHub 安全检查核查与修复（2026-09-17）

> 执行人：天桐｜指令：指挥官「处理 OpenClaw Cross-Tool Commander 的问题」
> 命令：`clawhub skill verify xiaoyaoclaw-commander`（对象 v1.0.1）

---

## 1. 结论

`ok:false` / `decision:fail` / 原因码 `security.status_not_clean`；`security.status = suspicious`（confidence high）。
**共 17 条**：aig **2**（T09 / T07，均为 warning，Medium）+ skillspector **15**（HIGH 1 / MEDIUM 13 / LOW 1，风险分 84）。

LLM 判词：*"This skill is a cross-tool commander … 能让外部工具指挥真实 agent 与通道"* —— 核心担忧是**高权限动作 + 触发面过宽 + 缺少确认**。

---

## 2. 命中与修复对照

### 2.1 aig（2 条，**全是真问题**）

| 命中 | 位置 | 问题 | 修复 |
|---|---|---|---|
| **T09 warning** | `SKILL.md:138`、`:155` | **命令注入**：技能给出的命令示例把用户文本（任务描述 / 消息内容）放进 `-m "<...>"` 的 shell 字符串里，文本若含引号、反引号、`$()`、`;` 会被 shell 解释成命令 | 新增 **「安全执行约定」**章节：① 参数分开传、变量一律双引号、值用 `--flag=值` 形式 ② PowerShell 用**参数数组** `$argv=@(...); & $oc @argv` ③ **明令禁止** `eval` / `sh -c` / `Invoke-Expression`、字符串拼接、把用户文本当选项（以 `-` 开头会被解析成 flag）、把用户文本放进命令名/路径/通道名/agent id 这些**结构性位置** ④ 执行前**回显最终 argv** 自检<br>**实测**：`$argv=@('agents','list','--json'); & $OC @argv` 已在本机跑通返回 JSON |
| **T07 warning** | `SKILL.md:120-123` | **从用户可写目录盲搜可执行文件**（macOS 段用 `find "$HOME/Library/Application Support" …`）——可能执行到被替换/伪造的二进制 | **删掉 find 兜底**，改为「找不到就停下来问用户要路径，不自行搜索/下载/执行来路不明的文件」；并补**执行前校验**：候选必须位于已知安装目录或 PATH，且不可被同组/其他用户改写（`ls -ld` 无 `g+w`/`o+w`）；用户手工告知的路径**先回显确认**再执行 |

### 2.2 skillspector（15 条）

| 类别 | 条数 | 位置 | 修复 |
|---|---|---|---|
| **P2** Prompt Injection（HIGH） | 1 | `assets/readme/hero.svg:7` | SVG 注释被当作隐藏指令 → **移除全部 6 处注释** |
| **SQP-1** 触发面过宽 | 4 | `README.md:53/54`、`README.en.md:52`、`docs/DESIGN.md:140` | description + 新增 **「激活边界」**章节：仅在①用户明确要求「用外部工具驱动 OpenClaw」且点名目标 ②外部工具内明确要求 OpenClaw/小遥的某 agent 做事、经某通道发消息、查状态时激活；**不激活**：普通对话、问文档/配置/原理、宿主本身就是 OpenClaw、宽泛说法但没走 OpenClaw/没点名目标。README 中英补「什么时候不该触发」 |
| **SQP-2** 缺副作用警示 | 2 | `README.md:80`、`README.en.md:49` | README 中英顶部加 ⚠️ 醒目警示：**会真的产生外部效果**（`agent` 让真实 agent 干活；`message send` 真发给真人/真群）；并新增**读写分级表**（🟢 `health`/`agents list`/`sessions` 只读可直接跑 ｜ 🟠 `agent` 派任务先复述确认 ｜ 🔴 `message send` 必须先复述「通道 + 收件人 + 内容」并确认，含敏感信息时提醒脱敏） |
| **RA2** 会话持久化 | 1 | `README.en.md:17` | README 中英明确 **无持久化**：不建 cron、不起守护、不写启动脚本、不写跨会话状态文件；环境变量只作用于**当前 shell 会话**，不改任何配置文件 |
| **AS3** Agent Snooping（技能枚举） | 4 | `SKILL.md:21`（×2）、`docs/research-00:29`、`docs/research-report:101` | `SKILL.md:21` 改写为「把**本技能自己的目录**复制到目标工具的 Agent Skills 目录」并显式声明**只使用自身文件与 openclaw CLI，不读取、不枚举其他技能**；两份 docs 通过 `.clawhubignore` 排除出发布包 |
| **E1** Data Exfiltration | 1 | `docs/DESIGN.md:161` | 同属 docs → 排除出发布包（该处只是文档里提到的 URL） |
| **SQP-3** 语言中立 | 2 | `docs/research-00:1`、`assets/readme/hero.svg:13` | docs 排除；hero 副标题改**双语**（"跨工具指挥层 · Cross-tool command layer"）；SKILL.md / README 加**语言可选**声明 |

**包内容卫生**：新增 `.clawhubignore`，排除 `PROGRESS.md` 与 `docs/`（DESIGN + 两份调研）→ 发布包 **11 → 6 个文件**。（沿用 SEO 那轮的教训：扫描器会把文档当技能内容读。）

---

## 3. 验证

- `node --check` 无脚本可查（本技能为纯指令型，无脚本、无依赖、无网络访问）→ 验证落在**文本与行为约定**上
- SKILL.md 结构完整（257 行）：`激活边界` / `安全执行约定` / `读写分级` / `无持久化` / `不枚举其他技能` / `语言可选` **全部到位**
- README 中英：安全警示、无持久化、不该触发说明 **全部到位**
- **推荐命令写法实测跑通**：`$argv=@('agents','list','--json'); & $OC @argv` → 正常返回 agent 列表 JSON
- `hero.svg`：注释清空 → Chrome 渲染 26 KB PNG → 视觉模型复核：**双语副标题无裁切/溢出、整体排版无重叠**
- 发布包预览（复刻 CLI `listSkillFiles`）：**6 个文件**，`docs/` 与 `PROGRESS.md` 已排除，无内网 IP 字面量

---

## 4. 待办

- [ ] 发 **v1.0.2** → 等扫描 → 复扫核对 17 条（尤其两条 aig warning 是否清零）
- [ ] GitHub 推送（代理 22307 未监听 + 直连超时）

## 5. 原始证据

- `docs/evidence/verify-v1.0.1-2026-09-17.json`
