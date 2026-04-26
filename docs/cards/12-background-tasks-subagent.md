# 章节卡片：12｜Background Tasks 与 Subagent：OpenClaw 如何记录 detached work

## 核心问题

OpenClaw 中哪些工作会脱离当前对话继续运行？如何被审计？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-3-time-automation`。

## 源码锚点

- `~/workspace/openclaw/docs/concepts/multi-agent.md`
- `~/workspace/openclaw/src/cron/isolated-agent/run.ts`
- `~/workspace/openclaw/src/agents/tools/AGENTS.md`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 中哪些工作会脱离当前对话继续运行？如何被审计？

**要点**：

- tasks 是 ledger，不是 scheduler；
- isolated cron、subagent、ACP run、CLI operations 都可以进入 task records；
- task reconciliation 处理 running/lost/completed 状态；
- cron runtime ownership 和 task ledger 之间有维护关系；
- subagent 和 multi-agent 不要混淆。

**源码/文档锚点**：

```text
docs/automation/tasks.md
docs/concepts/multi-agent.md
src/agents/subagent-spawn.ts
src/agents/tools/subagents-tool.ts
src/cron/service.ts
src/cron/isolated-agent/run.ts
```

**文章结论**：

> OpenClaw 需要 task ledger，是因为它不是只有同步回复；很多工作会在后台、隔离 session 或子 agent 中继续运行。
```
