# 章节卡片：13｜Reply Shaping：为什么聊天渠道需要一层回复整形

## 核心问题

为什么模型输出不能直接等于聊天消息？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-4-delivery-extension-safety`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/agent-loop.md`
- `~/workspace/openclaw/src/auto-reply/reply/*`
- `~/workspace/openclaw/src/infra/outbound/*`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：为什么模型输出不能直接等于聊天消息？

**原因**：

真实渠道里有：

```text
assistant deltas
tool events
chat delta/final
NO_REPLY / no_reply
HEARTBEAT_OK
message tool duplicate
fallback announce
media / structured payload
channel-specific constraints
```

所以需要：

```text
model/tool stream
  -> chat delta/final
  -> duplicate suppression
  -> no_reply / heartbeat ack filtering
  -> fallback delivery
  -> channel adapter
```

**源码/文档锚点**：

```text
docs/concepts/agent-loop.md
src/auto-reply/reply/*
src/auto-reply/heartbeat-filter.ts
src/cron/isolated-agent/run.ts
src/infra/outbound/*
src/channels/*
```

**文章结论**：

> OpenClaw 的输出不是“模型吐字”，而是要被转译成真实渠道里的消息行为。
```
