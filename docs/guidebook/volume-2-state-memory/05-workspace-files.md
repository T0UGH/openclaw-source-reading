---
title: "05｜Workspace Files：OpenClaw 的长期状态放在哪里"
status: draft
chapter: "05"
slug: "workspace-files"
source_files:
  - "~/workspace/openclaw/docs/concepts/multi-agent.md"
  - "~/workspace/openclaw/docs/concepts/memory.md"
  - "~/workspace/openclaw/src/agents/workspace.ts"
  - "~/workspace/openclaw/src/memory/root-memory-files.ts"
  - "~/workspace/openclaw/src/config/sessions/paths.ts"
---

# 05｜Workspace Files：OpenClaw 的长期状态放在哪里

如果把 OpenClaw 的 workspace 直接等同于 repo root，就会漏掉它的设计用意。对 coding agent 来说，repo root 通常回答“代码在哪里”。在 OpenClaw 里，workspace 更像 agent 的长期生活空间：规则、身份、用户信息、记忆、heartbeat 清单、会话痕迹和自动化状态，都会在这里，或相邻的 agent state 目录里留下痕迹。

这篇承接前面的“入口与路由”，转向“长期状态”。前面几篇解释事件怎么进来、怎么选 session；从这一篇开始，我们看 OpenClaw 如何让一个 agent 在回答之外，还能持续记住、醒来、执行计划、整理经验。

## 这篇先回答什么

- 为什么不能把 OpenClaw 的 workspace 当成普通 repo root；
- 哪些文件构成 agent 的长期可读状态；
- workspace、agentDir、sessions、memory 之间怎么分工。

这篇先讲文件地图和边界，不深入展开 Memory、Active Memory、Dreaming；这些会在后面三篇单独处理。

## 先看一张机制图

这张图回答一个问题：OpenClaw 的长期状态到底分布在哪些层。

```mermaid
flowchart TB
  subgraph WS["Agent Workspace"]
    A["AGENTS.md
长期规则"]
    U["USER.md / IDENTITY.md
身份与用户信息"]
    H["HEARTBEAT.md
周期检查清单"]
    M["MEMORY.md
长期事实"]
    D["memory/YYYY-MM-DD.md
每日上下文"]
    R["DREAMS.md
记忆整理回顾"]
  end

  subgraph State["Agent State Dir"]
    S["sessions/sessions.json
会话索引"]
    T["sessions/*.jsonl
会话 transcript"]
    C["cron/jobs*.json
定时任务状态"]
  end

  WS -->

## 源码锚点

- `~/workspace/openclaw/docs/concepts/multi-agent.md`
- `~/workspace/openclaw/docs/concepts/memory.md`
- `~/workspace/openclaw/src/agents/workspace.ts`
- `~/workspace/openclaw/src/memory/root-memory-files.ts`
- `~/workspace/openclaw/src/config/sessions/paths.ts`
 Run["Agent Run
启动时读取/运行时写入"]
  State --> Run
  Run --> WS
  Run --> State
```

读这张图时，建议按这个顺序看：
- 左边 workspace 是 agent 能直接读写、能被人类维护的长期文本空间；
- 右边 agent state dir 更偏运行时索引、transcript 和调度状态；
- agent run 每次启动都从这些文件和状态里恢复语境，而不是从空白开始；
- Memory、Heartbeat、Cron 不是孤立功能，它们都依赖这个长期状态面。

<!-- IMAGEGEN_PLACEHOLDER:
title: OpenClaw Workspace 长期状态地图
type: memory-map
purpose: 解释 workspace、agentDir、sessions、memory 文件如何共同构成 OpenClaw 的长期状态
prompt_seed: 生成一张 16:9 中文技术架构图，主题是 OpenClaw Workspace 长期状态地图。分成 Agent Workspace 与 Agent State Dir 两块，标出 AGENTS.md、USER.md、HEARTBEAT.md、MEMORY.md、memory/YYYY-MM-DD.md、DREAMS.md、sessions、cron 状态，并连接到 Agent Run。少字、高对比、无 logo、无水印。
asset_target: docs/assets/05-workspace-files-imagegen.png
status: pending
-->

## 第一层：workspace 是默认 cwd，也承载长期材料

`docs/concepts/multi-agent.md` 先给出边界：每个 agent 都有自己的 workspace、state directory 和 session history。它还提醒：workspace 是默认 cwd，不是硬 sandbox；相对路径会落在 workspace 里，但绝对路径仍可能访问主机其他位置，除非启用 sandboxing。

这说明 workspace 有两层含义：

1. 对 agent run 来说，它是默认工作目录；
2. 对 OpenClaw runtime 来说，它是 agent 长期材料的根目录。

第一层和 coding agent 很像，第二层就拉开了差异。OpenClaw 的 workspace 除了代码，还放这个 agent 如何行动、如何认识用户、如何记住事实、如何周期性醒来的材料。

## 第二层：启动文件是 agent 的长期行为面

`src/agents/workspace.ts` 定义了一组默认 bootstrap 文件：

- `AGENTS.md`：长期操作规则；
- `SOUL.md`：persona / 风格材料；
- `TOOLS.md`：工具使用说明；
- `IDENTITY.md`：agent 自身身份；
- `USER.md`：用户信息；
- `HEARTBEAT.md`：heartbeat 醒来时读取的检查清单；
- `BOOTSTRAP.md`：初始化/引导材料；
- `MEMORY.md`：根长期记忆文件。

`ensureAgentWorkspace()` 会在需要时创建这些文件；`loadWorkspaceBootstrapFiles()` 会从 workspace 里读取它们，作为 agent run 的启动材料之一。源码里还用 `openBoundaryFile` 做边界读取，并限制 bootstrap 文件大小，避免随便跨出 workspace root 或读入过大的文件。

所以，这些 Markdown 文件不是“文档顺手放这里”。它们是 OpenClaw 把长期行为、身份、用户资料和记忆显式化的方式。

## 第三层：Memory 是文件优先，不是隐藏状态

`docs/concepts/memory.md` 开头说得很直：OpenClaw 通过在 agent workspace 里写普通 Markdown 文件来记住事情，模型只会“记得”保存到磁盘的东西，没有隐藏状态。

其中几个文件分工不同：

- `MEMORY.md`：长期事实、偏好和决策；
- `memory/YYYY-MM-DD.md`：每日笔记，保存短期运行上下文和观察；
- `DREAMS.md`：dreaming sweep 和长期记忆晋升的 review 表面。

`src/memory/root-memory-files.ts` 进一步把 `MEMORY.md` 作为 canonical root memory filename，并保留对旧 `memory.md` 的修复路径。这说明 OpenClaw 对根记忆文件有明确规范，而不是让 agent 随意散落状态。

这也解释了为什么 workspace 像“长期生活空间”：记忆不是外部黑盒，也不是模型缓存，它是一组可读、可审查、可维护的文件。

## 第四层：sessions 和 agentDir 保存运行时索引

workspace 放可读材料，但 OpenClaw 的长期状态不只在 workspace 里。`docs/concepts/multi-agent.md` 把 `agentDir` 定义为每个 agent 的 on-disk state directory，默认在：

```text
~/.openclaw/agents/<agentId>/agent
```

Session store 则在：

```text
~/.openclaw/agents/<agentId>/sessions/sessions.json
~/.openclaw/agents/<agentId>/sessions/<sessionId>.jsonl
```

`src/config/sessions/paths.ts` 也把这个路径结构固化在源码里。这里要区分两类状态：

- workspace 文件：更适合人类和 agent 共同维护，比如规则、身份、记忆、heartbeat 清单；
- agent state/session 文件：更适合 runtime 管理，比如 session 索引、transcript、auth profile、调度状态。

两者合起来，才是 OpenClaw 的长期状态面。

## 第五层：多 agent 时，workspace 也是隔离单位

多 agent 文档还强调：一个 agent 是一个 fully scoped brain，包含 workspace、agentDir、session store、auth profiles 和模型配置。不同 agent 可以拥有不同电话号码/账号、不同 persona、不同 auth/session。

理解 workspace 时，要把这层隔离一起算进去。它既是“某个项目目录”，也是 agent 的身份边界之一。一个家庭助理、一个工作助理、一个社交助理，应该有不同 workspace 和 session store。否则规则、记忆、凭据和会话都会混在一起。

所以 workspace 在 OpenClaw 里同时承担三件事：

- 行为入口：agent 启动时读哪些长期规则；
- 记忆表面：长期事实和每日上下文写在哪里；
- 隔离边界：多 agent 场景下，哪个 brain 拥有哪些文件和状态。

## 对 Claude Code 读者的迁移点

如果你习惯 Claude Code，可以先把 workspace 类比成 repo root，但要立刻补上两点：

1. repo root 主要回答“代码在哪里”；
2. OpenClaw workspace 还回答“这个 agent 是谁、认识谁、记得什么、醒来检查什么、长期如何行动”。

因此，读 OpenClaw 的 workspace，不要只看路径解析。要看它如何把长期状态文件化：`AGENTS.md`、`USER.md`、`HEARTBEAT.md`、`MEMORY.md`、`memory/YYYY-MM-DD.md`、`DREAMS.md`、sessions 和 cron 状态，组成了一个个人 AI runtime 的生活空间。

## 小结

OpenClaw 的 workspace 不等同于普通 repo root，它更像 agent 的长期生活空间。规则、身份、用户信息、记忆、heartbeat、session 和自动化状态都围绕它组织。

下一篇开始进入这套长期状态里最成体系的一组机制：Memory。先看总览，再看 Active Memory，最后看 Flush 和 Dreaming 如何防止长期运行中的上下文流失。

## Readability-coach 自检

- 是否回答了读者问题：是，开头先从读者容易带入的旧心智模型进入，再给出本章判断。
- 是否降低术语密度：是，第一次出现的运行时概念都尽量配了中文解释，没有把英文术语当作入口。
- 是否保留源码锚点：是，锚点集中列出，正文只引用必要机制，不做目录游览。
- 是否避免无关项目叙事：是，只使用 Claude Code / coding agent 作为读者迁移背景，没有引入无关项目关系。
