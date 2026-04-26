# 章节卡片：15｜Control Plane vs Runtime Plane：为什么插件不能一上来就全加载

## 核心问题

OpenClaw 为什么强调 manifest-first 和 runtime lazy loading？

## 文章角色

待写作时根据 `docs/design/book-architecture-v0.3.md` 复核。本篇属于 `volume-4-delivery-extension-safety`。

## 源码锚点

- `~/workspace/openclaw/docs/plugins/architecture.md`
- `~/workspace/openclaw/src/plugins/loader.ts`
- `~/workspace/openclaw/src/plugins/public-surface-loader.ts`

## 图文要求

- 正文必须保留至少一张 Mermaid 图。
- 正文必须保留 `IMAGEGEN_PLACEHOLDER`。
- 完稿前必须走可读性教练检查。

## 设计文档摘录

```text
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
```
