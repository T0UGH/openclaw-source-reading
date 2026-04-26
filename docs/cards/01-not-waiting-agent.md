# 章节卡片：01｜为什么 OpenClaw 值得单独读：它不是一个只等输入的 Agent

## 核心问题

OpenClaw 的独特性到底在哪里？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-1-concept-boundary`。

## 源码锚点

- `~/workspace/openclaw/README.md`
- `~/workspace/openclaw/docs/concepts/architecture.md`
- `~/workspace/openclaw/docs/automation/index.md`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 的独特性到底在哪里？

**要点**：

- 普通 coding agent 是 request/response；
- OpenClaw 有长期进程、长期 workspace、长期 memory、周期 heartbeat、精确 cron、跨渠道 delivery；
- 它不是更强的 CLI，而是把 agent 放进真实世界事件流。

**源码/文档锚点**：

```text
README.md
docs/concepts/architecture.md
docs/automation/index.md
```

**文章结论**：

> OpenClaw 的独特性不在于一次 agent run，而在于它让 agent 长期活在 Gateway 里。
```
