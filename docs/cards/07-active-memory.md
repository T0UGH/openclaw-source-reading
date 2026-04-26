# 章节卡片：07｜Active Memory：为什么记忆不应该总靠主 Agent 自己想起来

## 核心问题

为什么 OpenClaw 要在主回复前加一层 blocking memory pass？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-2-state-memory`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/active-memory.md`
- `~/workspace/openclaw/src/plugins/memory-runtime.ts`
- `~/workspace/openclaw/src/auto-reply/reply/agent-runner-memory.ts`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：为什么 OpenClaw 要在主回复前加一层 blocking memory pass？

**运行形态**：

```text
User Message
  -> Build Memory Query
  -> Active Memory Blocking Sub-Agent
  -> memory_search / memory_get
  -> NONE or compact memory summary
  -> hidden active_memory_plugin context
  -> Main Reply
```

**关键边界**：

- 默认只跑 direct chat；
- 不跑 headless one-shot；
- 不跑 heartbeat/background run；
- 不跑 generic internal agent-command；
- 不跑 sub-agent/internal helper execution；
- memory sub-agent tool surface 很窄，只允许 memory_search / memory_get。

**源码/文档锚点**：

```text
docs/concepts/active-memory.md
src/plugins/memory-runtime.ts
src/plugins/memory-state.ts
src/agents/memory-search.ts
src/plugin-sdk/memory-host-search.ts
```

**文章结论**：

> Active Memory 把“是否需要召回长期记忆”从主回复里拆出来，让相关记忆在回复生成前自然出现。
```
