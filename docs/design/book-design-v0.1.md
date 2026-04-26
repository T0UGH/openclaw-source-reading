# OpenClaw 源码阅读小书整体设计

> 版本：v0.1  
> 目标：先确定整本小书的主线、章节结构、源码锚点与写作边界，再进入单篇正文。

---

## 0. 关键修正

这本小书不再把 OpenClaw 和 Pi 建立关系，也不使用“Pi 宿主化 / SDK integration / 嵌入 Pi agent loop”这类判断。

后续统一从 OpenClaw 自身架构出发：

> **OpenClaw 是一个把 agent 放进长期在线 Gateway 的个人 AI 运行时。**

它和 Claude Code / Codex / 常规 coding agent 的差异，不是“多了几个工具”，而是运行边界变了：

```text
终端/IDE 里的 coding task
  -> 真实消息渠道里的长期在线个人助理
```

所以这本小书要讲的不是“OpenClaw 如何也能做 coding agent”，而是：

> 当 agent 从终端搬到 Gateway、消息渠道、设备节点和长期会话里，源码架构会发生什么变化？

---

## 1. 小书定位

### 书名候选

优先推荐：

> **OpenClaw 源码导读：从 Coding Agent 到 Personal Agent Gateway**

备选：

1. **OpenClaw 不是另一个 Claude Code：它把 Agent 放进了个人 Gateway**
2. **从终端到消息入口：OpenClaw 如何重写 Agent 的运行边界**
3. **为什么 OpenClaw 的核心不是工具调用，而是 Session Routing**
4. **当 Agent 暴露给 WhatsApp / Telegram / Slack：OpenClaw 的安全与会话模型**

### 一句话卖点

> OpenClaw 最值得读的不是通用 agent loop，而是它如何把一个 agent 变成长期在线、可被多渠道消息唤起、可路由到不同身份和工作区、还能接入插件与设备节点的个人 AI Gateway。

### 读者对象

这本小书主要给三类人：

1. **做 coding agent / AI runtime 的工程师**  
   想看 agent 离开终端后，消息入口、session、权限、插件、设备节点会怎样改变架构。

2. **做个人 AI 助理 / 多渠道 bot / automation gateway 的工程师**  
   想看一个真实项目如何处理 Telegram、Slack、Discord、Feishu、iMessage、Cron、Webhook 等入口。

3. **已经读过 Claude Code / Codex 源码导读的人**  
   想比较：从“代码仓库里的 agent”到“个人消息系统里的 agent”，哪些设计问题会变成主线。

---

## 2. 核心判断

### 2.1 OpenClaw 的主语不是代码仓库，而是入口、身份和会话

Claude Code / Codex 的典型路径：

```text
用户在终端/IDE 发起任务
  -> agent 读取项目上下文
  -> 调工具
  -> 改代码 / 跑测试
  -> 返回结果
```

OpenClaw 的典型路径：

```text
消息从 Telegram / WhatsApp / Slack / Discord / Feishu / iMessage / Cron / Webhook / Node 进入
  -> Gateway 识别来源、账户、peer、group、agentId
  -> 路由到对应 agent/session/workspace
  -> 进入 agent runtime
  -> 工具执行 / 消息回复 / 设备动作 / Canvas 更新 / 后台任务
```

所以源码阅读入口不能是“工具列表”或“模型调用”，而应该是：

```text
Gateway -> channel intake -> session routing -> agent run -> reply shaping -> channel delivery
```

### 2.2 OpenClaw 的差异来自产品形态，而不是单点机制

它的独特性不是某一个算法，而是一组产品约束共同塑造的架构：

- agent 必须长期在线；
- 输入来自真实消息渠道，不只来自终端；
- 不同 channel/account/peer/group 要路由到不同 session；
- 多个 agent/persona 可以共存；
- 外部陌生人可能向 agent 发送 prompt；
- 有些工具会直接发消息，所以最终回复要去重和整形；
- 插件不只是工具扩展，还包括 channel、provider、speech、media、gateway discovery；
- 设备节点、Canvas、Voice 让 agent 进入个人设备网络。

这就是全书的中心线。

---

## 3. 不写什么

为了避免写成泛化 architecture tour，这本小书先不做这些事：

1. **不按目录逐个介绍模块**  
   目录游览没有价值，所有章节都要围绕“为什么 Gateway 改变了 agent 架构”来讲。

2. **不把 OpenClaw 写成 Claude Code 平替**  
   Claude Code / Codex 是对照物，不是主角。

3. **不把 OpenClaw 写成普通 bot 框架**  
   普通 bot 框架重点是 channel adapter；OpenClaw 的重点是 channel + session + agent runtime + tools + plugins + nodes 的组合。

4. **不再引入 Pi 关系叙事**  
   后续若源码路径或历史命名出现相似词，也只作为 OpenClaw 内部实现细节复核，不推导外部依赖关系。

5. **不从 generic agent loop 开场**  
   agent loop 重要，但它应该放在 Gateway 路由和消息入口之后讲。

---

## 4. 整体结构设计

建议做成一本 **8-10 篇的小书**，不是大而全的手册。

### 00. 导读：为什么 OpenClaw 值得单独读

**核心问题**：OpenClaw 和 Claude Code / Codex / 普通 coding agent 的差异到底在哪里？

**一行论点**：OpenClaw 的核心不是“更会写代码”，而是把 agent 放进长期在线的个人 Gateway。

**要讲清楚**：

- Claude Code / Codex：代码仓库里的 agent；
- OpenClaw：消息入口和个人设备网络里的 agent；
- 这会改变 session、安全、插件、回复、设备连接等一系列设计。

**源码/文档锚点**：

```text
README.md
docs/concepts/architecture.md
docs/concepts/agent-loop.md
docs/concepts/session.md
```

---

### 01. Gateway：为什么 OpenClaw 的第一主角不是 CLI

**核心问题**：为什么 OpenClaw 必须有一个长期运行的 Gateway？

**一行论点**：OpenClaw 的 Gateway 不是附属 API server，而是所有 channel、client、node、agent run 的控制平面。

**写作主线**：

```text
Gateway daemon
  -> WebSocket protocol
  -> clients / nodes connect
  -> channel providers live inside gateway
  -> agent request becomes stream/lifecycle event
```

**重点解释**：

- Gateway 为什么要长期运行；
- 控制平面和 assistant 产品之间的关系；
- WebSocket protocol 如何连接 CLI、web admin、macOS app、nodes；
- 为什么“一个 host 一个 Gateway”是架构约束。

**源码/文档锚点**：

```text
docs/concepts/architecture.md
docs/reference/rpc.md 或 docs/gateway/protocol*
src/gateway/**
src/rpc/** 或协议 schema 相关文件（待复核）
```

---

### 02. Channel Intake：真实世界消息如何进入 agent

**核心问题**：Telegram / Slack / Discord / Feishu / iMessage 等消息进来后，OpenClaw 如何把它们变成一次 agent run？

**一行论点**：OpenClaw 的输入不是单一 prompt，而是带 channel、account、peer、thread、group、身份上下文的事件。

**写作主线**：

```text
channel adapter
  -> inbound message normalization
  -> sender / peer / thread identity
  -> routing decision
  -> auto-reply / agent request
```

**重点解释**：

- 为什么 channel adapter 不只是“收消息”；
- message envelope 里哪些字段会影响 session；
- 群聊、DM、thread、account 的区别；
- 为什么真实消息入口会把 prompt injection 风险提前到 intake 层。

**源码/文档锚点**：

```text
src/channels/**
src/channels/plugins/**
src/auto-reply/reply/get-reply.ts
src/auto-reply/reply/get-reply-run.ts
docs/channels/**
docs/concepts/session.md
```

---

### 03. Session Routing：为什么 sessionKey 是 OpenClaw 的核心概念

**核心问题**：OpenClaw 如何决定“一条消息应该进入哪个会话”？

**一行论点**：OpenClaw 的 session 不是 CLI 历史，而是现实通信关系映射到 agent 上下文的路由状态。

**写作主线**：

```text
channel/account/peer/group/thread
  -> session key
  -> session store row
  -> transcript file
  -> reset / idle / daily lifecycle
```

**重点解释**：

- DM 默认共享 session 的便利与风险；
- per-peer / per-channel-peer / per-account-channel-peer 的取舍；
- group / room / cron / webhook 为什么要隔离；
- session store 和 transcript 分别保存什么；
- session reset 不是清空聊天那么简单，而是长期助理的上下文治理。

**源码/文档锚点**：

```text
docs/concepts/session.md
src/routing/session-key.ts
src/channels/session*.ts
src/session* 或 src/agents/session*（待复核）
```

---

### 04. Agent Runtime：Gateway 事件如何变成一次可执行回合

**核心问题**：OpenClaw 如何从一次已路由消息进入真正的 agent 执行？

**一行论点**：在 OpenClaw 中，agent run 是 Gateway 管理下的生命周期对象，不是单纯的一次 CLI 调用。

**写作主线**：

```text
agent request accepted
  -> run id / queue / session lock
  -> model + tools + workspace + prompt assembly
  -> streaming events
  -> lifecycle end/error
```

**重点解释**：

- 为什么同一个 session 的 run 要串行；
- runId、lifecycle、stream event 如何配合；
- prompt / skills / workspace / tools 如何在 run 前装配；
- agent.wait 为什么只是 wait，不一定等于停止或控制 run；
- session write lock 为什么重要。

**源码/文档锚点**：

```text
docs/concepts/agent-loop.md
src/agents/**
src/auto-reply/reply/agent-runner-execution.ts
src/auto-reply/reply/get-reply-run.ts
src/agents/system-prompt.ts
```

---

### 05. Reply Shaping：为什么聊天渠道需要一层回复整形

**核心问题**：为什么 OpenClaw 不能简单把模型最终文本发回去？

**一行论点**：面向真实聊天渠道时，assistant output、tool event、message tool、partial delta、silent reply 都需要被重新整形。

**写作主线**：

```text
assistant delta
  -> tool event
  -> lifecycle event
  -> chat delta/final
  -> duplicate suppression
  -> NO_REPLY filter
  -> channel delivery
```

**重点解释**：

- 为什么 tool 已经发消息后，assistant 不应重复确认；
- 为什么需要 NO_REPLY / no_reply；
- partial reply 和 final reply 的边界；
- reasoning / tool / assistant stream 如何分流；
- fallback tool error reply 的意义。

**源码/文档锚点**：

```text
docs/concepts/agent-loop.md
src/auto-reply/reply/get-reply.ts
src/auto-reply/reply/get-reply-run.ts
src/channels/** reply / delivery 相关实现
```

---

### 06. Multi-agent：一个 Gateway 如何托管多个独立人格

**核心问题**：OpenClaw 的 multi-agent 和 Claude Code / Codex 的 subagent 有什么不同？

**一行论点**：OpenClaw 的 multi-agent 首先是长期身份、工作区、凭证和 session store 的隔离，不是任务内临时分工。

**写作主线**：

```text
agentId
  -> workspace
  -> agentDir
  -> auth profiles
  -> model registry
  -> sessions
  -> channel binding
```

**重点解释**：

- 一个 agent 为什么是一个 fully scoped brain；
- account / channel / binding 如何路由到 agent；
- 多人共用一个 Gateway 时为什么必须隔离；
- multi-agent 和 subagent 的边界；
- 什么时候使用长期 agent，什么时候使用临时 subagent。

**源码/文档锚点**：

```text
docs/concepts/multi-agent.md
src/agents/**
src/agents/subagent-spawn.ts
src/agents/tools/subagents-tool.ts
src/agents/subagent-registry.ts
```

---

### 07. Plugin System：OpenClaw 插件为什么不只是工具扩展

**核心问题**：OpenClaw 的 plugin system 和 coding agent extension 有什么不同？

**一行论点**：OpenClaw 插件扩展的是 Gateway 能力面，而不只是 agent 工具面。

**写作主线**：

```text
manifest + discovery
  -> enablement + validation
  -> capability registration
  -> runtime loading
  -> tools / channels / providers / speech / media / services
```

**重点解释**：

- capability model 为什么重要；
- plugin shape：plain-capability / hybrid-capability / hook-only / non-capability；
- 为什么 manifest/config validation 尽量不执行插件代码；
- channel plugin 和 provider plugin 与普通 tool plugin 的差异；
- 插件系统如何服务 Gateway，而不是只服务 agent loop。

**源码/文档锚点**：

```text
docs/plugins/architecture.md
src/plugin-sdk/**
src/plugins/**
src/channels/plugins/**
```

---

### 08. Context Engine：长期在线 agent 如何治理上下文

**核心问题**：OpenClaw 的 context engine 为什么不是简单 compaction？

**一行论点**：OpenClaw 把上下文治理做成生命周期组件，让 assemble、compact、afterTurn、subagent preparation 都可被引擎接管。

**写作主线**：

```text
ingest / ingestBatch
  -> maintain / afterTurn
  -> assemble
  -> systemPromptAddition
  -> compact
  -> subagent lifecycle hooks
```

**重点解释**：

- assemble 和 compact 的职责边界；
- systemPromptAddition 为什么重要；
- ownsCompaction 表达什么；
- foreground/background maintenance 的意义；
- context engine 如何和 sessionKey、transcript、subagent 关联。

**源码/文档锚点**：

```text
src/context-engine/types.ts
src/context-engine/registry.ts
docs/concepts/compaction.md
docs/concepts/session.md
```

---

### 09. Nodes / Canvas / Voice：为什么 OpenClaw 接近个人设备操作层

**核心问题**：为什么 OpenClaw 不只是聊天 bot 或 coding agent？

**一行论点**：Node、Canvas、Voice 把 OpenClaw 从“消息回复系统”扩展成个人设备网络中的 agent runtime。

**写作主线**：

```text
Gateway WebSocket
  -> node connect role=node
  -> device identity / pairing
  -> canvas / camera / screen / location commands
  -> voice wake / talk mode
```

**重点解释**：

- node 与普通 client 的区别；
- device pairing 为什么是安全边界；
- Canvas 为什么不是普通 UI，而是 agent 可控制的 visual workspace；
- Voice Wake / Talk Mode 如何改变交互入口。

**源码/文档锚点**：

```text
docs/concepts/architecture.md
docs/nodes/**
docs/platforms/**
src/nodes/** 或相关实现（待复核）
src/canvas/** 或相关实现（待复核）
```

---

### 10. Security：真实消息入口如何改变 agent 安全模型

**核心问题**：OpenClaw 的安全为什么不能只按 bash permission 理解？

**一行论点**：当 agent 暴露给真实消息渠道时，安全边界从“本地命令审批”扩展成入口身份、DM pairing、session 隔离、sandbox、plugin install 和 channel delivery 的组合。

**写作主线**：

```text
unknown sender
  -> pairing / allowlist
  -> session isolation
  -> sandbox policy
  -> tool allow/deny
  -> plugin install guard
  -> channel delivery guard
```

**重点解释**：

- unknown DM 为什么默认不能直接进 agent；
- group/channel prompt injection 的风险；
- main session full host access 与 non-main sandbox 的取舍；
- plugin/channel/provider 扩展带来的供应链风险；
- `openclaw doctor` / security audit 这类工具为什么重要。

**源码/文档锚点**：

```text
README.md
docs/gateway/security.md
docs/gateway/sandboxing.md
src/security/** 或 audit/doctor 相关实现（待复核）
src/channels/plugins/** allowlist / pairing 相关实现
```

---

## 5. 推荐写作顺序

不要按最终章节顺序全部铺开。建议第一批先写 4 篇，先把差异主线立住。

### 第一批：立主线

1. **00 导读：为什么 OpenClaw 值得单独读**  
   目标：和 Claude Code / Codex / 普通 coding agent 拉开距离。

2. **01 Gateway：为什么 OpenClaw 的第一主角不是 CLI**  
   目标：建立 Gateway-first 视角。

3. **03 Session Routing：为什么 sessionKey 是 OpenClaw 的核心概念**  
   目标：解释真实消息入口如何改变会话模型。

4. **05 Reply Shaping：为什么聊天渠道需要一层回复整形**  
   目标：用一个具体机制说明“聊天渠道 runtime”和“终端输出”的差异。

### 第二批：补 runtime 和隔离

5. **04 Agent Runtime：Gateway 事件如何变成一次可执行回合**
6. **06 Multi-agent：一个 Gateway 如何托管多个独立人格**
7. **10 Security：真实消息入口如何改变 agent 安全模型**

### 第三批：扩展生态和设备边界

8. **07 Plugin System：OpenClaw 插件为什么不只是工具扩展**
9. **08 Context Engine：长期在线 agent 如何治理上下文**
10. **09 Nodes / Canvas / Voice：为什么 OpenClaw 接近个人设备操作层**

---

## 6. 第一批写作卡片

### 00 导读：为什么 OpenClaw 值得单独读

**核心问题**：OpenClaw 到底和 Claude Code / Codex / 普通 coding agent 差在哪里？

**一行论点**：OpenClaw 的核心不是“更强 coding”，而是把 agent 放进了长期在线的个人 Gateway。

**写作骨架**：

1. 从读者误解开场：又一个 coding agent？
2. 对照 Claude Code / Codex：代码仓库里的 agent；
3. 引出 OpenClaw：消息入口、Gateway、长期在线、设备节点；
4. 给出全书地图：Gateway / session / runtime / reply / multi-agent / plugin / security；
5. 明确本书不做目录游览。

**必须覆盖**：

- README 中 “personal AI assistant” 和 “Gateway is control plane” 的定位；
- supported channels；
- Gateway architecture 的基本图景；
- session routing 是后续重点。

**不要覆盖**：

- 不展开全部工具；
- 不做安装教程；
- 不讲成 Pi 或任何外部 runtime 的宿主化。

---

### 01 Gateway：为什么 OpenClaw 的第一主角不是 CLI

**核心问题**：OpenClaw 为什么需要一个长期运行的 Gateway？

**一行论点**：Gateway 是 OpenClaw 的控制平面，它统一承载 channel、client、node 和 agent run。

**写作骨架**：

1. CLI-first agent 的默认假设；
2. OpenClaw 的 Gateway-first 架构；
3. Gateway 连接哪些对象：channels / clients / nodes；
4. WS protocol 和 event stream 的作用；
5. 为什么这会改变 agent runtime 的组织方式。

**源码锚点**：

```text
docs/concepts/architecture.md
README.md
Gateway protocol / rpc schema 相关源码待复核
```

---

### 03 Session Routing：为什么 sessionKey 是 OpenClaw 的核心概念

**核心问题**：同一条消息为什么不能只进入一个默认聊天历史？

**一行论点**：sessionKey 是 OpenClaw 把现实通信关系映射到 agent 上下文的关键机制。

**写作骨架**：

1. 终端 agent 的 session 通常是什么；
2. OpenClaw 面对的是 DM、group、room、cron、webhook；
3. session routing 表；
4. session store / transcript；
5. DM sharing 的便利与安全代价；
6. 为什么 sessionKey 是后续 multi-agent 和 security 的基础。

**源码锚点**：

```text
docs/concepts/session.md
src/routing/session-key.ts
src/channels/session*.ts
```

---

### 05 Reply Shaping：为什么聊天渠道需要一层回复整形

**核心问题**：为什么 OpenClaw 不能直接把模型输出发回渠道？

**一行论点**：面向真实聊天渠道时，回复不是单个字符串，而是 assistant/tool/lifecycle stream 经过去重、静默过滤和 channel delivery 后的结果。

**写作骨架**：

1. 终端 agent 的输出模型：打印文本和工具结果；
2. OpenClaw 的输出模型：assistant delta / tool stream / lifecycle；
3. chat delta/final；
4. messaging tool duplicate suppression；
5. NO_REPLY / no_reply；
6. fallback tool error reply；
7. 这层机制说明 OpenClaw 是 channel runtime，不只是 agent loop。

**源码锚点**：

```text
docs/concepts/agent-loop.md
src/auto-reply/reply/get-reply.ts
src/auto-reply/reply/get-reply-run.ts
channel delivery 相关源码待复核
```

---

## 7. 建议配图

整本小书至少需要 4 张高价值图：

### 图 1：OpenClaw 与 Claude Code / Codex 的运行边界对比

```text
Claude Code / Codex:
User -> CLI/IDE -> Agent Runtime -> Repo/Tools -> Terminal Reply

OpenClaw:
Channels/Nodes/Cron/Webhook -> Gateway -> Routing -> Agent Runtime -> Channel/Device Delivery
```

用途：00 导读。

### 图 2：Gateway 控制平面

```text
Clients         Channels
   \              /
    \            /
      Gateway ---- Nodes
         |
      Agent Runs
         |
      Sessions / Workspace / Tools
```

用途：01 Gateway。

### 图 3：Session Routing

```text
channel + account + peer + group/thread + agentId
  -> sessionKey
  -> sessions.json
  -> transcript.jsonl
  -> agent run
```

用途：03 Session Routing。

### 图 4：Reply Shaping Pipeline

```text
model/tool events
  -> assistant stream
  -> tool stream
  -> lifecycle
  -> chat delta/final
  -> duplicate suppression / NO_REPLY
  -> channel delivery
```

用途：05 Reply Shaping。

---

## 8. 项目落地建议

如果要开独立源码阅读仓库，建议结构：

```text
openclaw-source-reading/
  README.md
  docs/
    guidebook/
      00-book-design.md
      README-writing-cards-v1.md
      00-why-openclaw.md
      01-gateway-first.md
      02-channel-intake.md
      03-session-routing.md
      04-agent-runtime.md
      05-reply-shaping.md
      06-multi-agent.md
      07-plugin-system.md
      08-context-engine.md
      09-nodes-canvas-voice.md
      10-security.md
    notes/
      source-map-gateway.md
      source-map-session-routing.md
      source-map-reply-shaping.md
    assets/
      openclaw-boundary-comparison.svg
      gateway-control-plane.svg
      session-routing.svg
      reply-shaping-pipeline.svg
```

第一步不是写正文，而是先补三份 evidence note：

1. `source-map-gateway.md`：Gateway / RPC / WS / clients / nodes 源码锚点；
2. `source-map-session-routing.md`：sessionKey / session store / channel routing 源码锚点；
3. `source-map-reply-shaping.md`：assistant/tool/lifecycle stream 与 final delivery 源码锚点。

然后再开始写 `00` 和 `01`。

---

## 9. 当前待复核清单

为了避免再次建立错误关系，下一轮源码调研优先验证这些点：

1. Gateway RPC / WS protocol 的真实入口文件；
2. channel inbound message 到 auto-reply / agent request 的调用链；
3. sessionKey 的构造、规范化和 store 写入链路；
4. agent run queue / lifecycle / stream 的实际实现；
5. reply shaping 的具体函数边界；
6. plugin capability registration 的核心 registry；
7. nodes / canvas / voice 的运行入口；
8. security audit / pairing / allowlist / sandbox 的实现边界。

---

## 10. 最终原则

这本小书的写法要始终服务一个判断：

> OpenClaw 的价值不是把 agent loop 再写一遍，而是展示当 agent 进入真实个人消息系统后，Gateway、session routing、channel delivery、multi-agent isolation、plugin capabilities、nodes 和 security 如何一起重塑 runtime。

只要某篇文章偏离这个判断，变成“泛泛介绍模块”，就应该砍掉或重写。
