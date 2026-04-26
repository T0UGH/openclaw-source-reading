# 章节卡片：09｜Heartbeat：为什么 OpenClaw 会在没有用户输入时醒来

## 核心问题

Heartbeat 解决什么问题？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-3-time-automation`。

## 源码锚点

- `~/workspace/openclaw/docs/gateway/heartbeat.md`
- `~/workspace/openclaw/src/infra/heartbeat-runner.ts`
- `~/workspace/openclaw/src/auto-reply/heartbeat-filter.ts`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
**核心问题**：Heartbeat 解决什么问题？

**定义**：

```text
Heartbeat = periodic main-session agent turn
```

**关键行为**：

- 默认 every 30m；
- 默认 prompt 读取 HEARTBEAT.md；
- 没事回复 HEARTBEAT_OK；
- HEARTBEAT_OK 会被识别、剥离、必要时静默丢弃；
- heartbeat run 不创建 background task record；
- 可配置 target、last contact、lightContext、isolatedSession、activeHours；
- 适合 inbox/calendar/notification/checkup，不适合精确日报。

**源码/文档锚点**：

```text
docs/gateway/heartbeat.md
docs/automation/index.md
src/infra/heartbeat-runner.ts
src/infra/heartbeat-schedule.ts
src/infra/heartbeat-active-hours.ts
src/infra/heartbeat-wake.ts
src/auto-reply/heartbeat.ts
src/auto-reply/heartbeat-filter.ts
src/agents/heartbeat-system-prompt.ts
```

**文章结论**：

> Heartbeat 让 OpenClaw 有了低频、克制、可静默的在场感：没事不打扰，有事才提醒。
```
