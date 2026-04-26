# 章节卡片：16｜安全与隔离：当 Agent 暴露给真实消息渠道之后

## 核心问题

OpenClaw 的安全边界为什么比 CLI coding agent 更复杂？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-4-delivery-extension-safety`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/architecture.md`
- `~/workspace/openclaw/docs/concepts/session.md`
- `~/workspace/openclaw/src/gateway/*`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 的安全边界为什么比 CLI coding agent 更复杂？

**Claude Code 主要风险**：

```text
文件读写
bash 执行
MCP tool
本地项目边界
```

**OpenClaw 额外风险**：

```text
陌生 DM
群聊 prompt injection
channel/account 误路由
owner-only tools
memory 泄漏
cron/webhook 滥用
错误 delivery target
background task 越权
sandbox / non-main session 策略
```

**源码/文档锚点**：

```text
README.md
docs/channels/groups.md
docs/channels/discord.md
docs/automation/cron-jobs.md
src/agents/tools/cron-tool.ts
src/plugin-sdk/webhook-memory-guards.ts
src/agents/sandbox.ts
src/routing/session-key.ts
```

**文章结论**：

> Claude Code 的权限问题主要发生在工具调用前；OpenClaw 的权限问题从消息进入 Gateway 的那一刻就开始了。
```
