# OpenClaw 源码阅读小书架构设计 v0.3 完整版

> 面向读者：已经比较了解 Claude Code / Codex / 一般 coding agent 的工程师。  
> 目标：不是写一组零散源码笔记，而是设计一本能讲清 OpenClaw 独特架构的小书。  
> 核心修正：不把 OpenClaw 与 Pi 建立关系；从 OpenClaw 自身的 Gateway、Memory、Heartbeat、Cron、Session、Plugin、Channel、Automation 设计出发。

---

## 0. 一句话定位

**OpenClaw 是一个会记忆、会醒来、会定时执行、会跨渠道投递结果的个人 AI 运行时。**

它不是另一个只围绕代码仓库运行的 coding agent，也不是 Claude Code 的平替。它的核心差异不是“能不能调用工具”，而是：

```text
普通 coding agent：
  等待用户输入 -> 读上下文 -> 调工具 -> 输出结果

OpenClaw：
  长期运行 Gateway
    -> 接收消息 / webhook / heartbeat / cron / UI / device event
    -> 路由到 agent + session + workspace
    -> memory 召回或写入长期上下文
    -> agent run 执行回复、任务或自动化
    -> delivery / reply shaping 决定如何送回真实渠道
    -> session / memory / task / cron state 留在系统里继续生长
```

这本小书要讲的不是“OpenClaw 怎么实现一个 agent loop”，而是：

> **OpenClaw 如何把 agent 从一次性对话工具，设计成一个长期在线的个人 AI 操作系统雏形。**

---

## 1. 写作总原则

### 1.1 不写什么

这本书需要避免三个误区：

1. **不写成 Claude Code 对照说明书**  
   Claude Code 是读者背景，但不能成为 OpenClaw 的解释框架。

2. **不写成 Gateway 插件目录说明**  
   Gateway 很重要，但如果只讲 Channel / Plugin / Reply Shaping，会漏掉 OpenClaw 的长期生命体机制。

3. **不写成工具调用源码拆解**  
   OpenClaw 的独特性不是工具列表，而是 event source、session routing、memory、heartbeat、cron、delivery 的组合。

### 1.2 必须反复强调什么

全书反复打三个判断：

```text
1. OpenClaw 是 event-driven，不只是 prompt-driven。
2. OpenClaw 的 workspace 是 agent 的长期生活空间，不只是 repo root。
3. OpenClaw 的核心能力来自 Memory + Heartbeat + Cron + Gateway 的组合。
```

### 1.3 写作风格

- 先讲读者已经熟悉的 Claude Code / coding agent 心智模型；
- 再指出这个心智模型在 OpenClaw 里哪里失效；
- 最后回到源码，解释 OpenClaw 为什么这样设计。

每篇文章尽量遵循：

```text
读者已有模型
  -> OpenClaw 差异
  -> 源码入口
  -> 设计意图
  -> 和普通 coding agent 的分界线
```

---

## 2. 给 Claude Code 读者的概念迁移表

这一节建议作为第 00 篇，也可以作为全书前言。

| Claude Code / Coding Agent 概念 | OpenClaw 对应概念 | 关键差异 |
|---|---|---|
| CLI 会话 | routed session / agent session | OpenClaw session 来自消息渠道、用户、群组、cron、webhook、agentId 等路由上下文 |
| 项目目录 | agent workspace | workspace 不只是代码目录，也是 memory、heartbeat、standing orders、sessions 的长期状态容器 |
| CLAUDE.md | AGENTS.md / USER.md / HEARTBEAT.md / MEMORY.md | OpenClaw 把操作规则、用户资料、周期检查、长期记忆拆成不同文件 |
| 用户 prompt | event source | OpenClaw 的输入不只来自用户，也来自 heartbeat、cron、webhook、channel event、device node |
| agent run | Gateway-triggered run | run 是 Gateway 对事件的一种响应，不是唯一入口 |
| compaction | compaction + memory flush | OpenClaw 在压缩前会尝试把重要上下文写入 memory |
| memory | memory files + search + active memory + dreaming | OpenClaw memory 是文件优先、插件托管、可检索、可主动召回、可后台整理的系统 |
| notification | delivery target / announce / channel reply | 输出要进入真实渠道，所以有投递、去重、静默、fallback 等策略 |
| cron 外部脚本 | Gateway cron | OpenClaw cron 是 Gateway 内建 scheduler，会创建 background task，并能投递结果 |
| hooks | lifecycle hooks / plugin hooks / webhook | OpenClaw hooks 覆盖生命周期、消息流、工具调用、外部事件 |
| MCP / tool extension | plugin / channel / provider / memory / node surface | OpenClaw 插件不只是工具扩展，也可以是新渠道、新模型、新记忆后端、新媒体能力 |
| 权限审批 | runtime boundary | OpenClaw 的边界从消息进入 Gateway 就开始，包括身份、渠道、session、memory、cron、delivery、tool 权限 |

一句话总结：

> **Claude Code 的中心是开发任务；OpenClaw 的中心是长期在线的个人 agent runtime。**

---

## 3. 全书主线图

建议全书使用这张总图作为认知地图：

```text
External World
  ├─ Chat Channels: Telegram / Discord / Feishu / Slack / WhatsApp / ...
  ├─ UI / CLI
  ├─ Webhooks
  ├─ Cron Scheduler
  ├─ Heartbeat Timer
  └─ Device / Node Events
        |
        v
Gateway
  ├─ Channel Adapter
  ├─ Auth / Pairing / Owner Boundary
  ├─ Session Routing
  ├─ Agent Selection
  ├─ Plugin Runtime
  └─ Delivery Runtime
        |
        v
Agent Workspace
  ├─ AGENTS.md / USER.md
  ├─ MEMORY.md
  ├─ memory/YYYY-MM-DD.md
  ├─ HEARTBEAT.md
  ├─ DREAMS.md
  ├─ sessions/*.jsonl
  └─ cron/jobs.json + jobs-state.json
        |
        v
Agent Run
  ├─ Active Memory
  ├─ Main Reply / Task Execution
  ├─ Tool Calls
  ├─ Memory Flush
  ├─ Compaction
  └─ Subagent / Background Task
        |
        v
Output
  ├─ chat delta / final
  ├─ NO_REPLY / HEARTBEAT_OK filtering
  ├─ duplicate suppression
  ├─ fallback announce
  ├─ webhook delivery
  └─ task/run log
```

这张图的核心是：OpenClaw 不是一条单线 agent loop，而是一个围绕 Gateway 和 workspace 展开的长期运行系统。

---

## 4. 全书结构：四卷十五篇

## 第一卷：概念迁移与运行边界

第一卷解决“了解 Claude Code 的人，如何正确进入 OpenClaw”。

---

### 00｜给 Claude Code 用户的 OpenClaw 概念迁移表

**核心问题**：哪些 Claude Code 心智模型不能直接套到 OpenClaw？

**要点**：

- Claude Code 的中心是 CLI / 项目目录 / 开发任务；
- OpenClaw 的中心是 Gateway / workspace / long-running runtime；
- session、memory、hooks、permission、notification 在 OpenClaw 里都变了含义。

**源码/文档锚点**：

```text
README.md
docs/concepts/architecture.md
docs/automation/index.md
docs/concepts/memory.md
docs/gateway/heartbeat.md
docs/automation/cron-jobs.md
```

**文章结论**：

> 读 OpenClaw 前，要先从“开发任务 agent”切换到“个人运行时 agent”。否则后面所有机制都会被误解成 Claude Code 的变体。

---

### 01｜为什么 OpenClaw 值得单独读：它不是一个只等输入的 Agent

**核心问题**：OpenClaw 的独特性到底在哪里？

**要点**：

- 普通 coding agent 是 request/response；
- OpenClaw 有长期进程、长期 workspace、长期 memory、周期 heartbeat、精确 cron、跨渠道 delivery；
- 它不是更强的 CLI，而是把 agent 放进真实世界事件流。

**源码/文档锚点**：

```text
README.md
docs/concepts/architecture.md
docs/automation/index.md
```

**文章结论**：

> OpenClaw 的独特性不在于一次 agent run，而在于它让 agent 长期活在 Gateway 里。

---

### 02｜Event Sources：OpenClaw 的 Agent Run 从哪里来

**核心问题**：OpenClaw 不是只有用户 prompt，它有哪些事件来源？

**要点**：

```text
User message
Channel event
Heartbeat tick
Cron schedule
Webhook
Background task completion
Node/device event
UI/CLI command
```

**文章结构**：

1. 从 Claude Code 的 user prompt 心智模型说起；
2. OpenClaw 的外部事件来源；
3. 哪些事件触发 main session，哪些触发 isolated run；
4. 事件进入 Gateway 后如何进入 routing / agent / delivery。

**源码/文档锚点**：

```text
docs/concepts/architecture.md
docs/automation/index.md
docs/gateway/heartbeat.md
docs/automation/cron-jobs.md
src/gateway/*
src/infra/heartbeat-runner.ts
src/gateway/server-cron.ts
```

**文章结论**：

> OpenClaw 是 event-driven。用户 prompt 只是事件来源之一，不是全部。

---

### 03｜Gateway：为什么 OpenClaw 的入口不是 CLI

**核心问题**：为什么 Gateway 是 OpenClaw 的第一运行边界？

**要点**：

- Gateway 是长期进程；
- 多渠道消息、webhook、control plane、node 都接入 Gateway；
- CLI 只是控制入口，不是唯一产品形态；
- Gateway 负责把真实世界事件转成内部运行时事件。

**源码/文档锚点**：

```text
docs/concepts/architecture.md
src/gateway/*
src/channels/*
src/plugin-sdk/*
```

**文章结论**：

> OpenClaw 的入口不是终端，而是 Gateway。agent run 是 Gateway 对外部事件的响应。

---

### 04｜Session Routing：真实世界的消息如何变成 Agent 上下文

**核心问题**：OpenClaw 的 session 和普通 CLI session 有什么不同？

**要点**：

- DM、group、channel、topic、cron、webhook 对应不同 session 策略；
- sessionKey 是真实世界通信关系到内部状态的映射；
- session routing 同时影响 memory、workspace、工具权限、delivery target。

**源码/文档锚点**：

```text
docs/concepts/session.md
src/routing/session-key.ts
src/config/sessions/*
src/auto-reply/reply/*
```

**文章结论**：

> OpenClaw 的 session 不是聊天历史 ID，而是外部关系、agent identity 和 runtime state 的路由键。

---

## 第二卷：长期状态与记忆系统

第二卷是全书核心之一，讲 OpenClaw 如何让 agent 有长期状态。

---

### 05｜Workspace Files：OpenClaw 的长期状态放在哪里

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

---

### 06｜Memory 总览：OpenClaw 如何让 Agent 真正“有记忆”

**核心问题**：OpenClaw 的 memory 由哪些层组成？

**分层**：

```text
File layer
  MEMORY.md / memory/YYYY-MM-DD.md / DREAMS.md

Access layer
  memory_search / memory_get

Backend layer
  builtin / QMD / Honcho

Prompt layer
  memory prompt section / active memory hidden context

Lifecycle layer
  memory flush before compaction / dreaming background consolidation

Knowledge layer
  memory-wiki
```

**源码/文档锚点**：

```text
docs/concepts/memory.md
docs/concepts/memory-search.md
docs/concepts/memory-builtin.md
docs/concepts/memory-qmd.md
docs/concepts/memory-honcho.md
docs/plugins/memory-wiki.md
src/plugins/memory-state.ts
src/plugins/memory-runtime.ts
src/plugin-sdk/memory-*.ts
src/memory/root-memory-files.ts
```

**文章结论**：

> OpenClaw 的 memory 不是模型里的隐藏状态，而是一套文件优先、插件托管、可检索、可刷新、可后台整理的长期上下文系统。

---

### 07｜Active Memory：为什么记忆不应该总靠主 Agent 自己想起来

**核心问题**：为什么 OpenClaw 要在主回复前加一层 blocking memory pass？

**运行形态**：

```text
User Message
  -> Build Memory Query
  -> Active Memory Blocking Sub-Agent
  -> memory_search / memory_get
  -> NONE or compact memory summary
  -> hidden active_memory_plugin context
  -> Main Reply
```

**关键边界**：

- 默认只跑 direct chat；
- 不跑 headless one-shot；
- 不跑 heartbeat/background run；
- 不跑 generic internal agent-command；
- 不跑 sub-agent/internal helper execution；
- memory sub-agent tool surface 很窄，只允许 memory_search / memory_get。

**源码/文档锚点**：

```text
docs/concepts/active-memory.md
src/plugins/memory-runtime.ts
src/plugins/memory-state.ts
src/agents/memory-search.ts
src/plugin-sdk/memory-host-search.ts
```

**文章结论**：

> Active Memory 把“是否需要召回长期记忆”从主回复里拆出来，让相关记忆在回复生成前自然出现。

---

### 08｜Memory Flush 与 Dreaming：OpenClaw 如何防止上下文在压缩时丢失

**核心问题**：长对话压缩前，OpenClaw 如何避免重要信息消失？

**要点**：

- compaction 前可能还有重要事实没写入 memory；
- memory flush 是一次静默保存机会；
- flush plan 由 memory plugin capability 提供；
- dreaming 是后台整理和长期记忆晋升机制；
- DREAMS.md 是 human review surface。

**源码/文档锚点**：

```text
docs/concepts/memory.md
docs/concepts/dreaming.md
src/auto-reply/reply/memory-flush.ts
src/auto-reply/reply/agent-runner-memory.ts
src/plugins/memory-state.ts
src/commands/doctor-cron-dreaming-payload-migration.ts
```

**文章结论**：

> OpenClaw 把 compaction 前的上下文丢失风险，转化成一次 memory flush 机会；再用 dreaming 做更慢、更审慎的长期整理。

---

## 第三卷：主动性与自动化

第三卷讲 OpenClaw 如何从被动回复变成主动运行。

---

### 09｜Heartbeat：为什么 OpenClaw 会在没有用户输入时醒来

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

---

### 10｜Cron：精确调度、隔离执行与结果投递

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

---

### 11｜Automation Layer：Heartbeat、Cron、Hooks、Tasks、Standing Orders 如何分工

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

---

### 12｜Background Tasks 与 Subagent：OpenClaw 如何记录 detached work

**核心问题**：OpenClaw 中哪些工作会脱离当前对话继续运行？如何被审计？

**要点**：

- tasks 是 ledger，不是 scheduler；
- isolated cron、subagent、ACP run、CLI operations 都可以进入 task records；
- task reconciliation 处理 running/lost/completed 状态；
- cron runtime ownership 和 task ledger 之间有维护关系；
- subagent 和 multi-agent 不要混淆。

**源码/文档锚点**：

```text
docs/automation/tasks.md
docs/concepts/multi-agent.md
src/agents/subagent-spawn.ts
src/agents/tools/subagents-tool.ts
src/cron/service.ts
src/cron/isolated-agent/run.ts
```

**文章结论**：

> OpenClaw 需要 task ledger，是因为它不是只有同步回复；很多工作会在后台、隔离 session 或子 agent 中继续运行。

---

## 第四卷：渠道、插件、投递与安全

第四卷讲 OpenClaw 如何把长期 agent runtime 接到真实世界。

---

### 13｜Reply Shaping：为什么聊天渠道需要一层回复整形

**核心问题**：为什么模型输出不能直接等于聊天消息？

**原因**：

真实渠道里有：

```text
assistant deltas
tool events
chat delta/final
NO_REPLY / no_reply
HEARTBEAT_OK
message tool duplicate
fallback announce
media / structured payload
channel-specific constraints
```

所以需要：

```text
model/tool stream
  -> chat delta/final
  -> duplicate suppression
  -> no_reply / heartbeat ack filtering
  -> fallback delivery
  -> channel adapter
```

**源码/文档锚点**：

```text
docs/concepts/agent-loop.md
src/auto-reply/reply/*
src/auto-reply/heartbeat-filter.ts
src/cron/isolated-agent/run.ts
src/infra/outbound/*
src/channels/*
```

**文章结论**：

> OpenClaw 的输出不是“模型吐字”，而是要被转译成真实渠道里的消息行为。

---

### 14｜Plugin 与 Channel：OpenClaw 如何扩展新的世界入口

**核心问题**：OpenClaw plugin 和普通 coding agent extension 有什么不同？

**要点**：

- plugin 可以提供 channel、provider、memory、media、search、hook、tool；
- channel plugin 把真实世界消息转进 Gateway；
- provider plugin 提供模型/推理能力；
- memory plugin 提供 memory runtime；
- manifest-first 控制面让 discovery/config validation 不必加载完整 runtime。

**源码/文档锚点**：

```text
docs/plugins/architecture.md
docs/plugins/manifest.md
docs/plugins/sdk-overview.md
src/plugins/*
src/plugin-sdk/*
src/channels/*
extensions/*
```

**文章结论**：

> OpenClaw 的插件系统不是给 agent 加几个工具，而是给长期运行的 Gateway 增加新的外部世界入口和能力平面。

---

### 15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载

**核心问题**：OpenClaw 为什么强调 manifest-first 和 runtime lazy loading？

**要点**：

```text
Control Plane
  discovery
  manifest parsing
  config validation
  setup/onboarding hints
  inventory

Runtime Plane
  provider execution
  channel delivery
  tool execution
  memory backend
  media generation
```

**设计价值**：

- 降低启动成本；
- 避免插件副作用；
- 支持第三方插件；
- 让 setup/doctor/discovery 不依赖完整 runtime；
- 保持 core extension-agnostic。

**源码/文档锚点**：

```text
docs/plugins/architecture.md
src/plugins/loader.ts
src/plugins/runtime/*
src/plugins/public-surface-loader.ts
src/plugins/public-surface-runtime.ts
src/plugin-sdk/*
```

**文章结论**：

> OpenClaw 的插件架构本质上分成控制面和运行面：先识别能力，再按需执行能力。

---

### 16｜安全与隔离：当 Agent 暴露给真实消息渠道之后

**核心问题**：OpenClaw 的安全边界为什么比 CLI coding agent 更复杂？

**Claude Code 主要风险**：

```text
文件读写
bash 执行
MCP tool
本地项目边界
```

**OpenClaw 额外风险**：

```text
陌生 DM
群聊 prompt injection
channel/account 误路由
owner-only tools
memory 泄漏
cron/webhook 滥用
错误 delivery target
background task 越权
sandbox / non-main session 策略
```

**源码/文档锚点**：

```text
README.md
docs/channels/groups.md
docs/channels/discord.md
docs/automation/cron-jobs.md
src/agents/tools/cron-tool.ts
src/plugin-sdk/webhook-memory-guards.ts
src/agents/sandbox.ts
src/routing/session-key.ts
```

**文章结论**：

> Claude Code 的权限问题主要发生在工具调用前；OpenClaw 的权限问题从消息进入 Gateway 的那一刻就开始了。

---

## 5. 推荐写作顺序

正式目录可以是 00-16，但写作不要严格按目录推进。建议分三阶段。

### 第一阶段：先打差异点

先写最能和 Claude Code / coding agent 拉开距离的文章：

```text
00 给 Claude Code 用户的 OpenClaw 概念迁移表
01 为什么 OpenClaw 值得单独读
05 Workspace Files
06 Memory 总览
09 Heartbeat
10 Cron
```

理由：

- 先让读者知道 OpenClaw 不是普通 coding agent；
- 先建立“长期状态 + 主动性 + 调度”的主线；
- Gateway 和 Plugin 之后再讲，避免再次偏成“多渠道框架”。

### 第二阶段：补运行时承接

```text
02 Event Sources
03 Gateway
04 Session Routing
13 Reply Shaping
```

理由：

- 第一阶段讲了“它有什么长期机制”；
- 第二阶段解释“这些机制如何被 Gateway 承接、路由和投递”。

### 第三阶段：补生态和安全边界

```text
07 Active Memory
08 Memory Flush 与 Dreaming
11 Automation Layer
12 Background Tasks 与 Subagent
14 Plugin 与 Channel
15 Control Plane vs Runtime Plane
16 安全与隔离
```

理由：

- 这些篇章更细；
- 适合在读者已经理解主线后，进入工程设计细节。

---

## 6. 第一批文章写作卡片

### 卡片 00：Claude Code 用户迁移表

**一句话**：

> 读 OpenClaw 前，要先把“开发任务 agent”的心智模型切换成“长期个人 agent runtime”。

**必须讲清**：

- session 不再只是 CLI 对话；
- workspace 不再只是 repo；
- memory 不再只是上下文压缩；
- cron/heartbeat 是 runtime 主动性；
- delivery 是真实渠道行为。

---

### 卡片 01：为什么 OpenClaw 值得单独读

**一句话**：

> 普通 coding agent 等输入，OpenClaw 则长期运行、记忆、醒来、调度、投递。

**必须讲清**：

- 不从第三方 agent 宿主关系讲；
- 不只讲 Gateway；
- 用 Memory + Heartbeat + Cron + Gateway 打开差异。

---

### 卡片 05：Workspace Files

**一句话**：

> OpenClaw workspace 是 agent 的长期生活空间。

**必须讲清**：

- AGENTS.md / USER.md / MEMORY.md / HEARTBEAT.md 分工；
- daily notes 和 DREAMS.md；
- sessions 与 cron state；
- 为什么这比普通 repo root 更像个人 AI runtime 的 home directory。

---

### 卡片 06：Memory 总览

**一句话**：

> OpenClaw memory 是文件、检索、主动召回、压缩前保存、后台整理的组合系统。

**必须讲清**：

- long-term memory；
- daily notes；
- memory_search / memory_get；
- active memory；
- memory flush；
- dreaming；
- memory-wiki。

---

### 卡片 09：Heartbeat

**一句话**：

> Heartbeat 是 OpenClaw 的低频在场机制：没事静默，有事提醒。

**必须讲清**：

- main-session periodic turn；
- HEARTBEAT.md；
- HEARTBEAT_OK contract；
- activeHours、lightContext、isolatedSession；
- 和 cron 的边界。

---

### 卡片 10：Cron

**一句话**：

> Cron 是 OpenClaw 的精确时间轴：让 agent 在指定时间运行，并把结果投递回真实渠道。

**必须讲清**：

- jobs.json / jobs-state.json；
- main / isolated / current / custom session；
- announce / webhook / none；
- background task record；
- run log / failure notification / duplicate suppression。

---

## 7. 配图规划

### 图 1：OpenClaw 总体架构图

```text
External Event Sources
  -> Gateway
  -> Session Routing
  -> Agent Workspace
  -> Agent Run
  -> Delivery
```

用途：第 01 篇。

### 图 2：Claude Code 到 OpenClaw 概念迁移图

```text
CLI Session / Repo / Prompt
        |
        v
Gateway / Workspace / Event Source / Memory / Automation
```

用途：第 00 篇。

### 图 3：Workspace 文件地图

```text
workspace/
  AGENTS.md
  USER.md
  MEMORY.md
  HEARTBEAT.md
  DREAMS.md
  memory/YYYY-MM-DD.md
  sessions/*.jsonl
  cron/jobs.json
```

用途：第 05 篇。

### 图 4：Memory 分层图

```text
Files -> Search Backend -> Active Memory -> Main Reply -> Flush -> Dreaming
```

用途：第 06-08 篇。

### 图 5：Heartbeat 运行图

```text
Timer Tick -> activeHours gate -> HEARTBEAT.md -> main-session turn -> HEARTBEAT_OK filter -> delivery or silence
```

用途：第 09 篇。

### 图 6：Cron 执行图

```text
Job Definition -> CronService -> session target -> agent run -> delivery -> task/run log
```

用途：第 10 篇。

### 图 7：Control Plane vs Runtime Plane

```text
Manifest / Discovery / Config Validation
        |
        v
Runtime Provider / Channel / Memory / Tool Execution
```

用途：第 15 篇。

---

## 8. 源码复核清单

正式起稿前，建议围绕以下链路做源码二次复核。

### 8.1 Memory 线

```text
docs/concepts/memory.md
docs/concepts/active-memory.md
docs/concepts/memory-search.md
docs/concepts/dreaming.md
src/plugins/memory-state.ts
src/plugins/memory-runtime.ts
src/plugin-sdk/memory-*.ts
src/agents/memory-search.ts
src/auto-reply/reply/memory-flush.ts
src/auto-reply/reply/agent-runner-memory.ts
```

要回答：

- active memory 在 reply path 里具体在哪里插入；
- memory_search / memory_get 如何由 plugin runtime 提供；
- memory flush 的触发阈值和 compaction 关系；
- dreaming 如何通过 cron 或后台机制运行。

### 8.2 Heartbeat 线

```text
docs/gateway/heartbeat.md
src/infra/heartbeat-runner.ts
src/infra/heartbeat-schedule.ts
src/infra/heartbeat-active-hours.ts
src/infra/heartbeat-events.ts
src/infra/heartbeat-wake.ts
src/auto-reply/heartbeat.ts
src/auto-reply/heartbeat-filter.ts
src/agents/heartbeat-system-prompt.ts
```

要回答：

- heartbeat scheduler 如何算 nextDue；
- activeHours 如何 gate；
- HEARTBEAT.md 如何注入；
- HEARTBEAT_OK 如何识别和过滤；
- heartbeat 为什么不创建 task record。

### 8.3 Cron 线

```text
docs/automation/cron-jobs.md
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

要回答：

- CronService 如何持久化 job 和 state；
- main session job 如何变成 heartbeat wake；
- isolated session 如何构造 fresh session；
- delivery fallback 如何避免和 message tool 重复；
- run log / task ledger 如何形成审计链。

### 8.4 Gateway / Session / Delivery 线

```text
docs/concepts/architecture.md
docs/concepts/session.md
docs/concepts/agent-loop.md
src/gateway/*
src/routing/session-key.ts
src/config/sessions/*
src/auto-reply/reply/*
src/infra/outbound/*
src/channels/*
```

要回答：

- 不同 channel event 如何进入 reply path；
- sessionKey 如何生成和 canonicalize；
- agent run 的 stream 如何变成 chat delta/final；
- NO_REPLY、HEARTBEAT_OK、duplicate delivery 如何处理。

### 8.5 Plugin / Control Plane 线

```text
docs/plugins/architecture.md
docs/plugins/manifest.md
docs/plugins/sdk-overview.md
src/plugins/loader.ts
src/plugins/runtime/*
src/plugins/public-surface-loader.ts
src/plugins/public-surface-runtime.ts
src/plugin-sdk/*
extensions/*
```

要回答：

- manifest-first discovery 如何避免加载完整 runtime；
- plugin capability 如何注册；
- channel/provider/memory plugin 的边界；
- core 如何保持 extension-agnostic。

---

## 9. 最终目录建议

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

---

## 10. v0.3 相比 v0.2 的变化

v0.2 的核心修正是加入 Memory / Heartbeat / Cron，但它还缺少一个对 Claude Code 读者友好的迁移层。

v0.3 增加：

```text
00 概念迁移表
02 Event Sources
05 Workspace Files
12 Background Tasks 与 Subagent
15 Control Plane vs Runtime Plane
```

这样全书主线更完整：

```text
Claude Code 读者迁移
  -> OpenClaw 运行边界
  -> 长期状态与记忆
  -> 主动性与自动化
  -> 渠道、插件、投递与安全
```

最终判断：

> **普通 coding agent 的问题是“如何完成这次任务”；OpenClaw 的问题是“一个个人 AI 助理如何长期存在、持续记忆、周期醒来、准时执行，并安全地连接真实世界”。**
