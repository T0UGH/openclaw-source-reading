# 章节卡片：11｜Automation Layer：Heartbeat、Cron、Hooks、Tasks、Standing Orders 如何分工

## 核心问题

OpenClaw 的自动化层由哪些机制组成？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-3-time-automation`。

## 源码锚点

- `~/workspace/openclaw/docs/automation/index.md`
- `~/workspace/openclaw/docs/automation/cron-vs-heartbeat.md`
- `~/workspace/openclaw/src/agents/tools/cron-tool.ts`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 的自动化层由哪些机制组成？

**机制分工**：

```text
Heartbeat
  周期性轻量检查，不创建 task record

Cron
  精确调度和后台运行，每次创建 task record

Hooks
  响应生命周期、消息流、工具调用、gateway startup 等事件

Tasks
  detached work ledger，记录 ACP runs、subagents、isolated cron、CLI operations

Task Flow
  durable multi-step orchestration

Standing Orders
  长期授权、规则和持续任务边界
```

**源码/文档锚点**：

```text
docs/automation/index.md
docs/automation/tasks.md
docs/automation/hooks.md
docs/automation/taskflow.md
docs/automation/standing-orders.md
```

**文章结论**：

> OpenClaw 的自动化不是一个“定时任务功能”，而是让 agent 拥有长期行为边界的操作系统层。
```
