---
title: OpenClaw 源码阅读小书
date: 2026-04-26
---

# OpenClaw 源码阅读小书

> 从 **个人 AI runtime** 视角阅读 OpenClaw：它如何记忆、醒来、定时执行，并把结果投递回真实渠道。

<div class="hero-actions" markdown="1">

[开始阅读](./guidebook/README.md){ .md-button .md-button--primary }
[从 00 概念迁移开始](./guidebook/volume-1-concept-boundary/00-concept-migration.md){ .md-button }

</div>

## 你会读到什么

这套小书不是功能清单，也不是 Claude Code 平替说明。它沿着一条运行时链路读 OpenClaw：

```text
外部事件 -> Gateway -> Session Routing -> Workspace / Memory
-> Heartbeat / Cron / Background Tasks -> Reply Shaping / Delivery
-> Plugin / Channel -> Control Plane / Runtime Plane -> Safety
```

读完以后，你应该能回答四个问题：

- OpenClaw 为什么不是只等 prompt 的 agent？
- Gateway、Session Routing、Workspace / Memory 怎样把真实世界事件变成可持续工作的上下文？
- Heartbeat、Cron、Background Tasks 怎样让系统在没有即时用户输入时继续运行？
- Reply Shaping、Plugin / Channel、Control Plane / Runtime Plane 和 Safety 怎样把结果安全投回真实渠道？

## 四卷入口

| 卷 | 先读 | 解决的问题 |
|---|---|---|
| 第一卷｜概念迁移与入口边界 | [00｜概念迁移表](./guidebook/volume-1-concept-boundary/00-concept-migration.md) | 从 coding agent 心智切到个人 AI runtime 心智 |
| 第二卷｜长期状态与记忆 | [05｜Workspace Files](./guidebook/volume-2-state-memory/05-workspace-files.md) | 长期状态、memory、active recall、flush / dreaming 如何工作 |
| 第三卷｜时间、自动化与后台工作 | [09｜Heartbeat](./guidebook/volume-3-time-automation/09-heartbeat.md) | 周期醒来、精确定时、后台任务和自动化边界如何分工 |
| 第四卷｜投递、扩展与安全边界 | [13｜Reply Shaping](./guidebook/volume-4-delivery-extension-safety/13-reply-shaping.md) | 模型输出如何变成真实渠道里的安全投递 |

## 推荐路线

### 第一次系统阅读

[开始阅读｜小书总览](./guidebook/README.md) → [00｜概念迁移表](./guidebook/volume-1-concept-boundary/00-concept-migration.md) → 按 01 到 16 顺序继续。

### 只想抓时间与自动化

[09｜Heartbeat](./guidebook/volume-3-time-automation/09-heartbeat.md) → [10｜Cron](./guidebook/volume-3-time-automation/10-cron.md) → [11｜Automation Layer](./guidebook/volume-3-time-automation/11-automation-layer.md) → [12｜Background Tasks](./guidebook/volume-3-time-automation/12-background-tasks-subagent.md)

### 只想抓投递、扩展与安全

[13｜Reply Shaping](./guidebook/volume-4-delivery-extension-safety/13-reply-shaping.md) → [14｜Plugin 与 Channel](./guidebook/volume-4-delivery-extension-safety/14-plugin-channel.md) → [15｜Control Plane vs Runtime Plane](./guidebook/volume-4-delivery-extension-safety/15-control-plane-runtime-plane.md) → [16｜安全与隔离](./guidebook/volume-4-delivery-extension-safety/16-safety-isolation.md)

## 当前入口说明

- `guidebook/` 是当前正式阅读入口。
- `cards/`、`design/`、`notes/` 是写作支撑材料，不进入站点主导航。
- 站点导航按四卷 17 篇正文展开。
