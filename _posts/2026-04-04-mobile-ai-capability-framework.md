---
layout: post
title: "移动平台 AI 能力暴露：Android 和 iOS 谁的路更对"
date: 2026-04-04
category: engineering
tags: [mobile, ai-engineering, android, ios, app-intents]
---

> 我的 APP 团队下半年要做的一件事，是让智能家居设备的能力能被系统级 AI 调用。这意味着我们必须直面一个问题：Android 的 AppFunctions 和 iOS 的 App Intents，哪条路的代价更小，长期更稳？

## 核心判断

**Android 的「双轨制」是历史包袱，iOS 的「统一框架」是未来方向。**

这不是站队，而是两条路在工程成本上的直接对比：Google 自己也没想清楚怎么把 App Actions（语音那条线）和 AppFunctions（AI 那条线）缝在一起，而 Apple 已经用 App Intents 把 Siri、Shortcuts、Apple Intelligence 统一在同一张网里了。

## Android：两套系统，两份维护成本

Android 的麻烦在于**并行**。想做语音控制？走 App Actions，要配 XML、要遵守 schema.org 的 BII 语义。想让 Gemini 调用你的功能？走 AppFunctions，要重写 Kotlin 代码、要注册 JSON Schema。

两条线的语义模型不一样，部署方式不一样，生命周期管理也不一样。对于我们这种设备控制逻辑本来就要跨平台复用的团队来说，这意味着同一套业务能力要暴露两次。

更实际的问题是：App Actions 这两年基本停滞了，Google 的重心明显在往 AppFunctions 上挪。你现在投精力去铺 App Actions，明年可能又得迁移。

## iOS：一条路走到黑，但至少是直的

iOS 16 之后的 App Intents 设计非常简洁。一个 `AppIntent` 结构体定义清楚，Siri 能用、Shortcuts 能用、Apple Intelligence 也能用。没有额外的 Extension 进程，没有两套声明文件，编译时就给了类型安全。

它的代价也很明显：**封闭**。你想给 Bixby 或小爱用？没门。想脱离 Apple 的审核和分发体系？也几乎不可能。

但对于我来说，这个封闭性是某种意义上的减负。我的团队不需要去研究不同 OEM 的兼容性，不需要维护一份开放标准的语义映射表。我们只服务 Apple 生态的用户，而他们的人口和付费能力足够支撑独立的 iOS 方案。

## 执行层的直接体感

| Android AppFunctions | iOS App Intents |
|:---|:---|
| 系统服务通过 IPC/反射调用 | 主 App 进程直接执行 |
| 异步回调或轮询 | Swift async/await |
| 受限系统 API | 完整 App 能力 |

从开发效率的角度看，iOS  side 的代码写起来更顺。AppFunctions 的权限模型、发现机制、状态轮询，都需要额外的学习和调试成本。而这个成本乘以功能点的数量，不是小数字。

## 对我们做智能家居的特殊意味

我的团队要解决的实际场景：

- **语音控制设备**：Android 用 DEVICE_CONTROL BII，iOS 用 App Intent + HomeKit。两边都必须做，没有捷径。
- **AI 场景编排**：比如"当我到家时自动开灯"。Android 需要 AppFunctions 暴露接口，iOS 的 App Intent 天然支持 Shortcuts 自动化。后者的用户体验目前明显更成熟。
- **跨品牌联动**：Matter 是唯一的出路。不管是 Android 还是 iOS，最好都在 Matter 上做一层抽象。
- **车机联动**：Android Auto 和 CarPlay 分别走各自的协议，业务层能复用的有限。

## 我的选型结论

**iOS**：尽快把核心设备控制能力和场景编排能力封装成 App Intents。路径清晰，生态托底，用户侧的回报明确。

**Android**：观望 AppFunctions 的正式版实际表现，暂时不做 App Actions 的新投入。等 Google 把两条线真正合并了再大规模迁移。

**跨平台抽象层**：共享设备控制协议和场景规则引擎，平台适配层分别实现。不要幻想用一套接口打通两个生态。

## 给开发者的落地建议

1. **业务逻辑必须完全抽象**。不管是 AppFunctions 还是 App Intents，最终调用的应该是同一套 KMP 共享逻辑。
2. **平台差异不要硬抹平**。Android 和 iOS 的权限模型、生命周期、调用方式都不同，强行统一的适配层只会越写越厚。
3. **紧盯 MCP 协议**。它有可能成为云端工具调用的统一标准，但本地能力的统一标准短期内还不会出现。

## 最后

技术选型这件事，**冷静的算术比热情的叙事更可靠**。Android 的开放口号喊了很多年，但落到工程现场，双轨制带来的维护成本是真实的。iOS 的封闭被人诟病，但对于我们的业务场景，它恰好省掉了很多不必要的决策和兼容。

这不是说 Apple 更先进，而是说**在当前的约束条件下，iOS 那边的账更容易算平**。
