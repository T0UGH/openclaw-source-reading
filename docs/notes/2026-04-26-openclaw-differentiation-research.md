---
title: OpenClaw 源码阅读差异化调研
date: 2026-04-26
status: draft
correction: 2026-04-26 已修正，OpenClaw 与 Pi 无关系，不再使用 Pi 宿主化叙事
tags:
  - OpenClaw
  - 源码阅读
  - CodingAgent
  - PersonalAgentGateway
---

# OpenClaw 源码阅读差异化调研

## 修正说明

本笔记最初版本误把 OpenClaw 定位为 Pi coding-agent 的 SDK 宿主化案例。这个判断已被纠正：**OpenClaw 与 Pi 没有关系**。后续 OpenClaw 源码阅读不再使用“Pi 宿主化”“嵌入 Pi agent loop”“Pi SDK integration”这条叙事。

新的判断应从 OpenClaw 自身源码与产品形态出发：

> OpenClaw 的差异点不是“把某个 coding-agent 内核包了一层”，而是它自己构建了一个面向多渠道消息、长期在线会话、插件生态、设备节点和个人助理体验的 agent gateway runtime。

---

## 这轮记录什么

准备开始写 OpenClaw 源码阅读。参照之前 Claude Code 源码阅读、Codex 源码阅读的流程，本轮先不急着动正文，而是先调研代码与文档，找出 OpenClaw 相比 Claude Code、Codex 以及一般 coding agent 的独特点。

当前第一轮修正后的结论：**OpenClaw 不是另一个 Claude Code，也不是某个 coding agent 的外壳；它更像一个长期运行的个人 AI Gateway，把 agent 能力放进多渠道消息入口、设备节点、插件系统和会话路由里。**

---

## 先给结论

OpenClaw 真正独特的地方，不在于“能不能读写代码、调 bash、跑工具”，而在于：

> 它把 agent 从终端里的单次交互工具，提升成了跨消息渠道、跨设备、跨人格、长期在线的个人助理运行时。

换句话说：

- Claude Code / Codex 的主语通常是 **代码仓库与一次 coding task**；
- 一般 coding agent 的主语通常是 **终端会话、IDE 会话或单次任务执行**；
- OpenClaw 的主语是 **长期在线的个人 AI Gateway**。

因此，OpenClaw 源码阅读不应该重复 Claude Code / Codex 的“工具调用、上下文、权限、子任务”老路线，而应该围绕：

```text
消息入口 -> Gateway -> 路由/sessionKey -> agent/session/workspace -> agent loop -> channel delivery / device action / background task
```

---

## 1. 和 Claude Code / Codex 的差异：OpenClaw 的主语不是代码仓库，而是人和消息入口

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
  -> 进入 agent loop
  -> 工具执行 / 消息回复 / 设备动作 / Canvas 更新 / 后台任务
```

README 中的关键表述：

```text
OpenClaw is a personal AI assistant you run on your own devices.
It answers you on the channels you already use.
The Gateway is just the control plane — the product is the assistant.
```

这意味着 OpenClaw 源码阅读的第一性原理应该是：

> agent 不再住在终端里，而是住在 Gateway 里。

---

## 2. 第一层独特点：Local-first Gateway

关键文档：

```text
docs/concepts/architecture.md
```

OpenClaw 的架构中心是一个长期运行的 Gateway：

```text
A single long-lived Gateway owns all messaging surfaces.
Control-plane clients connect over WebSocket.
Nodes also connect over WebSocket.
One Gateway per host.
```

支持的入口包括：

```text
WhatsApp / Telegram / Slack / Discord / Google Chat / Signal / iMessage /
BlueBubbles / IRC / Microsoft Teams / Matrix / Feishu / LINE / Mattermost /
Nextcloud Talk / Nostr / Synology Chat / Tlon / Twitch / Zalo / WeChat / QQ / WebChat
```

这和普通 coding agent 最大的区别是：

> OpenClaw 不是“打开 CLI 后开始一轮任务”，而是“一个 daemon 一直活着，所有消息渠道都可以把事件送进来”。

这一点适合作为开篇主线。

---

## 3. 第二层独特点：Session 不是 CLI 历史，而是消息来源路由状态

关键文档与源码：

```text
docs/concepts/session.md
src/routing/session-key.ts
```

OpenClaw 文档中的 session routing 规则：

```text
Direct messages -> shared session by default
Group chats     -> isolated per group
Rooms/channels  -> isolated per room
Cron jobs       -> fresh session per run
Webhooks        -> isolated per hook
```

状态位置：

```text
~/.openclaw/agents/<agentId>/sessions/sessions.json
~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl
```

OpenClaw 的 session 不是简单的终端交互历史；它是某个消息来源、某个 channel、某个 account、某个 peer、某个 group、某个 agent 的持续上下文。

所以 OpenClaw 的 `sessionKey` 值得作为重点讲：

> 它不是简单的“会话 ID”，而是把现实世界里的通信关系映射成 agent 运行上下文的路由键。

---

## 4. 第三层独特点：Multi-agent 不是 subagent，而是多人格、多账户、多工作区隔离

关键文档：

```text
docs/concepts/multi-agent.md
```

OpenClaw 的 multi-agent 指的是：一个 Gateway 里托管多个长期存在、彼此隔离的 agent/persona。

每个 agent 有自己的：

```text
workspace
agentDir
auth profiles
model registry
session store
skills
```

文档定义：

```text
An agent is a fully scoped brain with its own workspace, state directory, and session history.
```

这和 Claude Code / Codex 的 subagent 有本质区别：

| 类型 | 关注点 |
|---|---|
| Claude Code / Codex subagent | 一个任务内的分工执行 |
| OpenClaw multi-agent | 多人、多身份、多渠道、多状态隔离 |

OpenClaw 也有 subagent：

```text
src/agents/tools/subagents-tool.ts
src/agents/subagent-spawn.ts
```

但写作时要区分两层：

1. **multi-agent routing**：长期存在的多个 agent/persona；
2. **subagent spawn/control**：某个 agent 内部发起的临时协作执行。

---

## 5. 第四层独特点：安全模型来自真实消息入口，不只是本地 CLI 权限

README 强调：

```text
OpenClaw connects to real messaging surfaces.
Treat inbound DMs as untrusted input.
```

默认 DM 安全策略：

```text
unknown senders receive a pairing code
bot does not process their message
approve with openclaw pairing approve <channel> <code>
```

沙箱策略：

```text
main session 默认 host full access
non-main session 可以 sandbox
Docker / SSH / OpenShell backend
```

Claude Code / Codex 的风险主要是：

```text
本地命令执行
文件修改
权限审批
项目边界
```

OpenClaw 额外面对：

```text
远程陌生人通过消息渠道给 agent 发 prompt
群聊里的 prompt injection
不同用户共享 DM session 导致隐私泄露
channel account 被错误路由到 agent
```

因此，OpenClaw 的安全章节不能只讲 bash permission，而要讲：

> 当 agent 被放到真实通信渠道里，prompt injection 不再只是本地风险，而是入口治理、身份绑定、session 隔离和 sandbox 策略的组合问题。

---

## 6. 第五层独特点：Context Engine 是可替换的上下文治理层

关键源码：

```text
src/context-engine/types.ts
src/context-engine/registry.ts
```

`ContextEngine` 接口包括：

```ts
bootstrap(...)
maintain(...)
ingest(...)
ingestBatch(...)
afterTurn(...)
assemble(...)
compact(...)
prepareSubagentSpawn(...)
onSubagentEnded(...)
```

`assemble` 返回：

```ts
{
  messages,
  estimatedTokens,
  systemPromptAddition?
}
```

这说明 OpenClaw 把上下文治理拆成一个插件式 engine：

- 可以接管 assemble；
- 可以接管 compact；
- 可以追加 system prompt；
- 可以参与 subagent spawn；
- 可以声明自己是否 ownsCompaction；
- 还保留 sessionKey / prompt legacy 兼容层。

这和普通“历史裁剪/自动压缩”不同：

> OpenClaw 的 context 不是单纯“把历史喂给模型前裁剪一下”，而是 Gateway / agent runtime 里的一个可插拔生命周期组件。

---

## 7. 第六层独特点：Plugin 能力模型远超 coding agent extension

关键文档与源码：

```text
docs/plugins/architecture.md
src/plugin-sdk
```

OpenClaw plugin capability model 很宽：

```text
Text inference
CLI inference backend
Speech
Realtime transcription
Realtime voice
Media understanding
Image generation
Music generation
Video generation
Web fetch
Web search
Channel / messaging
Gateway discovery
```

插件加载分层：

```text
1. Manifest + discovery
2. Enablement + validation
3. Runtime loading
4. Surface consumption
```

关键边界：

```text
manifest/config validation should work without executing plugin code
```

所以 OpenClaw plugin 不是“给 coding agent 加几个工具”，而是：

> 给 Gateway 增加渠道、模型、语音、媒体、搜索、服务、CLI、HTTP route、hook、tool 的能力。

---

## 8. 第七层独特点：回复整形是聊天渠道运行时的一等能力

OpenClaw 面向真实聊天渠道，因此它不只是把模型文本打印到终端。

`docs/concepts/agent-loop.md` 中有这些机制：

```text
assistant deltas are streamed
tool events are emitted
chat channel buffers assistant deltas into chat delta messages
chat final emitted on lifecycle end/error
NO_REPLY / no_reply is filtered
messaging tool duplicates are removed
fallback tool error reply
```

这说明它有一层“回复整形”：

```text
模型输出
  -> assistant stream
  -> tool stream
  -> lifecycle stream
  -> chat delta/final
  -> duplicate suppression
  -> no_reply filter
  -> channel delivery
```

普通 coding agent 多数只需要：

```text
终端上显示模型文本和工具结果
```

OpenClaw 必须处理：

- 有些工具已经发了消息，assistant 不要再重复确认；
- 有些场景需要静默；
- 有些渠道适合 partial reply；
- reasoning 和 assistant reply 要分流；
- 工具错误需要转成用户可见 fallback；
- final payload 要和真实消息渠道适配。

这个点很适合单篇展开。

---

## 9. 第八层独特点：Node / Canvas / Voice 让它接近个人设备操作层

README 与架构文档中反复出现：

```text
Voice Wake
Talk Mode
Live Canvas
macOS/iOS/Android nodes
camera / screen / location / canvas commands
```

`docs/concepts/architecture.md` 中说明：

```text
Nodes connect over WebSocket with role: node
Expose commands like canvas.*, camera.*, screen.record, location.get
```

这说明 OpenClaw 不是纯 coding agent，而更像个人设备网络里的 agent：

> Claude Code / Codex 是“代码工作区里的 agent”。  
> OpenClaw 是“个人设备网络里的 agent”。

它可以接消息、控 Canvas、听语音、连手机节点、跑后台任务。

---

## 初步源码阅读主线

建议总题：

# OpenClaw 源码导读：从 Coding Agent 到 Personal Agent Gateway

## 第一卷：OpenClaw 到底不是哪类 agent

核心问题：

- 为什么它不是 Claude Code 平替？
- 为什么 Gateway 是第一主角？
- 为什么它的核心不是工具调用，而是消息入口与路由？

对应源码/文档：

```text
README.md
docs/concepts/architecture.md
docs/concepts/agent-loop.md
src/agents/**  # 待复核具体 agent loop 入口
```

## 第二卷：Gateway 如何把真实世界消息变成 agent run

核心问题：

- channel message 怎么进来？
- sessionKey 怎么生成？
- DM / group / cron / webhook 如何路由？
- agent.wait / agent stream / lifecycle 怎么跑？

对应源码：

```text
docs/concepts/session.md
src/routing/session-key.ts
src/auto-reply/reply/get-reply.ts
src/auto-reply/reply/get-reply-run.ts
src/agents/**  # 待复核具体 agent run 执行入口
```

## 第三卷：Agent Loop：OpenClaw 如何把 Gateway 事件变成可执行回合

核心问题：

- agent run 如何被创建、排队、执行、结束？
- session/global queue 怎么保证同一 session 串行？
- OpenClaw 如何注入 prompt、skills、tools、workspace、model？
- agent events 如何转成 OpenClaw stream？

对应源码：

```text
src/agents/**
src/auto-reply/reply/**
src/agents/system-prompt.ts
src/agents/tools/**
```

## 第四卷：Session / Context / Memory：长期在线 agent 的上下文治理

核心问题：

- session store 和 transcript 如何保存？
- context engine 怎么 assemble / compact？
- session reset / idle reset / daily reset 怎么处理？
- context engine 如何参与 subagent？

对应源码：

```text
src/context-engine/types.ts
src/context-engine/registry.ts
docs/concepts/session.md
docs/concepts/compaction.md
```

## 第五卷：Multi-agent 与 Subagent：两个完全不同的“多 agent”

核心问题：

- OpenClaw multi-agent 是 persona / account / workspace 隔离；
- subagent 是单个 agent 内部的协作执行；
- 两者不要混讲。

对应源码：

```text
docs/concepts/multi-agent.md
src/agents/subagent-spawn.ts
src/agents/tools/subagents-tool.ts
src/agents/subagent-announce-delivery.ts
src/agents/subagent-registry.ts
```

## 第六卷：Plugin / Channel / Node：OpenClaw 的生态边界

核心问题：

- plugin capability model；
- channel plugin；
- provider plugin；
- node / canvas / voice；
- why Gateway plugin != coding agent extension。

对应源码：

```text
docs/plugins/architecture.md
src/plugin-sdk
src/channels/plugins
docs/concepts/architecture.md
```

---

## 候选开篇标题

1. **OpenClaw 不是另一个 Claude Code：它把 agent 放进了个人 Gateway**
2. **从终端到消息入口：OpenClaw 如何重写 agent 的运行边界**
3. **为什么 OpenClaw 的核心不是工具调用，而是 session routing**
4. **当 agent 暴露给 WhatsApp / Telegram / Slack：OpenClaw 的安全模型为何不同**
5. **从 coding task 到 always-on assistant：OpenClaw 的源码主线应该怎么读**

当前最推荐第一篇标题：

> OpenClaw 不是另一个 Claude Code：它把 Agent 放进了个人 Gateway

---

## 下一步

继续深读下面几条主链：

```text
src/agents/**                         # 重新复核 agent loop 入口，不沿用 Pi 叙事
src/auto-reply/reply/get-reply.ts
src/routing/session-key.ts
src/context-engine/*
src/agents/subagent-spawn.ts
src/plugin-sdk/*
src/channels/plugins/*
```

目标不是马上写正文，而是先产出一份：

> OpenClaw 源码导读总纲 + 每篇源码锚点表

写作策略仍然沿用 Claude Code / Codex 源码阅读经验：**先立差异，再搭骨架，最后进入单篇源码锚定。**
