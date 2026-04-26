# OpenClaw 源码阅读小书

> OpenClaw 是一个会记忆、会醒来、会定时执行、会跨渠道投递结果的个人 AI runtime。

这不是 OpenClaw 的功能清单，也不是“Claude Code 平替”说明。这个仓库把 OpenClaw 当作一个长期运行的个人 AI runtime 来读：真实世界事件先进入 Gateway，再经过 session routing、workspace / memory、heartbeat / cron、background tasks、reply shaping、plugin / channel 和安全边界，最后回到真实沟通渠道。

## 从哪里开始

正式正文入口：

- **[OpenClaw 源码阅读小书目录](docs/guidebook/README.md)**
- 建议第一篇从 [00｜给 Claude Code 用户的 OpenClaw 概念迁移表](docs/guidebook/volume-1-concept-boundary/00-concept-migration.md) 开始。

如果只想先抓住全书主线，可以按这条认知链读：

```text
外部事件 -> Gateway -> Session Routing -> Workspace / Memory
-> Heartbeat / Cron / Background Tasks -> Reply Shaping / Delivery
-> Plugin / Channel -> Control Plane / Runtime Plane -> Safety
```

## 这套小书回答什么

- 为什么 OpenClaw 不只是一个等用户 prompt 的 agent；
- 事件、时间、记忆和投递如何共同构成长期运行时；
- Memory / Heartbeat / Cron / Tasks 各自解决什么问题，边界在哪里；
- 模型输出为什么还要经过 Reply Shaping 和 Delivery；
- Plugin / Channel 为什么是外部世界能力面，而不只是 tool extension；
- 安全边界为什么从消息进入 Gateway 就开始，而不是只在 shell permission 出现。

## 正文目录

### 第一卷｜概念迁移与入口边界

- [00｜给 Claude Code 用户的 OpenClaw 概念迁移表](docs/guidebook/volume-1-concept-boundary/00-concept-migration.md)
- [01｜为什么 OpenClaw 值得单独读：它不是一个只等输入的 Agent](docs/guidebook/volume-1-concept-boundary/01-not-waiting-agent.md)
- [02｜Event Sources：OpenClaw 的 Agent Run 从哪里来](docs/guidebook/volume-1-concept-boundary/02-event-sources.md)
- [03｜Gateway：为什么 OpenClaw 的入口不是 CLI](docs/guidebook/volume-1-concept-boundary/03-gateway.md)
- [04｜Session Routing：真实世界的消息如何变成 Agent 上下文](docs/guidebook/volume-1-concept-boundary/04-session-routing.md)

### 第二卷｜长期状态与记忆

- [05｜Workspace Files：OpenClaw 的长期状态放在哪里](docs/guidebook/volume-2-state-memory/05-workspace-files.md)
- [06｜Memory 总览：OpenClaw 如何让 Agent 拥有可用记忆](docs/guidebook/volume-2-state-memory/06-memory-overview.md)
- [07｜Active Memory：为什么 OpenClaw 要在主回复前加一层 blocking memory pass](docs/guidebook/volume-2-state-memory/07-active-memory.md)
- [08｜Memory Flush 与 Dreaming：OpenClaw 如何防止上下文在压缩时丢失](docs/guidebook/volume-2-state-memory/08-memory-flush-dreaming.md)

### 第三卷｜时间、自动化与后台工作

- [09｜Heartbeat：为什么 OpenClaw 会在没有用户输入时醒来](docs/guidebook/volume-3-time-automation/09-heartbeat.md)
- [10｜Cron：精确调度、隔离执行与结果投递](docs/guidebook/volume-3-time-automation/10-cron.md)
- [11｜Automation Layer：Heartbeat、Cron、Hooks、Tasks、Standing Orders 如何分工](docs/guidebook/volume-3-time-automation/11-automation-layer.md)
- [12｜Background Tasks 与 Subagent：OpenClaw 如何记录 detached work](docs/guidebook/volume-3-time-automation/12-background-tasks-subagent.md)

### 第四卷｜投递、扩展与安全边界

- [13｜Reply Shaping：为什么聊天渠道需要一层回复整形](docs/guidebook/volume-4-delivery-extension-safety/13-reply-shaping.md)
- [14｜Plugin 与 Channel：OpenClaw 如何扩展新的世界入口](docs/guidebook/volume-4-delivery-extension-safety/14-plugin-channel.md)
- [15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载](docs/guidebook/volume-4-delivery-extension-safety/15-control-plane-runtime-plane.md)
- [16｜安全与隔离：当 Agent 暴露给真实消息渠道之后](docs/guidebook/volume-4-delivery-extension-safety/16-safety-isolation.md)

## 仓库结构

```text
docs/
  guidebook/                   # 正文入口：17 篇源码阅读文章
    README.md                  # 分卷目录与阅读路径
    volume-1-concept-boundary/
    volume-2-state-memory/
    volume-3-time-automation/
    volume-4-delivery-extension-safety/
  cards/                       # 写作卡片：每篇文章的问题、锚点和图文要求
  design/                      # 小书架构设计与历史版本
  notes/                       # 差异化调研笔记
  AUTOPILOT_STATUS.md          # 12 轮自动推进状态
```

## 当前状态

- 正文 00-16 共 17 篇已完成草稿。
- 每篇正文包含源码锚点、Mermaid 机制图、imagegen2 配图、`IMAGEGEN_PLACEHOLDER` 与可读性自检。
- 已完成全书可读性复审、Mermaid / imagegen 验收、源码锚点复核与写作教练复看。
- 当前已按 `claude-code-source-guide` 的 MkDocs / GitHub Pages 结构整理为公开站点。

## 支撑材料

- [写作卡片索引](docs/cards/README.md)：保留每篇文章的写作约束和源码锚点。
- [小书架构 v0.3](docs/design/book-architecture-v0.3.md)：当前结构的设计来源。
- [OpenClaw 差异化调研](docs/notes/2026-04-26-openclaw-differentiation-research.md)：早期定位和差异判断。

## 本地阅读

直接从 `docs/guidebook/README.md` 进入即可。

如果要预览 GitHub Pages 站点：

```bash
pip install -r requirements.txt
mkdocs serve
```

## GitHub Pages

公开站点地址：<https://t0ugh.github.io/openclaw-source-reading/>
