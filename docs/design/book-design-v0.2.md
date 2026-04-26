# OpenClaw 源码阅读小书整体设计 v0.2

> 目标：从 OpenClaw 自身架构出发，设计一本能讲清楚它和普通 coding agent 差异的小书。  
> 修正：v0.1 过度强调 Gateway / Channel / Reply Shaping，漏掉了 OpenClaw 更关键的“长期生命体机制”：Memory、Heartbeat、Cron。v0.2 把它们前置为主线。

---

## 0. 这本小书的核心判断

OpenClaw 不是另一个只围绕代码仓库运行的 coding agent。它更像一个 **长期在线的个人 AI 运行时**。

它的核心不是单次任务执行，而是下面五层组合：

```text
Gateway
  接入多渠道消息、HTTP hooks、控制面、设备节点

Session Routing
  把不同来源的消息映射到不同 agent/session/workspace

Memory
  把长期事实、日常上下文、可检索知识和主动召回接进回复链路

Heartbeat
  让 agent 在无人输入时周期性醒来，做轻量检查和提醒

Cron / Tasks
  让 agent 以精确时间或后台任务方式运行，并把结果投递回渠道
```

所以这本书的主题不应该是“OpenClaw 如何实现一个聊天 coding agent”，而应该是：

> **OpenClaw 如何把 agent 从一次性对话工具，设计成一个会记忆、会醒来、会定时执行、会跨渠道投递的个人 AI 运行时。**

---

## 1. 和普通 Coding Agent 的差异

普通 coding agent 的主循环大致是：

```text
用户输入任务
  -> 读取项目上下文
  -> 调工具
  -> 修改代码 / 跑命令
  -> 返回结果
```

OpenClaw 的主循环更接近：

```text
外部事件进入 Gateway
  -> 消息渠道 / webhook / cron / heartbeat / UI / 节点
  -> 路由到 agent + session + workspace
  -> memory 决定是否召回长期上下文
  -> agent run 执行回复或任务
  -> reply shaping / delivery 决定是否发、发到哪里、发几次
  -> 任务、记忆、会话、定时器继续留在系统里
```

这意味着 OpenClaw 的差异不是“工具更多”，而是：

1. **它有长期记忆系统**：`MEMORY.md`、daily notes、memory search、active memory、memory flush、dreaming。
2. **它有自发醒来的 heartbeat**：不是用户问了才运行，而是周期性主会话 turn。
3. **它有 cron 和 task ledger**：可以精确调度、隔离执行、投递结果、记录后台任务。
4. **它有 Gateway 和多渠道路由**：消息入口不是 CLI，而是各种真实沟通渠道。
5. **它有回复整形与投递策略**：因为输出要进入真实聊天渠道，而不是只打印到终端。

---

## 2. 小书总标题建议

首选：

# OpenClaw 源码导读：一个会记忆、会醒来、会定时执行的个人 AI 运行时

备选：

1. **OpenClaw 源码导读：从 Coding Agent 到 Personal Agent Runtime**
2. **OpenClaw 源码导读：Gateway、Memory、Heartbeat 与 Cron 如何组成长期在线 Agent**
3. **OpenClaw 源码导读：Agent 不再只是等待输入，而是开始拥有生活节律**

我更建议用第一个。因为它直接把 OpenClaw 最独特的三件事讲出来：

```text
记忆
醒来
定时执行
```

这三个词比“Gateway”更像读者能立刻感知到的差异。

---

## 3. 整体结构：三卷十二篇

### 第一卷：OpenClaw 的运行边界

这一卷回答：OpenClaw 到底和普通 coding agent 不一样在哪里？

#### 01｜为什么 OpenClaw 值得单独读：它不是一个只等输入的 Agent

核心问题：

- coding agent 通常是 request/response 模型；
- OpenClaw 是 Gateway + session + memory + heartbeat + cron 的长期运行模型；
- 它的“产品形态”不是终端，而是个人 AI 助理运行时。

源码锚点：

```text
README.md
docs/concepts/architecture.md
docs/automation/index.md
docs/concepts/memory.md
docs/gateway/heartbeat.md
docs/automation/cron-jobs.md
```

文章结论：

> OpenClaw 的独特性不在于一次 agent run，而在于它把 agent 放进一个长期存在的运行时里：会话会延续，记忆会沉淀，heartbeat 会周期唤醒，cron 会精确执行。

---

#### 02｜Gateway：为什么 OpenClaw 的入口不是 CLI

核心问题：

- Gateway 是长期进程；
- 多渠道消息、控制面、节点、webhook 都从这里进入；
- agent run 只是 Gateway 内部被触发的一种结果。

源码锚点：

```text
docs/concepts/architecture.md
src/gateway/*
src/channels/*
src/plugin-sdk/*
```

文章结论：

> OpenClaw 的第一主角不是 CLI，而是 Gateway。CLI 只是控制入口之一，真实产品形态是一个长期在线、接收外部事件的 agent gateway。

---

#### 03｜Session Routing：真实世界的消息如何变成 agent 上下文

核心问题：

- DM、group、channel、topic、cron、webhook 对应不同 session 策略；
- sessionKey 是外部通信关系和内部 agent 状态之间的桥；
- 多 agent / 多账户 / 多渠道时，session routing 决定隔离边界。

源码锚点：

```text
docs/concepts/session.md
src/routing/session-key.ts
src/config/sessions/*
src/auto-reply/reply/*
```

文章结论：

> OpenClaw 的 session 不是“聊天历史 ID”，而是把现实世界沟通关系映射成 agent runtime 状态的路由键。

---

### 第二卷：长期生命体机制

这一卷应该成为小书的核心。它回答：OpenClaw 为什么不像一个“被动问答工具”，而像一个长期在线的个人助理？

---

#### 04｜Memory 总览：OpenClaw 如何让 Agent 真正“有记忆”

核心问题：

OpenClaw 的 memory 不是一个单点功能，而是一组层次：

```text
MEMORY.md
  长期事实、偏好、决策

memory/YYYY-MM-DD.md
  每日上下文、运行观察、短期记录

memory_search / memory_get
  可检索访问层

memory backend
  builtin / QMD / Honcho

memory flush
  compaction 前的静默保存

dreaming
  后台整理和长期记忆晋升

memory-wiki
  把 durable memory 编译为可维护知识库
```

源码/文档锚点：

```text
docs/concepts/memory.md
docs/concepts/memory-search.md
docs/concepts/memory-builtin.md
docs/concepts/active-memory.md
docs/plugins/memory-wiki.md
src/plugins/memory-state.ts
src/plugins/memory-runtime.ts
src/plugin-sdk/memory-*.ts
src/memory/root-memory-files.ts
src/auto-reply/reply/memory-flush.ts
src/auto-reply/reply/agent-runner-memory.ts
```

文章结论：

> OpenClaw 的 memory 不是隐藏在模型里的神秘状态，而是一套文件优先、插件托管、可检索、可刷新、可后台整理的长期上下文系统。

建议图：

```text
User / Agent Turn
      |
      v
Active Memory?  ---> memory_search / memory_get
      |
      v
Main Reply
      |
      v
Memory Flush before compaction
      |
      v
MEMORY.md / daily notes / DREAMS.md / wiki layer
```

---

#### 05｜Active Memory：为什么记忆不应该总靠主 Agent 自己想起来

核心问题：

- 很多 agent 的 memory 是 reactive：用户说“记一下”或主 agent 自己决定 search；
- OpenClaw 的 active memory 是 reply 前的阻塞式 memory sub-agent；
- 它只在合适的交互式持久聊天 session 中运行；
- 它用 `memory_search` / `memory_get` 找到相关记忆，再以隐藏上下文注入主回复。

关键设计：

```text
User Message
  -> Build Memory Query
  -> Active Memory Blocking Pass
  -> NONE 或 relevant summary
  -> Hidden active_memory_plugin context
  -> Main Reply
```

源码/文档锚点：

```text
docs/concepts/active-memory.md
src/plugins/memory-runtime.ts
src/plugins/memory-state.ts
src/agents/memory-search.ts
src/plugin-sdk/memory-host-search.ts
```

文章结论：

> Active Memory 的价值是把“记忆召回”从主回复里拆出来，让长期上下文在回复生成前自然出现，而不是等主 agent 碰运气想起来。

---

#### 06｜Memory Flush 与 Dreaming：OpenClaw 如何防止上下文在压缩时丢失

核心问题：

- 长对话一定会 compaction；
- compaction 前如果重要事实还没落盘，就会丢；
- memory flush 是 compaction 前的静默 turn；
- dreaming 是更后台的长期记忆晋升机制，写入 `DREAMS.md` 供人审查。

源码/文档锚点：

```text
docs/concepts/memory.md
docs/concepts/dreaming.md
src/auto-reply/reply/memory-flush.ts
src/auto-reply/reply/agent-runner-memory.ts
src/plugins/memory-state.ts
src/commands/doctor-cron-dreaming-payload-migration.ts
```

文章结论：

> OpenClaw 把“上下文压缩”前移成一次记忆保存机会：先让 agent 把重要内容写入 memory，再进行 compaction。这是长期在线系统必须有的防丢机制。

---

#### 07｜Heartbeat：为什么 OpenClaw 会在没有用户输入时醒来

核心问题：

Heartbeat 是 OpenClaw 很有辨识度的机制。它不是 cron，也不是后台任务，而是：

```text
周期性主会话 agent turn
```

它默认：

```text
every: 30m
prompt: Read HEARTBEAT.md if it exists...
如果没事，回复 HEARTBEAT_OK
```

关键行为：

- heartbeat 运行在 main session，默认带主会话上下文；
- 可以 `lightContext`，只注入 `HEARTBEAT.md`；
- 可以 `isolatedSession`，避免带完整历史；
- 可以配置 active hours；
- 可以投递到 last contact 或指定 channel；
- `HEARTBEAT_OK` 会被识别、剥离、必要时静默丢弃；
- heartbeat run 不创建 background task records。

源码/文档锚点：

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

文章结论：

> Heartbeat 让 OpenClaw 从“被动响应用户”变成“周期性检查世界”。它不是为了准点执行任务，而是为了让个人助理拥有低频、克制、可静默的在场感。

建议图：

```text
Timer Tick
  -> activeHours gate
  -> queue/session lane check
  -> build heartbeat prompt + HEARTBEAT.md
  -> run main-session turn
  -> if HEARTBEAT_OK: drop/suppress
  -> else deliver alert/check-in
```

---

#### 08｜Cron：精确调度、隔离执行与结果投递

核心问题：

Cron 和 Heartbeat 的边界必须讲清楚：

| 维度 | Heartbeat | Cron |
|---|---|---|
| 时间 | 近似周期 | 精确 at/every/cron expression |
| 会话 | 主会话 turn | main / isolated / current / custom session |
| 任务记录 | 不创建 task record | 每次执行创建 background task record |
| 适合 | inbox/calendar/轻量检查 | 日报、提醒、后台分析、webhook 触发 |
| 输出 | 主会话内联或 last target | announce / webhook / none |

Cron 的核心是 Gateway 内置 scheduler：

```text
jobs.json       持久化 job definitions
jobs-state.json 持久化 runtime execution state
run-log         记录执行历史
background task 记录 detached work
```

源码/文档锚点：

```text
docs/automation/cron-jobs.md
docs/automation/index.md
src/gateway/server-cron.ts
src/cron/service.ts
src/cron/store.ts
src/cron/isolated-agent/run.ts
src/cron/isolated-agent/session.ts
src/agents/tools/cron-tool.ts
src/cli/cron-cli.ts
```

文章结论：

> Cron 是 OpenClaw 的精确时间轴：它把 agent run 从“有人说话才发生”扩展成“时间到了也能发生”，并且把执行、投递、失败通知和任务审计全部纳入 Gateway。

建议图：

```text
Cron Job Definition
  -> CronService scheduler
  -> choose session target
       main: enqueue system event + heartbeat wake
       isolated: fresh cron:<jobId> agent turn
       current/custom: bound persistent context
  -> run agent
  -> message tool or fallback announce/webhook
  -> task ledger + run log + state update
```

---

#### 09｜Automation Layer：Heartbeat、Cron、Hooks、Tasks、Standing Orders 如何分工

核心问题：

OpenClaw 的 automation 不只是 cron。它有一组机制：

```text
Heartbeat       周期性轻量检查
Cron            精确调度和后台任务
Hooks           生命周期/消息流/工具调用事件
Tasks           detached work ledger
Task Flow       多步骤 durable orchestration
Standing Orders 持久授权与长期指令
```

源码/文档锚点：

```text
docs/automation/index.md
docs/automation/tasks.md
docs/automation/hooks.md
docs/automation/taskflow.md
docs/automation/standing-orders.md
```

文章结论：

> OpenClaw 的自动化层不是一个“定时任务功能”，而是一套让 agent 拥有长期行为边界的操作系统层：什么时候醒、能做什么、做完如何审计、失败如何通知。

---

### 第三卷：消息入口与生态边界

这一卷回到 Gateway、插件和真实渠道，把长期生命体机制放到真实通信环境里看。

---

#### 10｜Reply Shaping：为什么聊天渠道需要一层回复整形

核心问题：

普通 coding agent 输出到终端就结束了，但 OpenClaw 要面对真实聊天渠道：

```text
assistant deltas
工具事件
最终回复
NO_REPLY / no_reply
HEARTBEAT_OK
message tool 已经发过的内容
fallback announce
Discord/Telegram/Feishu 等渠道差异
```

所以它需要回复整形：

```text
model/tool stream
  -> chat delta/final
  -> duplicate suppression
  -> no_reply / heartbeat ack filtering
  -> fallback delivery
  -> channel adapter
```

源码锚点：

```text
docs/concepts/agent-loop.md
src/auto-reply/reply/*
src/auto-reply/heartbeat-filter.ts
src/cron/isolated-agent/run.ts
src/infra/outbound/*
src/channels/*
```

文章结论：

> OpenClaw 的输出不是“模型吐字”，而是要被转译成真实渠道里的消息行为：什么时候发、发几条、是否静默、是否去重、失败时是否兜底。

---

#### 11｜Plugin 与 Channel：OpenClaw 如何扩展新的世界入口

核心问题：

- plugin 不只是工具扩展；
- 它可以提供 channel、provider、memory、media、web、hooks 等能力；
- manifest-first 设计让 discovery/config validation 不必加载完整 runtime；
- channel plugin 把真实世界消息转进 Gateway。

源码/文档锚点：

```text
docs/plugins/architecture.md
docs/plugins/manifest.md
docs/plugins/sdk-overview.md
src/plugins/*
src/plugin-sdk/*
src/channels/*
extensions/*
```

文章结论：

> OpenClaw 的插件系统不是“给 agent 加工具”，而是给长期运行的 Gateway 增加新的外部世界入口和能力平面。

---

#### 12｜安全与隔离：当 Agent 暴露给真实消息渠道之后

核心问题：

OpenClaw 面临的风险不同于单机 CLI coding agent：

- 陌生 DM；
- 群聊 prompt injection；
- channel/account 误路由；
- memory 泄漏；
- cron/webhook 被滥用；
- owner-only tools；
- sandbox 和工具权限。

源码/文档锚点：

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

文章结论：

> 当 agent 进入真实消息渠道，安全边界不再只是“能不能执行 bash”，而是身份、会话、记忆、调度、渠道投递和工具权限共同组成的运行时边界。

---

## 4. 推荐写作顺序

不要按源码目录写，按读者理解递进写。

第一阶段先写 4 篇，建立主线：

```text
01 为什么 OpenClaw 值得单独读
04 Memory 总览
07 Heartbeat
08 Cron
```

原因：

- 这四篇最能和普通 coding agent 拉开距离；
- 先讲“会记忆、会醒来、会定时执行”，比先讲 Gateway 更有辨识度；
- Gateway / Session / Reply Shaping 可以作为第二阶段解释这些能力如何落到真实渠道。

第二阶段写运行时承接：

```text
02 Gateway
03 Session Routing
10 Reply Shaping
```

第三阶段写生态和边界：

```text
05 Active Memory
06 Memory Flush 与 Dreaming
09 Automation Layer
11 Plugin 与 Channel
12 安全与隔离
```

---

## 5. 第一批文章卡片

### 卡片 A：为什么 OpenClaw 值得单独读

一句话：

> OpenClaw 把 agent 从“用户输入才运行的工具”，变成“长期在线、能记忆、能醒来、能定时执行的个人 AI 运行时”。

必须讲清：

- 不从第三方 agent 宿主关系讲；
- 不只讲 Gateway；
- 重点是长期运行机制的组合：Memory + Heartbeat + Cron + Gateway。

---

### 卡片 B：Memory 总览

一句话：

> OpenClaw 的 memory 是文件、检索、主动召回、压缩前保存、后台整理的组合系统。

必须讲清：

- `MEMORY.md` 和 daily notes 的职责；
- memory tools 是访问层；
- active memory 是回复前召回层；
- memory flush 是 compaction 前防丢层；
- dreaming 是后台晋升层。

---

### 卡片 C：Heartbeat

一句话：

> Heartbeat 是 OpenClaw 的低频在场机制：没事就静默，有事才提醒。

必须讲清：

- heartbeat 是 main-session turn；
- 默认 prompt 和 `HEARTBEAT_OK` contract；
- 和 cron 的差异；
- 为什么它适合 inbox/calendar/checkup，而不是精确日报。

---

### 卡片 D：Cron

一句话：

> Cron 是 OpenClaw 的精确时间轴：让 agent 在指定时间运行，并把执行纳入任务、投递和审计系统。

必须讲清：

- job definitions / state 分离；
- main / isolated / current / custom session；
- announce / webhook / none；
- 每次 cron 创建 background task；
- isolated run 的清理和最终结果投递。

---

## 6. 本轮源码复核清单

下一步不急着写正文，先围绕三条核心机制深读源码：

### Memory 线

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

- active memory 在 agent reply path 中具体在哪里插入？
- memory_search / memory_get 是如何由 plugin runtime 提供的？
- memory flush 的触发阈值和 compaction 关系是什么？
- dreaming 如何通过 cron 或后台机制运行？

### Heartbeat 线

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

- heartbeat scheduler 如何算 nextDue？
- activeHours 如何 gate？
- `HEARTBEAT.md` 如何注入？
- `HEARTBEAT_OK` 如何被识别和过滤？
- heartbeat 为什么不创建 task record？

### Cron 线

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

- CronService 如何持久化 job 和 state？
- main session job 如何变成 heartbeat wake？
- isolated session 如何构造 fresh session？
- delivery fallback 如何避免和 message tool 重复？
- run log / task ledger 如何形成审计链？

---

## 7. v0.2 的章节顺序总表

| 顺序 | 标题 | 主问题 |
|---:|---|---|
| 01 | 为什么 OpenClaw 值得单独读 | 它为什么不是普通 coding agent |
| 02 | Gateway | 为什么入口不是 CLI |
| 03 | Session Routing | 真实消息如何变成 agent 上下文 |
| 04 | Memory 总览 | OpenClaw 如何让 agent 有长期记忆 |
| 05 | Active Memory | 为什么记忆要在主回复前召回 |
| 06 | Memory Flush 与 Dreaming | 如何防止压缩丢上下文，并后台整理记忆 |
| 07 | Heartbeat | 为什么 agent 会在没人说话时醒来 |
| 08 | Cron | 如何精确调度、隔离执行、投递结果 |
| 09 | Automation Layer | Heartbeat/Cron/Hooks/Tasks 如何分工 |
| 10 | Reply Shaping | 为什么聊天渠道需要输出整形 |
| 11 | Plugin 与 Channel | 如何扩展新的世界入口 |
| 12 | 安全与隔离 | 真实渠道暴露后的安全边界 |

---

## 8. 这版和 v0.1 的关键差异

v0.1 的问题：

```text
Gateway / Channel / Reply Shaping 讲得太靠前
Memory / Heartbeat / Cron 没有成为小书主线
读者会以为 OpenClaw 只是“多渠道 agent gateway”
```

v0.2 的修正：

```text
把 OpenClaw 定位为长期在线个人 AI 运行时
把 Memory / Heartbeat / Cron 作为核心差异
Gateway 回到承载层
Session Routing 回到状态隔离层
Reply Shaping 回到渠道输出层
```

最终主线：

> **普通 coding agent 等待用户输入；OpenClaw 则把 agent 放进一个长期在线系统：它会记忆、会醒来、会定时执行，并能把结果送回真实沟通渠道。**
