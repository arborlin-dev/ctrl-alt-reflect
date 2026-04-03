---
layout: post
title: "移动平台 AI 能力暴露框架对比"
date: 2026-04-04
category: engineering
tags: [mobile, ai-engineering, android, ios, app-intents]
---

> Android App Actions + AppFunctions vs iOS App Intents：架构哲学、生态定位与选型建议。

## 核心结论

### 两大平台策略对比

| 维度 | Android (App Actions + AppFunctions) | iOS (App Intents) |
|:---|:---|:---|
| **核心哲学** | Web 语义化 (schema.org) + 本地功能暴露 | 原生 Swift 框架 + 系统级深度集成 |
| **意图定义** | XML 声明 (actions.xml) + schema.org BII | Swift 代码定义 (`AppIntent` protocol) |
| **功能暴露** | **双轨制**：App Actions(语音) + AppFunctions(AI 调用) | **统一框架**：App Intents 覆盖所有场景 |
| **AI 集成** | Android 16 AppFunctions = 类 MCP 本地版 | Apple Intelligence 直接消费 App Intents |
| **跨 App 调用** | A2A 协议 + AppFunctions 编排 | App Intent Domains (预计 iOS 26) |
| **开发者成本** | 较低（声明式配置） | 较高（需要 Swift 代码实现） |
| **生态系统** | 开放但碎片化（OEM 差异） | 封闭但统一（Apple 全平台一致） |

### 关键判断

> **Android 的「双轨制」是历史包袱，iOS 的「统一框架」是未来方向。**

Google 正在努力融合 App Actions 和 AppFunctions，而 Apple 已经在 App Intents 上实现了完全统一。

---

## Android 平台：双轨架构

### App Actions：语音优先的语义层

**架构流程**：

```
用户语音："订一份披萨"
       ↓
Google Assistant 语义解析
       ↓
schema.org/BII 匹配 (Built-in Intent)
       ↓
actions.xml 映射 → 提取参数
       ↓
触发 App 的特定 Activity/Slice
```

**配置示例**：

```xml
<!-- res/xml/actions.xml -->
<actions>
  <action intentName="actions.intent.ORDER_FOOD">
    <fulfillment urlTemplate="https://example.com/order{?food,quantity}">
      <parameter-mapping 
        intentParameter="food.name" 
        urlParameter="food" />
    </fulfillment>
  </action>
</actions>
```

**核心特点**：
- 基于 **schema.org** 标准化语义（与 Web SEO 同构）
- **BII (Built-in Intents)**：预定义 100+ 通用意图
- 在线语义解析，依赖 Google 服务

### AppFunctions：AI 时代的功能暴露层

**定位**：Android 16 引入的「类 MCP 本地版」，让 AI Agent 能直接调用 App 功能。

**架构图**：

```
┌─────────────────────────────────────────────────────────┐
│                    AppFunctions 架构                     │
├─────────────────────────────────────────────────────────┤
│  App A (日历)        App B (外卖)        App C (出行)    │
│     │                   │                   │            │
│     ▼                   ▼                   ▼            │
│  Function            Function            Function        │
│  -create_event       -order_food       -book_ride        │
│     │                   │                   │            │
│     └───────────────────┼───────────────────┘            │
│                         ▼                                │
│              ┌─────────────────────┐                     │
│              │   AppFunctionManager │                    │
│              │     (系统级服务)      │                    │
│              └──────────┬──────────┘                     │
│                         ▼                                │
│              ┌─────────────────────┐                     │
│              │  Gemini / 第三方 AI  │                    │
│              └─────────────────────┘                     │
└─────────────────────────────────────────────────────────┘
```

**代码示例**：

```kotlin
// 定义 AppFunction
class CreateEventFunction : AppFunction(
    schema = "createEvent",
    description = "Create a calendar event"
) {
    override suspend fun invoke(params: CreateEventParams): EventResult {
        return calendarRepository.createEvent(
            title = params.title,
            time = params.parseTime(),
            location = params.location
        )
    }
}

// AndroidManifest.xml 声明
<app-function
    android:name=".functions.CreateEventFunction"
    android:exported="true" />
```

**关键设计**：

| 特性 | 说明 |
|:---|:---|
| **权限控制** | `EXECUTE_APP_FUNCTIONS` vs `EXECUTE_APP_FUNCTIONS_TRUSTED` |
| **发现机制** | 集成 Android App Search，可被系统索引 |
| **异步支持** | 支持长时间运行的任务，Agent 可轮询状态 |
| **参数序列化** | JSON Schema 定义，自动生成类型安全代码 |

### 双轨制的协同与割裂

```
App Actions (语音)              AppFunctions (AI)
───────────────────            ───────────────────
schema.org 语义层                 Kotlin/Java 代码层
Google Assistant 独占             Gemini + 第三方 Agent
在线语义解析                      本地函数调用
只能触发浅层操作                  可执行复杂业务逻辑
──────────────                  ─────────────────
     │                                  │
     └────────── 未来融合？ ─────────────┘
              (Android 17+?)
```

**当前问题**：
- 两条线并行，开发者需要分别实现
- App Actions 功能长期停滞（Google 重心转向 AppFunctions）
- 语音交互与 AI Agent 交互体验不统一

---

## iOS 平台：统一框架

### 架构演进

```
SiriKit (iOS 10-15)            App Intents (iOS 16+)
───────────────────            ───────────────────
Intents Extension                主 App 直接实现
单独进程，启动延迟               原生 Swift 代码
Info.plist 静态声明              @AppIntent 宏注解
仅限 Siri                       Siri + Shortcuts + Spotlight
                                + Widgets + Control Center
                                + Apple Intelligence
```

### 核心架构

**代码示例**：

```swift
// 定义 Intent
struct OrderFoodIntent: AppIntent {
    // 元数据
    static var title: LocalizedStringResource = "Order Food"
    static var description = IntentDescription("Order food from nearby restaurants")
    
    // 参数
    @Parameter(title: "Food Type")
    var foodType: String?
    
    @Parameter(title: "Quantity", default: 1)
    var quantity: Int
    
    // 执行逻辑
    @MainActor
    func perform() async throws -> some IntentResult {
        let order = try await FoodService.shared.order(
            type: foodType ?? "pizza",
            quantity: quantity
        )
        
        return .result(
            value: order.id,
            dialog: "Ordered \(quantity) \(foodType ?? "pizza") for you!"
        )
    }
}

// 快捷方式声明
struct FoodShortcuts: AppShortcutsProvider {
    static var appShortcuts: [AppShortcut] {
        AppShortcut(
            intent: OrderFoodIntent(),
            phrases: [
                "Order \(.applicationName) a \($foodType)",
                "Get \($quantity) \($foodType) from \(.applicationName)"
            ]
        )
    }
}
```

### 生态系统整合

```
┌─────────────────────────────────────────────────────────┐
│                 App Intents 生态整合                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐     │
│   │  Siri  │ │ Shortcuts│ │ Spotlight│ │ Widgets  │     │
│   └───┬────┘ └────┬─────┘ └────┬─────┘ └────┬─────┘     │
│       │           │            │            │           │
│       └───────────┴─────┬──────┴────────────┘           │
│                         ▼                               │
│              ┌─────────────────────┐                    │
│              │    App Intents      │                    │
│              │     Framework       │                    │
│              └──────────┬──────────┘                    │
│                         ▼                               │
│              ┌─────────────────────┐                    │
│              │   Your App Logic    │                    │
│              └─────────────────────┘                    │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Apple Intelligence 时代的演进

**当前 (iOS 18-19)**：
- App Intents 被 Apple Intelligence 消费
- Siri 可以串联多个 Intents 完成复杂任务
- 屏幕内容感知 (On-screen awareness)

**未来 (iOS 26 预测)**：
- **App Intent Domains**：结构化分类（消息、文档、媒体、金融等）
- AI 自动发现和编排 Intents
- 自然语言直接创建 Shortcuts

---

## 架构深度对比

### 语义层对比

| | Android | iOS |
|:---|:---|:---|
| **标准** | schema.org (Web 标准) | 原生 Swift 类型系统 |
| **优势** | 跨平台语义互操作 | 编译时类型安全 |
| **劣势** | 语义解析在线依赖 Google | Apple 生态封闭 |
| **示例** | `actions.intent.ORDER_FOOD` | `OrderFoodIntent.self` |

### 执行层对比

| | Android AppFunctions | iOS App Intents |
|:---|:---|:---|
| **运行时** | 系统服务 (AppFunctionManager) | 主 App 进程 |
| **权限模型** | 声明式权限 (Manifest) | 动态权限 + 用户授权 |
| **调用方式** | 反射/IPC 调用 | 直接 Swift 方法调用 |
| **错误处理** | 异步回调/轮询 | Swift async/await |
| **沙箱** | 受限系统 API | 完整 App 能力 |

### 生态开放度对比

```
Android 生态                          iOS 生态
───────────────                      ───────────────
开放标准 (schema.org)                  私有框架
多 OEM 实现差异                        Apple 统一控制
Gemini / Bixby / 小爱                   仅限 Siri/Apple Intelligence
第三方 Agent 可接入                    仅限 Apple 授权
─────────────────                    ─────────────────
优势：灵活、创新快                      优势：体验一致、隐私可控
劣势：碎片化、质量参差                  劣势：封闭、创新受限
```

---

## 对智能家居/IoT的特殊意义

### 场景映射

| 场景 | Android 方案 | iOS 方案 | 建议策略 |
|:---|:---|:---|:---|
| **语音控制设备** | App Actions + DEVICE_CONTROL BII | App Intent + HomeKit | 同时支持 Google Home + HomeKit |
| **AI 场景编排** | AppFunctions 暴露场景接口 | App Intents 定义自动化 | 封装统一能力层 |
| **跨品牌联动** | A2A 协议成为关键 | Matter + HomeKit | 优先 Matter 标准 |
| **车机联动** | Android Auto AppFunctions | CarPlay App Intents | 分别实现，业务层复用 |

### 架构建议

```
┌─────────────────────────────────────────────────────────┐
│                    智能家居 App 架构                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   ┌─────────────────────────────────────────────────┐   │
│   │              业务逻辑层 (共享)                      │   │
│   │   • 设备控制 API                                   │   │
│   │   • 场景编排引擎                                   │   │
│   │   • 自动化规则                                     │   │
│   └──────────────────────┬──────────────────────────┘   │
│                          │                               │
│           ┌──────────────┼──────────────┐               │
│           ▼              ▼              ▼               │
│   ┌──────────────┐ ┌──────────┐ ┌──────────────┐       │
│   │ AppFunctions │ │ 传统 UI  │ │ App Intents  │       │
│   │  (Android)   │ │   层     │ │    (iOS)     │       │
│   └──────────────┘ └──────────┘ └──────────────┘       │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 技术选型建议

### 平台选择

**选择 Android AppFunctions 如果你：**
- 需要支持多 OEM 的 AI 助手（Gemini、Bixby、小爱）
- 希望与 Web 生态保持语义一致性
- 需要更细粒度的权限控制

**选择 iOS App Intents 如果你：**
- 追求开发效率和类型安全
- 深度整合 Apple 生态（Watch、Mac、Vision Pro）
- 依赖 Apple Intelligence 的端侧 AI 能力

### 迁移路径

**现有 Android App**：
```kotlin
// 阶段 1：保持 App Actions 兼容
// 新增 AppFunctions 支持 AI 场景

// 阶段 2：统一封装
abstract class AppCapability {
    abstract fun execute(params: Params): Result
}

class DeviceControlCapability : AppCapability() {
    override fun execute(params: ControlParams): ControlResult {
        // 核心业务逻辑
    }
}
```

**现有 iOS App**：
```swift
// 阶段 1：识别核心功能
// 列出所有可能用语音/AI 触发的功能

// 阶段 2：封装为 AppIntent
// 每个核心功能一个 Intent 结构体

// 阶段 3：接入 Apple Intelligence
// 系统自动发现，无需额外工作
```

---

## 未来趋势预测

### 协议融合趋势

```
当前状态                          未来趋势 (2027+)
────────────                      ────────────────
Android:                          统一 Agent 协议?
  - App Actions (schema.org)           │
  - AppFunctions (JSON Schema)         │
  - A2A (Agent 间通信)                 ▼
                              ┌──────────────┐
iOS:                          │  Universal   │
  - App Intents (Swift)       │   Agent      │
  - MCP (云端工具)             │   Protocol   │
                              └──────────────┘
```

### 关键里程碑

| 时间 | Android | iOS |
|:---|:---|:---|
| **2025 Q2** | Android 16 正式发布，AppFunctions GA | iOS 19 Apple Intelligence 扩展 |
| **2026** | App Actions + AppFunctions 融合启动 | iOS 26 App Intent Domains |
| **2027+** | 统一 Agent 框架？ | 自然语言 Shortcuts 创建 |

### 对开发者的终极建议

1. **业务逻辑完全抽象**，与平台解耦
2. **平台适配层分别实现**，但保持语义一致
3. **关注 MCP 协议**，它可能成为跨平台的统一标准

---

## 参考资源

### 官方文档
- [Android App Actions](https://developer.android.com/reference/app-actions/built-in-intents)
- [Android AppFunctions](https://developer.android.com/ai/appfunctions)
- [Apple App Intents](https://developer.apple.com/documentation/appintents)

### 关联分析
- [Vercel JSON Render vs Google A2UI](/engineering/2026/04/04/vercel-json-render-vs-google-a2ui/) - UI 渲染层对比

---

*这篇文章整理自对 Android AppFunctions 和 iOS App Intents 的深度研究，特别适用于智能家居和 IoT 领域的移动开发团队。*
