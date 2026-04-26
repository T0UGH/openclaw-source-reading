# 章节卡片：05｜Workspace Files：OpenClaw 的长期状态放在哪里

## 核心问题

OpenClaw 的 workspace 为什么不是普通 repo root？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-2-state-memory`。

## 源码锚点

- `~/workspace/openclaw/README.md`
- `~/workspace/openclaw/docs/concepts/multi-agent.md`
- `~/workspace/openclaw/docs/concepts/memory.md`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：OpenClaw 的 workspace 为什么不是普通 repo root？

**文件地图**：

```text
AGENTS.md
  长期操作规则、standing orders、系统级行为边界

USER.md
  用户身份、偏好、个人资料

MEMORY.md
  durable facts、偏好、决策、长期事实

memory/YYYY-MM-DD.md
  daily notes、短期运行上下文、每日观察

HEARTBEAT.md
  heartbeat 醒来时读取的周期检查清单

DREAMS.md
  dreaming sweep 和长期记忆晋升的 review 表面

sessions/*.jsonl
  会话、agent run、轨迹记录

cron/jobs.json
  cron job definitions

cron/jobs-state.json
  cron runtime execution state
```

**源码/文档锚点**：

```text
docs/concepts/memory.md
docs/gateway/heartbeat.md
docs/automation/cron-jobs.md
src/agents/workspace.ts
src/memory/root-memory-files.ts
src/config/sessions/*
src/cron/store.ts
```

**文章结论**：

> OpenClaw 的 workspace 是 agent 的长期生活空间：规则、身份、记忆、日程、会话和后台任务都在这里留下痕迹。
```
