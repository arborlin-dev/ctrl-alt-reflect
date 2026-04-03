---
layout: post
title: "Vercel JSON Render vs Google A2UI 深度对比"
date: 2026-04-04
category: engineering
tags: [ai-engineering, generative-ui, vercel, google, frontend]
---

> 两大生成式 UI 方案的全面对比：架构哲学、安全模型与选型建议。

## TL;DR

Vercel JSON Render 和 Google A2UI 都是解决同一问题的两种方案：**让 AI 代理能够安全地生成动态用户界面**。两者都采用声明式 JSON 规范、支持流式传输，但在架构哲学、生态定位和安全模型上有本质差异。

| 维度 | Vercel JSON Render | Google A2UI |
|------|-------------------|-------------|
| **定位** | React/Web 生态的生成式 UI 库 | 跨平台的开放 UI 协议 |
| **核心哲学** | 组件目录 + 渐进式流式渲染 | 原生优先 + 安全沙箱 |
| **数据格式** | JSONL (JSON Lines) | JSONL (JSON Lines) |
| **组件数量** | 开发者自定义 Catalog | 18 个标准原语 |
| **渲染方式** | React 组件动态映射 | 原生组件映射 (React/Flutter/Angular/SwiftUI) |
| **安全模型** | Schema 验证 + 白名单组件 | 声明式描述 (非可执行代码) |
| **流式支持** | ✅ 渐进式渲染 | ✅ 渐进式渲染 |
| **生态集成** | Vercel AI SDK 深度集成 | AG-UI 传输协议 + A2A 代理协议 |
| **开源协议** | 未明确 | Apache 2.0 |
| **生产状态** | 实验性/早期 | Google 内部产品使用 (Opal, Gemini Enterprise) |

---

## 背景：为什么需要生成式 UI？

### 传统 AI 交互的局限

传统 AI 助手只能返回**纯文本**，当用户需要：
- 填写复杂表单
- 查看数据可视化
- 进行多步骤操作

文本对话显得力不从心，形成了所谓的 **"Chat Wall"**（聊天墙）问题。

### 生成式 UI 的解决方案

核心思想：**让 AI 代理自己决定展示什么 UI**，而非开发者预先构建所有可能的界面。

用户说"显示我最近的销售数据"，AI 判断：
1. 需要展示表格还是图表？
2. 需要哪些筛选条件？
3. 是否需要导出按钮？

然后动态生成对应的 UI 描述，客户端负责渲染。

---

## Vercel JSON Render 详解

### 核心架构：三大概念

```
┌─────────────────────────────────────────────────────────┐
│                    JSON Render 架构                      │
├─────────────┬─────────────┬─────────────────────────────┤
│   Catalog   │   Registry  │           Spec              │
│  (组件目录)  │  (注册表)    │        (JSON 规范)          │
├─────────────┼─────────────┼─────────────────────────────┤
│ 定义 AI 可   │ 连接抽象定义  │ AI 生成的 UI 描述，         │
│ 使用的组件   │ 与实际 React  │ 描述组件树、数据绑定、       │
│ 及其 Props   │ 组件实现      │ 事件处理                   │
└─────────────┴─────────────┴─────────────────────────────┘
```

**Catalog（组件目录）**
- 严格限制 AI 可用的组件词汇表
- 定义每个组件的 props、类型、事件
- 类似"组件白名单"

**Registry（注册表）**
- 将 Catalog 中的抽象定义映射到实际 React 组件
- 应用层完全控制渲染实现

**Spec（规范）**
- AI 生成的 JSON 对象
- 描述当前界面状态
- 可被增量更新（JSON Patch 风格）

### 两种生成模式

| 模式 | 输出 | 用途 | 客户端 Hook |
|------|------|------|------------|
| **Generate** | 纯 JSONL | 独立 UI 生成器、仪表板 | `useUIStream` |
| **Chat** | 文本 + JSONL | 聊天机器人、Copilot | `useChat` + `pipeJsonRender` |

### 渐进式流式渲染

JSON Render 的核心优势：**不需要等待完整响应**。

```
AI 输出流：
{"type": "PetGrid", "props": {"pets": []}}          ← 先渲染空网格
{"type": "PetCard", "props": {"name": "Buddy"}}    ← 添加第一张卡片
{"type": "PetCard", "props": {"name": "Luna"}}     ← 添加第二张卡片
```

用户立即看到界面骨架，随后数据逐步填充。

---

## Google A2UI 详解

### 核心架构：18 个原语

A2UI 采取更保守的设计：**只提供 18 个标准组件原语**。

```
┌─────────────────────────────────────────────┐
│           A2UI 18 个标准原语                 │
├─────────────┬─────────────┬─────────────────┤
│   Layout    │    Input    │    Display      │
├─────────────┼─────────────┼─────────────────┤
│ Row         │ TextField   │ Text            │
│ Column      │ CheckBox    │ Image           │
│ Card        │ DateTime    │ Table           │
│ Divider     │ Select      │ Chart           │
│             │             │                 │
├─────────────┴─────────────┴─────────────────┤
│                  Action                      │
├─────────────────────────────────────────────┤
│ Button, Link                                 │
└─────────────────────────────────────────────┘
```

### "Native-First" 原生优先哲学

与 iframe 沙箱方案不同，A2UI 采用**声明式描述而非可执行代码**：

```
传统方案（iframe）：
AI → 生成 HTML/JS → iframe 沙箱渲染 → 样式隔离、安全风险

A2UI 方案：
AI → 生成 JSON 描述 → 客户端映射到原生组件 → 继承宿主样式
```

**关键优势：**
- ✅ **安全**：不执行任意代码，只是描述 UI 结构
- ✅ **一致性**：渲染使用客户端原生组件，继承品牌样式
- ✅ **可访问性**：自动继承宿主应用的无障碍特性
- ✅ **跨平台**：同一 JSON 描述可渲染为 React/Flutter/SwiftUI

### 与 AG-UI 的关系

A2UI 经常与 **AG-UI** (Agent-User Interaction Protocol) 配合使用：

| 协议 | 解决的问题 | 类比 |
|------|-----------|------|
| **A2UI** | "渲染什么"（What to render） | HTML/CSS（内容描述） |
| **AG-UI** | "如何传输"（How to deliver） | HTTP/WebSocket（传输协议） |

AG-UI 负责：
- SSE 流式传输
- 双向状态同步
- 工具调用事件
- 生命周期管理

---

## 关键差异深度对比

### 灵活度 vs 标准化

| 方面 | Vercel JSON Render | Google A2UI |
|------|-------------------|-------------|
| **组件定义** | 开发者完全自定义 | 18 个标准原语 |
| **灵活性** | 高（任何 React 组件） | 中（组合原语） |
| **学习成本** | 需学习 Catalog API | 只需学习 18 个组件 |
| **跨平台** | React 生态为主 | Web/移动/桌面 |
| **一致性** | 依赖开发者设计 | 谷歌设计规范 |

### 安全模型对比

**Vercel JSON Render 安全：**
```typescript
// 通过 Zod Schema 验证
const catalog = createCatalog({
  components: {
    SafeButton: {
      props: z.object({
        label: z.string().max(50),  // 限制长度
        onClick: z.literal("safeAction"),  // 白名单事件
      }),
    },
  },
});
```

**Google A2UI 安全：**
```json
{
  "type": "Button",  // 只能是预定义原语
  "props": {
    "label": "确认",  // 纯文本，无代码
    "action": "user_defined_event"  // 事件名由客户端定义
  }
}
```

A2UI 更激进：**完全不传输可执行代码**，从根本上消除 XSS 风险。

### 生态定位

```
Vercel 生态栈：
┌─────────────────────────────────────────────┐
│  Vercel AI SDK                              │
│  ├─ streamText/streamObject                 │
│  ├─ useChat/useCompletion                   │
│  └─ JSON Render (UI 生成)                   │
├─────────────────────────────────────────────┤
│  Next.js + React                            │
└─────────────────────────────────────────────┘

Google 生态栈：
┌─────────────────────────────────────────────┐
│  Agent Development Kit (ADK)                │
│  ├─ A2A (Agent-to-Agent)                    │
│  ├─ A2UI (Agent-to-UI)                      │
│  └─ AG-UI (传输协议)                         │
├─────────────────────────────────────────────┤
│  Flutter / Angular / Web Components         │
└─────────────────────────────────────────────┘
```

---

## 选型建议

### 选择 Vercel JSON Render 如果你：

- ✅ 正在使用 Next.js/React 生态
- ✅ 需要深度定制的组件（如自定义图表、复杂表单）
- ✅ 希望与 Vercel AI SDK 深度集成
- ✅ 团队熟悉 React 和现代前端开发
- ✅ 项目主要是 Web 应用

### 选择 Google A2UI 如果你：

- ✅ 需要跨平台支持（Web + 移动端）
- ✅ 安全性是首要考虑（金融、医疗等场景）
- ✅ 使用 Flutter/Angular 等非 React 技术栈
- ✅ 需要与 A2A 代理协议配合使用
- ✅ 希望使用标准化组件，减少设计决策

### 混合方案

在实际架构中，两者也可以互补：

```
┌─────────────────────────────────────────────┐
│           你的应用前端                        │
│  ┌─────────────────────────────────────┐   │
│  │  A2UI Renderer (标准化组件)          │   │
│  │  - 表单、表格、图表                   │   │
│  └─────────────────────────────────────┘   │
│  ┌─────────────────────────────────────┐   │
│  │  JSON Render (自定义组件)            │   │
│  │  - 复杂业务组件、品牌定制             │   │
│  └─────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

---

## 技术趋势与思考

### 生成式 UI 的三层协议栈

正在形成的行业标准架构：

```
┌─────────────────────────────────────────────┐
│  Layer 3: UI 描述层                         │
│  - A2UI (Google)                            │
│  - JSON Render (Vercel)                     │
│  - MCP-UI (Anthropic/OpenAI)                │
├─────────────────────────────────────────────┤
│  Layer 2: 传输协议层                         │
│  - AG-UI (SSE 流式)                         │
│  - A2A (代理间通信)                          │
│  - WebSocket                                │
├─────────────────────────────────────────────┤
│  Layer 1: 工具/上下文层                      │
│  - MCP (Model Context Protocol)             │
│  - Function Calling                         │
└─────────────────────────────────────────────┘
```

### 关键洞察

1. **安全优先**：A2UI 的"无代码执行"设计可能成为企业级应用的默认选择

2. **标准之争**：目前各厂商（Google、Vercel、Anthropic、OpenAI）都在推出自己的 UI 协议，统一标准尚需时日

3. **渐进式采用**：可以从简单场景（如表单生成）开始，逐步扩展

4. **人机协作**：生成式 UI 不是要替代设计师，而是让 AI 处理"该展示什么"，人类处理"如何呈现"

---

## 参考资源

### Vercel JSON Render
- [官方文档](https://json-render.dev/)
- [GitHub 仓库](https://github.com/vercel/json-render)

### Google A2UI
- [官方文档](https://www.atoui.org/)
- [Google 开发者博客](https://developers.googleblog.com/introducing-a2ui-an-open-project-for-agent-driven-interfaces/)
- [在线 Playground](https://atoui.org/playground)

### 相关协议
- [AG-UI (传输协议)](https://docs.ag-ui.com/)
- [A2A (代理间协议)](https://github.com/google/A2A)
- [MCP (Model Context Protocol)](https://modelcontextprotocol.io/)

---

*这篇文章整理自对 Vercel JSON Render 和 Google A2UI 的深度研究和对比分析，适用于正在评估生成式 UI 方案的技术团队。*
