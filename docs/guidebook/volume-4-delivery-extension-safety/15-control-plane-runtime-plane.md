---
title: "15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载"
status: draft
chapter: "15"
slug: "control-plane-runtime-plane"
---

# 15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载

如果沿用普通 coding agent 的经验，很容易把“插件”理解成一组工具：启动时把所有插件加载进来，工具表拼好，模型需要什么就调用什么。

OpenClaw 不能这样做。它运行在长期存在的个人 AI runtime 里，不只是一个守着终端输入的工具箱：Gateway 要先启动，真实消息渠道要接入，控制端要查询状态，配置页要展示插件能力，定时任务和后台会话还会在不同时间醒来。这里的分界线是：**Control Plane 先知道“有哪些能力、如何配置、什么时候该加载”；Runtime Plane 再执行这些能力。**

## 读者问题

OpenClaw 为什么强调 manifest-first 和 runtime lazy loading？

## 本篇先给结论

OpenClaw 的插件不适合当作“启动时全量 import 的工具目录”。它更像外部世界能力的所有权边界。manifest-first 让 Gateway 可以在不执行插件代码的前提下完成发现、校验、诊断、配置 UI、setup/onboarding 和 activation planning；runtime lazy loading 则保证只有当某个 provider、channel、route、command 或 capability 被当前事件需要时，才进入插件的真实运行时。

这不只是性能优化，也是在划定个人 AI runtime 的启动边界和安全边界：控制面要轻、可解释、可诊断；运行面要晚加载、窄加载，只为当前事件加载必要能力。

## 先看一张机制图

```mermaid
flowchart TB
  subgraph CP[Control Plane：不急着执行插件代码]
    Manifest[openclaw.plugin.json] --> Discovery[发现插件与读取元数据]
    Discovery --> Validation[configSchema / auth / setup 校验]
    Validation --> Planning[Activation Planning]
    Planning --> Diagnostics[状态、诊断、配置 UI]
  end

  subgraph RP[Runtime Plane：只在需要时加载]
    Loader[Plugin Loader] --> Register[register(api)]
    Register --> Registry[Capability Registry]
    Registry --> Channel[Channel / Provider / Tool / Hook / Service]
  end

  Planning -->|provider / channel / command / route / capability 命中| Loader
  Channel --> World[真实世界消息、模型、媒体、外部服务]
```

读这张图时可以把它分成两问：上半部分回答“系统知道什么”，下半部分回答“系统执行什么”。OpenClaw 把这两个问题拆开，Gateway 刚启动时就不必把所有渠道、模型、媒体、搜索、语音、hook 和服务全部拉进进程。

<!-- IMAGEGEN_PLACEHOLDER:
title: 15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载 机制图
type: boundary-map
purpose: 用一张正式技术架构图解释“OpenClaw 为什么强调 manifest-first 和 runtime lazy loading？”
prompt_seed: 生成一张 16:9 中文技术架构图，主题是 OpenClaw 源码阅读第 15 篇：Control Plane vs Runtime Plane。画面分成上下两层：上层 Control Plane 包含 Manifest、Discovery、Validation、Activation Planning、Diagnostics；下层 Runtime Plane 包含 Loader、register(api)、Capability Registry、Channel/Provider/Tool/Hook/Service；用一条“按需加载”箭头连接两层。高对比、少量标签、无 logo、无水印，不要装饰性插画。
asset_target: docs/assets/15-control-plane-runtime-plane-imagegen.png
status: pending
-->

## 源码锚点

- `~/workspace/openclaw/docs/plugins/architecture.md`
- `~/workspace/openclaw/docs/plugins/manifest.md`
- `~/workspace/openclaw/src/plugins/loader.ts`
- `~/workspace/openclaw/src/plugins/public-surface-loader.ts`
- `~/workspace/openclaw/src/plugins/manifest-registry.ts`
- `~/workspace/openclaw/src/plugins/registry.ts`
- `~/workspace/openclaw/src/plugin-sdk/index.ts`

## 1. manifest-first：先读元数据，不先跑代码

`docs/plugins/manifest.md` 说得很直接：每个 native OpenClaw plugin 都必须有 `openclaw.plugin.json`，OpenClaw 用它在**不执行插件代码**的情况下校验配置。这个文件承载的不是运行逻辑，而是控制面元数据：插件 id、`configSchema`、provider/channel 归属、auth/onboarding 信息、setup 描述、activation hints、contracts、UI hints 等。

这解决的是 Gateway 类 runtime 的基本问题：系统如果要回答“现在有哪些插件可用”“这个插件缺什么配置”“某个 provider 的 key 从哪里来”“某个 channel 是哪个插件拥有的”，不能每次都先把插件 runtime 启动一遍。否则，配置检查本身就会变成副作用入口。

所以 manifest 的边界很清楚：

- 它能声明身份、配置 schema、auth 元数据、setup/onboarding、activation hint、静态 capability snapshot；
- 它不能注册 runtime 行为；
- 它不能替代 `register(...)`、`setupEntry` 或插件代码入口；
- 它要足够便宜，适合 Gateway、CLI、UI 和诊断工具频繁读取。

这就是 manifest-first 的第一层含义：**控制面先拿到可解释的静态地图。**

## 2. Activation Planning：不是所有插件都和当前事件有关

`docs/plugins/architecture.md` 里把插件系统拆成四层：Manifest + discovery、Enablement + validation、Runtime loading、Surface consumption。最能体现 Control Plane / Runtime Plane 分界的是 activation planning。

manifest 里的 `activation` 字段可以声明：哪些 provider、command、channel、route、capability 会触发这个插件进入加载计划。文档也强调，它不是生命周期 hook，也不替代 `register(...)`。它只是 planner metadata，用来在运行前缩小候选插件范围。

这对 OpenClaw 很重要，因为事件来源很多：

- 用户从聊天渠道发来一条消息；
- CLI 查询模型列表；
- 配置页面要展示 provider auth 状态；
- cron 到点触发一个 isolated session；
- webhook 进入 Gateway；
- channel plugin 要暴露 scoped message actions。

这些事件不应该都触发同一套“全插件加载”。一个 provider 查询只需要相关 provider 插件；一个 channel setup 只需要相关 channel 插件；一个 CLI command 只需要声明该 command 的插件。activation planning 的价值就是：先在 Control Plane 上回答“这次最多需要谁”。

## 3. loader.ts：加载意图被拆成多种模式

`src/plugins/loader.ts` 做的不是 `import all plugins`。从 `PluginLoadOptions` 就能看出，加载过程带着明确意图：

- `mode?: "full" | "validate"`：当前是完整运行，还是只做校验；
- `onlyPluginIds?: string[]`：只加载特定插件范围；
- `activate?: boolean`：是否进入 active runtime；
- `loadModules?: boolean`：是否连模块都加载，还是 manifest-only；
- `includeSetupOnlyChannelPlugins` / `forceSetupOnlyChannelPlugins`：setup 场景可以只加载轻量入口；
- `preferSetupRuntimeForChannelPlugins`：某些 channel 在启动阶段优先走 setup runtime，而不是完整 runtime。

后面的 `resolvePluginRegistrationPlan(...)` 会把这些意图转成内部执行计划：`setup-only`、`setup-runtime`、`discovery`、`full` 等不同模式会决定是否加载 setup entry、是否跑 runtime capability policy、是否执行 full activation-only registrations。

也就是说，OpenClaw 的插件加载不是一个布尔值，而是一套分层协议：

```text
manifest only
  -> validate / discovery snapshot
  -> setup-only / setup-runtime
  -> full runtime register(api)
```

这条链路对应的正是 Control Plane 到 Runtime Plane 的逐步下沉。

## 4. public-surface-loader：公共表面也要有边界

`src/plugins/public-surface-loader.ts` 处理 bundled plugin 的 public artifact 加载。它做了几件事：解析 public surface 位置、缓存模块、通过 `openBoundaryFileSync` 检查路径边界、用文件身份校验防止验证后文件被替换，再通过 jiti 或 require 加载。

这说明 OpenClaw 对“能被控制面读取的公共表面”也不会随便 import。即使只是加载 bundled plugin 的 public artifact，也要有：

- 明确的 bundle / package root 边界；
- 缓存，避免重复加载；
- boundary file read，防止越界读取；
- same-file identity 检查，避免校验后路径漂移。

这不是框架洁癖。对长期运行进程来说，一旦插件系统允许外部扩展，任何“读取元数据”都可能变成边界问题。

## 5. 为什么不能一上来全加载

现在可以回到标题的问题。插件不能一上来全加载，至少有五个原因。

第一，**启动面太重**。OpenClaw 的 Gateway 需要先作为消息、控制端、节点、cron、heartbeat 的共同入口启动。如果启动时把所有 provider/channel/media/search/hook runtime 都跑起来，启动成本和失败面都会被不相关插件放大。

第二，**诊断面会混入副作用**。配置校验、auth 状态、setup UI、命令发现，本来应该是低风险控制面动作。如果这些动作必须执行插件 runtime，诊断就不再纯粹。

第三，**不同事件需要不同能力**。一个 `models.list` 查询、一次 channel message、一个 cron run、一个 setup flow 的加载范围不同。activation planning 允许系统按事件裁剪插件范围。

第四，**插件是所有权边界，不只是工具集合**。`docs/plugins/architecture.md` 明确区分 plugin 与 capability：plugin 是 company 或 feature 的 ownership boundary；capability 是 core 的共享契约。全量加载会把 ownership 边界变成启动时耦合。

第五，**安全边界更清楚**。manifest metadata 可以被控制面读取；runtime 行为必须等到明确触发、明确范围、明确注册模式之后才进入进程。这也给下一章的安全与隔离留下落点。

## 6. 和“工具列表”的区别

普通工具列表的思路是：工具越早注册越好，模型看到完整菜单之后自己选。

OpenClaw 的思路反过来：真实世界事件先进入 Gateway；Gateway 根据 channel、session、agent、provider、route、command 和 capability 判断这次需要哪些插件表面；插件 runtime 再按需加载，把能力注册到 registry；最后由 agent run 或 delivery 层消费这些能力。

所以插件系统不能被写成“工具扩展机制”。工具只是插件能力的一种。插件还可能拥有：

- provider；
- channel；
- memory/runtime；
- media/search/speech/realtime；
- hook；
- route；
- service；
- setup/onboarding；
- diagnostics。

只有把 Control Plane 和 Runtime Plane 分开，OpenClaw 才能让这些能力既可发现、可配置，又不在启动时全部变成运行时负担。

## 小结

本章保留一句话：**manifest-first 让 OpenClaw 在不启动插件 runtime 的情况下理解世界；lazy runtime loading 让实际能力只在相关事件触发时进入运行面。**

这条边界把插件系统从“工具目录”推进到个人 AI runtime 的能力治理层。下一章讲安全时，会继续沿着这条线往下看：当真实消息、设备、节点、文件、记忆、定时任务和投递都进入同一个 runtime，安全就不再只是“命令能不能执行”。

## Readability-coach 自检

- 是否回答了读者问题：是，围绕 manifest-first 与 lazy loading 的必要性展开，没有写成源码目录游览。
- 是否有源码锚点：有，覆盖插件架构文档、manifest 文档、loader、public surface loader 和 SDK/registry 边界。
- 是否避免无关项目叙事：是，全文只把普通 coding agent 作为读者心智迁移背景。
- 是否保留一句话 takeaway：有，manifest-first 理解世界，lazy loading 按需进入运行面。
