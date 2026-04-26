# OpenClaw 源码阅读章节卡片索引

这些卡片用于 autopilot 写作时锁定每篇文章的角色、问题、源码锚点与图文要求。

- [00｜给 Claude Code 用户的 OpenClaw 概念迁移表](./00-concept-migration.md)
- [01｜为什么 OpenClaw 值得单独读：它不是一个只等输入的 Agent](./01-not-waiting-agent.md)
- [02｜Event Sources：OpenClaw 的 Agent Run 从哪里来](./02-event-sources.md)
- [03｜Gateway：为什么 OpenClaw 的入口不是 CLI](./03-gateway.md)
- [04｜Session Routing：真实世界的消息如何变成 Agent 上下文](./04-session-routing.md)
- [05｜Workspace Files：OpenClaw 的长期状态放在哪里](./05-workspace-files.md)
- [06｜Memory 总览：OpenClaw 如何让 Agent 真正“有记忆”](./06-memory-overview.md)
- [07｜Active Memory：为什么记忆不应该总靠主 Agent 自己想起来](./07-active-memory.md)
- [08｜Memory Flush 与 Dreaming：OpenClaw 如何防止上下文在压缩时丢失](./08-memory-flush-dreaming.md)
- [09｜Heartbeat：为什么 OpenClaw 会在没有用户输入时醒来](./09-heartbeat.md)
- [10｜Cron：精确调度、隔离执行与结果投递](./10-cron.md)
- [11｜Automation Layer：Heartbeat、Cron、Hooks、Tasks、Standing Orders 如何分工](./11-automation-layer.md)
- [12｜Background Tasks 与 Subagent：OpenClaw 如何记录 detached work](./12-background-tasks-subagent.md)
- [13｜Reply Shaping：为什么聊天渠道需要一层回复整形](./13-reply-shaping.md)
- [14｜Plugin 与 Channel：OpenClaw 如何扩展新的世界入口](./14-plugin-channel.md)
- [15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载](./15-control-plane-runtime-plane.md)
- [16｜安全与隔离：当 Agent 暴露给真实消息渠道之后](./16-safety-isolation.md)
