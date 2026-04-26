---
title: "05｜Workspace Files：OpenClaw 的长期状态放在哪里"
status: draft-card
chapter: "05"
slug: "workspace-files"
---

# 05｜Workspace Files：OpenClaw 的长期状态放在哪里

> 本篇状态：章节卡片 / 正文待写。后续写作必须先重新阅读下方源码锚点，再展开机制判断。

## 读者问题

OpenClaw 的 workspace 为什么不是普通 repo root？

## 本篇先给的结论

待正文写作时补充。结论必须回到这条主线：OpenClaw 不是只等用户输入的 coding agent，而是由 Gateway、Session、Workspace、Memory、Heartbeat、Cron、Delivery 等机制组合起来的个人 AI 运行时。

## 先看一张机制图

这张图先作为本篇的低分辨率机制草图，后续正文写作时需要根据源码锚点细化。

```mermaid
flowchart TB
  WS[Workspace Files] --> Base[长期事实/规则]
  Base --> Search[Memory Search]
  Search --> Active[Active Memory]
  Active --> Run[Agent Run]
  Run --> Flush[Memory Flush / Dreaming]
  Flush --> WS
```

读这张图时，先按这个顺序看：
- 先看本篇讨论的入口或触发条件；
- 再看它进入 OpenClaw 运行时之后由哪一层接住；
- 最后看它如何影响长期状态、Agent Run 或真实渠道投递。

<!-- IMAGEGEN_PLACEHOLDER:
title: 05｜Workspace Files：OpenClaw 的长期状态放在哪里 机制图
type: architecture
purpose: 用一张正式技术架构图解释“OpenClaw 的 workspace 为什么不是普通 repo root？”
prompt_seed: 生成一张 16:9 中文技术架构图，主题是 OpenClaw 源码阅读第 05 篇：Workspace Files：OpenClaw 的长期状态放在哪里。图中只保留少量标签，突出层次、边界和主链路；高对比、无 logo、无水印，不要装饰性插画。
asset_target: docs/assets/05-workspace-files-imagegen.png
status: pending
-->

## 源码锚点

- `~/workspace/openclaw/README.md`
- `~/workspace/openclaw/docs/concepts/multi-agent.md`
- `~/workspace/openclaw/docs/concepts/memory.md`

## 写作边界

- 不引入无关项目叙事。
- 不写成 Claude Code 平替；Claude Code 只作为读者迁移背景。
- 不做目录游览；要回答读者问题。
- 术语第一次出现时要用中文说人话。

## 正文大纲草案

1. 从读者问题进入；
2. 先给本篇结论；
3. 对照普通 coding agent / Claude Code 心智模型的失效点；
4. 按源码锚点解释 3-5 个机制层；
5. 区分相邻机制边界；
6. 留下一个能接住下一篇的 takeaway。
