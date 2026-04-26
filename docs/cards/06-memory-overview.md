# 章节卡片：06｜Memory 总览：OpenClaw 如何让 Agent 真正“有记忆”

## 核心问题

OpenClaw 的 memory 由哪些层组成？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-2-state-memory`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/memory.md`
- `~/workspace/openclaw/docs/concepts/active-memory.md`
- `~/workspace/openclaw/src/plugins/memory-state.ts`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 的 memory 由哪些层组成？

**分层**：

```text
File layer
  MEMORY.md / memory/YYYY-MM-DD.md / DREAMS.md

Access layer
  memory_search / memory_get

Backend layer
  builtin / QMD / Honcho

Prompt layer
  memory prompt section / active memory hidden context

Lifecycle layer
  memory flush before compaction / dreaming background consolidation

Knowledge layer
  memory-wiki
```

**源码/文档锚点**：

```text
docs/concepts/memory.md
docs/concepts/memory-search.md
docs/concepts/memory-builtin.md
docs/concepts/memory-qmd.md
docs/concepts/memory-honcho.md
docs/plugins/memory-wiki.md
src/plugins/memory-state.ts
src/plugins/memory-runtime.ts
src/plugin-sdk/memory-*.ts
src/memory/root-memory-files.ts
```

**文章结论**：

> OpenClaw 的 memory 不是模型里的隐藏状态，而是一套文件优先、插件托管、可检索、可刷新、可后台整理的长期上下文系统。
```
