# 章节卡片：10｜Cron：精确调度、隔离执行与结果投递

## 核心问题

Cron 和 Heartbeat 的边界是什么？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-3-time-automation`。

## 源码锚点

- `~/workspace/openclaw/docs/automation/cron-jobs.md`
- `~/workspace/openclaw/src/gateway/server-cron.ts`
- `~/workspace/openclaw/src/cron/isolated-agent/run.ts`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：Cron 和 Heartbeat 的边界是什么？

**对照表**：

| 维度 | Heartbeat | Cron |
|---|---|---|
| 时间 | 近似周期 | 精确 at / every / cron expression |
| 会话 | 主会话 turn | main / isolated / current / custom session |
| 任务记录 | 不创建 task record | 每次执行创建 background task record |
| 适合 | 轻量检查、提醒、状态感知 | 日报、提醒、后台分析、webhook 触发 |
| 输出 | 主会话内联或 last target | announce / webhook / none |

**Cron 流程**：

```text
Cron Job Definition
  -> CronService scheduler
  -> choose session target
      main: enqueue system event + heartbeat wake
      isolated: fresh cron:<jobId> agent turn
      current/custom: persistent context
  -> run agent
  -> message tool or fallback announce/webhook
  -> task ledger + run log + state update
```

**源码/文档锚点**：

```text
docs/automation/cron-jobs.md
docs/automation/index.md
src/gateway/server-cron.ts
src/cron/service.ts
src/cron/store.ts
src/cron/isolated-agent/run.ts
src/cron/isolated-agent/session.ts
src/cron/delivery.ts
src/cron/run-log.ts
src/agents/tools/cron-tool.ts
src/cli/cron-cli.ts
```

**文章结论**：

> Cron 是 OpenClaw 的精确时间轴：它让 agent 在指定时间运行，并把执行纳入任务、投递和审计系统。
```
