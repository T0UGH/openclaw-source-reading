# 章节卡片：03｜Gateway：为什么 OpenClaw 的入口不是 CLI

## 核心问题

为什么 Gateway 是 OpenClaw 的第一运行边界？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-1-concept-boundary`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/architecture.md`
- `~/workspace/openclaw/src/gateway/*`
- `~/workspace/openclaw/docs/concepts/agent-loop.md`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：为什么 Gateway 是 OpenClaw 的第一运行边界？

**要点**：

- Gateway 是长期进程；
- 多渠道消息、webhook、control plane、node 都接入 Gateway；
- CLI 只是控制入口，不是唯一产品形态；
- Gateway 负责把真实世界事件转成内部运行时事件。

**源码/文档锚点**：

```text
docs/concepts/architecture.md
src/gateway/*
src/channels/*
src/plugin-sdk/*
```

**文章结论**：

> OpenClaw 的入口不是终端，而是 Gateway。agent run 是 Gateway 对外部事件的响应。
```
