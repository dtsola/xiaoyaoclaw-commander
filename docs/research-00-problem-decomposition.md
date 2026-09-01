# 00_问题拆解.md

## Step 0: 问题类型判断

- **类型**：知识梳理型 + 决策支持型（混合）
- **核心任务**：弄清「Claude Code 如何连接/使用 OpenClaw 的技能（skills）」——机制 + 路径 + 落地
- **侧重维度**：机制差异、连接方式、适用边界、实操可行性

## Step 0.5: 时效敏感性

- **级别**：🔴 极高（AI/大模型领域）
- **时间窗口**：3-6 个月（资料以 2026 年为主，2025 年资料仅作历史参考）
- **强制规则**：
  1. 官方源优先（OpenClaw docs / Anthropic docs / GitHub 官方仓库）
  2. 版本号强制标注（禁止「最新版本支持」）
  3. 关键信息至少 2 个独立来源确认
  4. 搜索带时间约束
  5. 协议名称必搜（MCP、ACP、Agent Skills、SKILL.md、ClawHub）

## Step 1: 问题拆解与边界界定

**研究对象**：
- 主体 A：OpenClaw（开源 AI 助手运行时，本机 state/agents/tiantong 即其工作区，技能在 state/skills/）
- 主体 B：Claude Code（Anthropic 官方 CLI 编程代理）
- 连接物：Skills（OpenClaw 技能 = SKILL.md 目录；Claude Code 技能 = Agent Skills）

**子问题**：
1. **Q1 概念对齐**：OpenClaw 的技能机制是什么（SKILL.md 结构、ClawHub、如何被 agent 加载）？
2. **Q2 概念对齐**：Claude Code 的技能机制是什么（Agent Skills 目录、.claude/skills、如何被加载）？两个技能体系有什么异同？
3. **Q3 连接路径**：Claude Code ↔ OpenClaw 之间有哪些官方/社区已实现的集成方式？
   - 路径候选：MCP server（OpenClaw 暴露 skills 给 Claude Code？）、ACP、CLI 调用、文件共享、ClawHub 技能直接被 Claude Code 用？
4. **Q4 实操落地**：本机（Windows + OpenClaw 桌面版 + 已装 Claude Code）如何配置实现？有无现成教程/插件/issue？

**边界**：
- ❌ 不调研 Claude Code 本身的所有功能（只关注 skills/连接能力）
- ❌ 不调研 OpenClaw 的全部架构（只关注 skills 机制 + 对外连接面）
- ✅ 聚焦「技能如何跨系统流动」

→ 下一步：Step 2 资料分层与搜索
