---
title: "03｜Gateway：为什么 OpenClaw 的入口不是 CLI"
status: draft
chapter: "03"
slug: "gateway"
source_files:
  - "~/workspace/openclaw/docs/concepts/architecture.md"
  - "~/workspace/openclaw/docs/concepts/agent-loop.md"
  - "~/workspace/openclaw/src/gateway/server-session-key.ts"
---

# 03｜Gateway：为什么 OpenClaw 的入口不是 CLI

如果你习惯从 Claude Code 或普通 coding agent 理解系统，很容易先问：OpenClaw 的入口是不是也应该是一条 CLI 命令？用户输入 prompt，agent 读项目文件，调用工具，然后返回结果。

OpenClaw 不是这样起步的。它当然可以有 CLI，但 CLI 不是它的第一运行边界。OpenClaw 的第一边界是 **Gateway**：一个长期运行的入口层，负责接住真实世界里的消息、控制端连接、设备节点、heartbeat、cron 等事件，再把它们转成内部 agent run。

这一点会影响后面的许多设计：OpenClaw 不只是“你敲一次命令，它答一次”。它更像一个个人 AI 运行时：外部世界先进入 Gateway，再进入 session、workspace、memory、automation 和 delivery。

## 这篇先回答什么

- 为什么 OpenClaw 的入口不是单次 CLI，而是长期运行的 Gateway；
- Gateway 在源码设计里拥有哪几类连接和事件；
- Gateway 和 Session Routing、Agent Loop、Delivery 的边界是什么。

这篇不展开 Memory、Heartbeat、Cron 的细节；这里只解释它们为什么都需要先进入同一个入口层。

## 先看一张机制图

这张图回答一个问题：真实世界里的事件，为什么要先经过 Gateway，而不是直接调用 agent loop。

```mermaid
flowchart LR
  Msg["消息渠道
WhatsApp/Telegram/Slack"] -->

## 源码锚点

- `~/workspace/openclaw/docs/concepts/architecture.md`
- `~/workspace/openclaw/docs/concepts/agent-loop.md`
- `~/workspace/openclaw/src/gateway/server-session-key.ts`
 GW["Gateway
长期入口层"]
  UI["控制端
CLI/Web/macOS"] --> GW
  Node["设备节点
node role"] --> GW
  Tick["系统事件
Heartbeat/Cron"] --> GW

  GW --> Proto["协议与身份边界
connect/auth/schema"]
  Proto --> Route["Session Routing
选择会话与 agent"]
  Route --> Run["Agent Run
组装上下文并执行"]
  Run --> Out["Reply Shaping / Delivery
渠道化输出"]
```

读这张图时，建议按这个顺序看：
- 左侧不是一种输入，而是一组外部事件来源；
- Gateway 先处理连接、认证、协议和事件类型，不等同于模型调用；
- Session Routing 决定“这件事属于哪个上下文”；
- Agent Run 完成后，输出还要再被塑造成具体渠道可接受的消息。

<!-- IMAGEGEN_PLACEHOLDER:
title: Gateway 作为第一运行边界
type: architecture
purpose: 解释为什么 OpenClaw 的入口是长期 Gateway，而不是单次 CLI 调用
prompt_seed: 生成一张 16:9 中文技术架构图，主题是 OpenClaw Gateway 作为第一运行边界。左侧是多渠道消息、控制端、设备节点、Heartbeat/Cron；中间是 Gateway，标注长期进程、协议认证、事件分发；右侧是 Session Routing、Agent Run、Delivery。少字、高对比、正式技术架构图、无 logo、无水印。
asset_target: docs/assets/03-gateway-imagegen.png
status: pending
-->

## 第一层：Gateway 是长期进程，不是一次命令

`docs/concepts/architecture.md` 对 Gateway 的描述很直接：一个长期存在的 Gateway 拥有所有 messaging surfaces，并且控制端通过 WebSocket 连到它。默认地址是 `127.0.0.1:18789`，控制端可以是 macOS app、CLI、web UI 或 automation。

这和普通 CLI agent 的入口心智不同。CLI agent 的主路径通常是：当前终端进程启动、读取当前目录、执行一次任务、退出。OpenClaw 的 Gateway 更像一个常驻服务：它维护 provider 连接，暴露 typed WebSocket API，发出 `agent`、`chat`、`presence`、`health`、`heartbeat`、`cron` 等事件。

所以，Gateway 的重点不在于“它也能转发消息”，而在于：它把 OpenClaw 从一次性工具变成了能持续接收事件的运行时。

## 第二层：Gateway 同时接住消息面、控制面和节点面

源码文档把 Gateway 的连接面分成几类：

- messaging surfaces：WhatsApp、Telegram、Slack、Discord、Signal、iMessage、WebChat 等；
- control-plane clients：CLI、web UI、macOS app、automation 等，通过 WebSocket 发请求；
- nodes：macOS/iOS/Android/headless 节点，也通过 WebSocket 连接，但声明 `role: node`，并带上设备能力和命令；
- Gateway HTTP server：同一端口还承载 canvas host 等 UI 表面。

这说明 Gateway 不是“聊天 adapter”。如果它只是 adapter，每个渠道把消息转成 prompt 就够了。但 OpenClaw 还要处理控制端、设备节点、health、presence、系统事件、agent streaming，这些都需要一个统一入口层。

## 第三层：协议边界先于 agent run

`architecture.md` 里有几个不该忽略的约束：

- WebSocket 第一帧必须是 `connect`；
- 之后才有 `{type:"req"}`、`{type:"res"}`、`{type:"event"}` 这些协议形态；
- inbound frames 会用 JSON Schema 校验；
- side-effecting methods，比如 `send` 和 `agent`，需要 idempotency keys 以支持安全重试；
- 非本地连接还涉及 auth、pairing、device identity、challenge signature。

也就是说，在模型看到任何上下文之前，OpenClaw 已经做了一层运行时入口控制：谁连进来、是什么角色、用什么协议、是否能发起副作用请求。这也是后面安全章节要展开的地方：OpenClaw 的边界不是从 shell permission 才开始，而是从 message ingress 就开始。

## 第四层：Gateway 不替代 Agent Loop，它负责把事件送进去

`docs/concepts/agent-loop.md` 把 agent loop 定义为 intake、context assembly、model inference、tool execution、streaming replies、persistence 的完整运行路径。Gateway RPC 里的 `agent` 和 `agent.wait` 是入口之一，但一次 agent run 还会继续进入 session 解析、workspace 准备、prompt 组装、工具执行、流式输出和 transcript 写入。

因此，Gateway 和 Agent Loop 的关系可以这样分：

- Gateway 负责“事件进入系统”以及协议、连接、身份、事件推送；
- Agent Loop 负责“这次运行如何执行”；
- Session Routing 负责“这次运行属于哪个上下文”；
- Reply Shaping / Delivery 负责“结果如何回到真实渠道”。

这四层如果混在一起，OpenClaw 就会退化成一个渠道脚本。分开之后，它才能同时处理聊天、控制端、节点、heartbeat、cron 和后台任务。

## 对 Claude Code 读者的迁移点

如果用 Claude Code 心智迁移，可以这么理解：

- Claude Code 的入口通常是“我在项目里发起一次对话”；
- OpenClaw 的入口是“外部世界向一个长期运行的 Gateway 发出事件”；
- Claude Code 的上下文主要由当前项目和会话驱动；
- OpenClaw 的上下文还要经过 channel、account、peer、agentId、sessionKey、workspace、memory 等运行时层。

所以本篇不是要说 Gateway 比 CLI “更高级”。更准确地说，OpenClaw 的产品形态要求它先有 Gateway。没有这个入口层，后面的 session routing、长期 workspace、heartbeat、cron 和跨渠道 delivery 都缺一个共同的运行边界。

## 小结

OpenClaw 的入口不是终端，而是 Gateway。Gateway 把真实世界事件收进来，先经过协议、身份和事件边界，再交给 session 和 agent loop。

下一篇会继续追问：事件进来了以后，OpenClaw 怎么判断它属于哪条会话、哪个用户、哪个 agent、哪份长期状态？这就是 Session Routing 的问题。

## Readability-coach 自检

- 是否回答了读者问题：是，开头先从读者容易带入的旧心智模型进入，再给出本章判断。
- 是否降低术语密度：是，第一次出现的运行时概念都尽量配了中文解释，没有把英文术语当作入口。
- 是否保留源码锚点：是，锚点集中列出，正文只引用必要机制，不做目录游览。
- 是否避免无关项目叙事：是，只使用 Claude Code / coding agent 作为读者迁移背景，没有引入无关项目关系。
