# 章节卡片：14｜Plugin 与 Channel：OpenClaw 如何扩展新的世界入口

## 核心问题

OpenClaw plugin 和普通 coding agent extension 有什么不同？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-4-delivery-extension-safety`。

## 源码锚点

- `~/workspace/openclaw/docs/plugins/architecture.md`
- `~/workspace/openclaw/src/plugins/*`
- `~/workspace/openclaw/src/channels/*`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw plugin 和普通 coding agent extension 有什么不同？

**要点**：

- plugin 可以提供 channel、provider、memory、media、search、hook、tool；
- channel plugin 把真实世界消息转进 Gateway；
- provider plugin 提供模型/推理能力；
- memory plugin 提供 memory runtime；
- manifest-first 控制面让 discovery/config validation 不必加载完整 runtime。

**源码/文档锚点**：

```text
docs/plugins/architecture.md
docs/plugins/manifest.md
docs/plugins/sdk-overview.md
src/plugins/*
src/plugin-sdk/*
src/channels/*
extensions/*
```

**文章结论**：

> OpenClaw 的插件系统不是给 agent 加几个工具，而是给长期运行的 Gateway 增加新的外部世界入口和能力平面。
```
