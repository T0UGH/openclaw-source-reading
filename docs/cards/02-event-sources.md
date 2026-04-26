# 章节卡片：02｜Event Sources：OpenClaw 的 Agent Run 从哪里来

## 核心问题

OpenClaw 不是只有用户 prompt，它有哪些事件来源？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-1-concept-boundary`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/architecture.md`
- `~/workspace/openclaw/docs/gateway/heartbeat.md`
- `~/workspace/openclaw/docs/automation/cron-jobs.md`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 不是只有用户 prompt，它有哪些事件来源？

**要点**：

```text
User message
Channel event
Heartbeat tick
Cron schedule
Webhook
Background task completion
Node/device event
UI/CLI command
```

**文章结构**：

1. 从 Claude Code 的 user prompt 心智模型说起；
2. OpenClaw 的外部事件来源；
3. 哪些事件触发 main session，哪些触发 isolated run；
4. 事件进入 Gateway 后如何进入 routing / agent / delivery。

**源码/文档锚点**：

```text
docs/concepts/architecture.md
docs/automation/index.md
docs/gateway/heartbeat.md
docs/automation/cron-jobs.md
src/gateway/*
src/infra/heartbeat-runner.ts
src/gateway/server-cron.ts
```

**文章结论**：

> OpenClaw 是 event-driven。用户 prompt 只是事件来源之一，不是全部。
```
