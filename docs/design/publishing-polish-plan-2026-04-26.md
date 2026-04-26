# OpenClaw 源码阅读小书发布态阅读清理计划

> 记录时间：2026-04-26
> 目标：把当前 GitHub Pages 从“完成度很高的草稿站点”清理成更像正式读物的公开站点。

## 总判断

当前站点技术发布正常，源码事实与章节契约基本稳；主要问题是阅读体验：读者连续阅读时会被重复定位、双图堆叠和作者自检痕迹打断。

## P0：发布态必须先清理

1. **首页聚焦**
   - 首屏突出一个主入口：开始阅读。
   - 保留四卷总览与 2-3 条阅读路线，但降低解释密度。
   - 避免首页同时承担完整介绍、全目录、路线、状态说明等过多任务。

2. **作者自检改为读者检查点**
   - 当前 17 篇都有 `## Readability-coach 自检`，正式读者会出戏。
   - 改成面向读者的 `## 本章检查点`。
   - 句式从“本文是否……”改为“读完你应该能区分 / 回答……”。

3. **双图降噪**
   - 当前每篇都有 Mermaid + imagegen 图，连续读会重复。
   - 保留 Mermaid 作为机制图。
   - imagegen 配图改成折叠的“配图”块，保留资产与引用，但不打断正文主线。

## P1：全书阅读节奏优化

4. **第一卷压缩重复定位**
   - 00-01 负责纠偏：OpenClaw 不是普通 coding agent，而是个人 AI runtime。
   - 02 起减少“不是 prompt / 不是 CLI”的重复起步，直接进入事件分类、Gateway 边界、sessionKey 示例。

5. **05/06 去重，重构 06**
   - 05 只回答：长期状态放在哪里，workspace 与 agent state dir 如何分工。
   - 06 只回答：memory 如何变成 runtime 能力。
   - 06 降低 backend / wiki / publicArtifacts 等扩展细节权重，收束为四层：事实来源、检索访问、回复使用、整理晋升。

6. **第三卷加统一框架**
   - 在 11 章加入自动化四维框架：trigger / execution context / record ledger / delivery。
   - 减少 Heartbeat / Cron / Tasks 之间反复“不是 X”的防守式表述。

7. **第四卷加最终闭环**
   - 在 16 章结尾补一个跨 13-16 的真实消息闭环例子。
   - 把 Ingress → Control Plane → Runtime Plane → Reply Shaping → Channel Delivery → Safety closure 压成读者能复述的模型。

## P2：后续增强

8. 左侧导航标题继续压短，正文 H1 保留完整标题。
9. 源码锚点分成“主锚点 / 深挖锚点”，后续可补 GitHub 链接。
10. 统一术语大小写和章节结尾风格。

## 本轮执行范围

本轮先执行 P0 + 关键 P1：

- 首页聚焦；
- 17 篇自检改读者检查点；
- imagegen 图折叠降噪；
- 02/03/04 局部压缩重复定位并补强具体表；
- 06 重构为四层总览；
- 11 加自动化四维框架；
- 16 加最终闭环例子。

## 验证要求

- `python3 -m mkdocs build --strict`
- 检查 17 篇仍包含 Mermaid、imagegen asset 引用、源码锚点、本章检查点。
- `git diff --check`
- push 后确认 GitHub Actions Pages 成功。
