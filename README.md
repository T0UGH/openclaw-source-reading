# OpenClaw 源码阅读

这是一组围绕 OpenClaw 源码阅读的小书设计与后续写作材料。

## 当前定位

OpenClaw 不是另一个只围绕代码仓库运行的 coding agent，也不是 Claude Code 的平替。它更像一个：

> 会记忆、会醒来、会定时执行、会跨渠道投递结果的个人 AI 运行时。

本项目后续写作重点从 OpenClaw 自身架构出发，围绕以下主线展开：

- Gateway 与事件来源
- Session Routing
- Workspace Files 与长期状态
- Memory / Active Memory / Memory Flush / Dreaming
- Heartbeat
- Cron / Tasks / Automation
- Reply Shaping 与 Delivery
- Plugin / Channel / Control Plane vs Runtime Plane
- 安全与隔离

## 文档结构

```text
docs/
  design/
    book-architecture-v0.3.md   # 完整版小书架构设计
    book-design-v0.2.md         # 加入 Memory / Heartbeat / Cron 的设计版
    book-design-v0.1.md         # 早期设计版，仅作历史参考
  notes/
    2026-04-26-openclaw-differentiation-research.md
```

## 当前首选目录

```text
00 给 Claude Code 用户的 OpenClaw 概念迁移表
01 为什么 OpenClaw 值得单独读：它不是一个只等输入的 Agent
02 Event Sources：OpenClaw 的 Agent Run 从哪里来
03 Gateway：为什么 OpenClaw 的入口不是 CLI
04 Session Routing：真实世界的消息如何变成 Agent 上下文
05 Workspace Files：OpenClaw 的长期状态放在哪里
06 Memory 总览：OpenClaw 如何让 Agent 真正“有记忆”
07 Active Memory：为什么记忆不应该总靠主 Agent 自己想起来
08 Memory Flush 与 Dreaming：OpenClaw 如何防止上下文在压缩时丢失
09 Heartbeat：为什么 OpenClaw 会在没有用户输入时醒来
10 Cron：精确调度、隔离执行与结果投递
11 Automation Layer：Heartbeat、Cron、Hooks、Tasks、Standing Orders 如何分工
12 Background Tasks 与 Subagent：OpenClaw 如何记录 detached work
13 Reply Shaping：为什么聊天渠道需要一层回复整形
14 Plugin 与 Channel：OpenClaw 如何扩展新的世界入口
15 Control Plane vs Runtime Plane：为什么插件不能一上来就全加载
16 安全与隔离：当 Agent 暴露给真实消息渠道之后
```

## 重要边界

- 不把 OpenClaw 与 Pi 建立关系。
- 不使用“Pi 宿主化 / SDK integration / 嵌入 Pi agent loop”叙事。
- 面向已经了解 Claude Code / Codex / 一般 coding agent 的读者，但不让 Claude Code 框架主导 OpenClaw 的解释。
