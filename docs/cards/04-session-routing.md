# 章节卡片：04｜Session Routing：真实世界的消息如何变成 Agent 上下文

## 核心问题

OpenClaw 的 session 和普通 CLI session 有什么不同？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-1-concept-boundary`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/session.md`
- `~/workspace/openclaw/src/routing/session-key.ts`
- `~/workspace/openclaw/src/config/sessions/*`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 的 session 和普通 CLI session 有什么不同？

**要点**：

- DM、group、channel、topic、cron、webhook 对应不同 session 策略；
- sessionKey 是真实世界通信关系到内部状态的映射；
- session routing 同时影响 memory、workspace、工具权限、delivery target。

**源码/文档锚点**：

```text
docs/concepts/session.md
src/routing/session-key.ts
src/config/sessions/*
src/auto-reply/reply/*
```

**文章结论**：

> OpenClaw 的 session 不是聊天历史 ID，而是外部关系、agent identity 和 runtime state 的路由键。
```
