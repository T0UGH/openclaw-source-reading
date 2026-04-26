# 章节卡片：00｜给 Claude Code 用户的 OpenClaw 概念迁移表

## 核心问题

哪些 Claude Code 心智模型不能直接套到 OpenClaw？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-1-concept-boundary`。

## 源码锚点

- `docs/design/book-architecture-v0.3.md`
- `~/workspace/openclaw/README.md`
- `~/workspace/openclaw/docs/concepts/architecture.md`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：哪些 Claude Code 心智模型不能直接套到 OpenClaw？

**要点**：

- Claude Code 的中心是 CLI / 项目目录 / 开发任务；
- OpenClaw 的中心是 Gateway / workspace / long-running runtime；
- session、memory、hooks、permission、notification 在 OpenClaw 里都变了含义。

**源码/文档锚点**：

```text
README.md
docs/concepts/architecture.md
docs/automation/index.md
docs/concepts/memory.md
docs/gateway/heartbeat.md
docs/automation/cron-jobs.md
```

**文章结论**：

> 读 OpenClaw 前，要先从“开发任务 agent”切换到“个人运行时 agent”。否则后面所有机制都会被误解成 Claude Code 的变体。
```
