---
title: OpenClaw 源码阅读小书
date: 2026-04-26
---

# OpenClaw 源码阅读小书

> 一套从 **个人 AI runtime** 视角阅读 OpenClaw 的中文源码导读小书。

这里是公开站点首页。当前正式阅读入口已经切到 **guidebook**：17 篇正文按 4 卷组织，围绕 Gateway、Memory、Heartbeat、Cron、Background Tasks、Reply Shaping、Plugin / Channel 与安全边界展开。

## 从哪里开始

### 第一次进入这个项目

建议直接从这里开始：

- [开始阅读｜小书总览](./guidebook/README.md)
- [00｜给 Claude Code 用户的 OpenClaw 概念迁移表](./guidebook/volume-1-concept-boundary/00-concept-migration.md)
- [01｜为什么 OpenClaw 值得单独读：它不是一个只等输入的 Agent](./guidebook/volume-1-concept-boundary/01-not-waiting-agent.md)

### 如果你想先抓整套结构

可以直接按四卷导读进入：

1. [第一卷｜概念迁移与入口边界](./guidebook/volume-1-concept-boundary/00-concept-migration.md)
2. [第二卷｜长期状态与记忆](./guidebook/volume-2-state-memory/05-workspace-files.md)
3. [第三卷｜时间、自动化与后台工作](./guidebook/volume-3-time-automation/09-heartbeat.md)
4. [第四卷｜投递、扩展与安全边界](./guidebook/volume-4-delivery-extension-safety/13-reply-shaping.md)

## 这套小书在讲什么

这套小书不是带你“逛源码目录”，而是在回答一串更关键的问题：

- 为什么 OpenClaw 不只是一个等用户 prompt 的 agent；
- 真实世界事件如何进入 Gateway，并被路由成可工作的 agent session；
- workspace、memory、active memory、memory flush / dreaming 如何维持长期状态；
- heartbeat、cron、hooks、tasks、standing orders 如何组成时间与自动化层；
- 模型输出为什么还要经过 reply shaping、过滤、去重和 delivery；
- plugin / channel 为什么是 runtime capability plane，而不只是工具扩展；
- 安全边界为什么从消息进入 Gateway 就开始。

## 一句话主线

```text
外部事件 -> Gateway -> Session Routing -> Workspace / Memory
-> Heartbeat / Cron / Background Tasks -> Reply Shaping / Delivery
-> Plugin / Channel -> Control Plane / Runtime Plane -> Safety
```

## 四卷各自回答什么

- **第一卷**：先把 Claude Code / coding agent 的旧心智模型迁移到 OpenClaw 的运行时视角。
- **第二卷**：看长期状态、workspace、memory 与上下文压缩如何支撑持续工作。
- **第三卷**：看 heartbeat、cron、automation layer、background tasks 如何让系统在没有即时用户输入时继续运行。
- **第四卷**：看真实渠道投递、插件扩展、控制面 / 运行面与安全隔离如何收口。

## 推荐阅读路线

### 路线 A：第一次系统阅读

- [开始阅读｜小书总览](./guidebook/README.md)
- [00｜概念迁移表](./guidebook/volume-1-concept-boundary/00-concept-migration.md)
- 然后按 01 到 16 顺序继续

### 路线 B：只想抓时间与自动化

- [09｜Heartbeat：为什么 OpenClaw 会在没有用户输入时醒来](./guidebook/volume-3-time-automation/09-heartbeat.md)
- [10｜Cron：精确调度、隔离执行与结果投递](./guidebook/volume-3-time-automation/10-cron.md)
- [11｜Automation Layer：Heartbeat、Cron、Hooks、Tasks、Standing Orders 如何分工](./guidebook/volume-3-time-automation/11-automation-layer.md)
- [12｜Background Tasks 与 Subagent：OpenClaw 如何记录 detached work](./guidebook/volume-3-time-automation/12-background-tasks-subagent.md)

### 路线 C：只想抓投递、扩展与安全

- [13｜Reply Shaping：为什么聊天渠道需要一层回复整形](./guidebook/volume-4-delivery-extension-safety/13-reply-shaping.md)
- [14｜Plugin 与 Channel：OpenClaw 如何扩展新的世界入口](./guidebook/volume-4-delivery-extension-safety/14-plugin-channel.md)
- [15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载](./guidebook/volume-4-delivery-extension-safety/15-control-plane-runtime-plane.md)
- [16｜安全与隔离：当 Agent 暴露给真实消息渠道之后](./guidebook/volume-4-delivery-extension-safety/16-safety-isolation.md)

## 当前入口说明

- `guidebook/` 是当前正式阅读入口。
- `cards/`、`design/`、`notes/` 是写作支撑材料，不进入站点主导航。
- 站点导航默认按四卷 17 篇正文展开。
