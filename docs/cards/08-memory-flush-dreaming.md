# 章节卡片：08｜Memory Flush 与 Dreaming：OpenClaw 如何防止上下文在压缩时丢失

## 核心问题

长对话压缩前，OpenClaw 如何避免重要信息消失？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-2-state-memory`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/dreaming.md`
- `~/workspace/openclaw/src/auto-reply/reply/memory-flush.ts`
- `~/workspace/openclaw/src/plugins/memory-state.ts`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：长对话压缩前，OpenClaw 如何避免重要信息消失？

**要点**：

- compaction 前可能还有重要事实没写入 memory；
- memory flush 是一次静默保存机会；
- flush plan 由 memory plugin capability 提供；
- dreaming 是后台整理和长期记忆晋升机制；
- DREAMS.md 是 human review surface。

**源码/文档锚点**：

```text
docs/concepts/memory.md
docs/concepts/dreaming.md
src/auto-reply/reply/memory-flush.ts
src/auto-reply/reply/agent-runner-memory.ts
src/plugins/memory-state.ts
src/commands/doctor-cron-dreaming-payload-migration.ts
```

**文章结论**：

> OpenClaw 把 compaction 前的上下文丢失风险，转化成一次 memory flush 机会；再用 dreaming 做更慢、更审慎的长期整理。
```
