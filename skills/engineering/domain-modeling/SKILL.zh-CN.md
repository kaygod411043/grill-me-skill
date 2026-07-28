---
name: domain-modeling
description: 构建并打磨项目的领域模型。当用户想要明确领域术语或统一语言、记录架构决策，或者其他 skill 需要维护领域模型时使用。
---

# 领域建模

在设计过程中主动构建并打磨项目的领域模型。这是一项*主动*实践：质疑术语、构造边界场景，并在术语表述和决策明确成形的当下立即将其写下。（仅仅为了使用词汇而*读取* `CONTEXT.md` 并不属于此 skill，那只是任何 skill 都能做到的一行习惯。本 skill 用于改变模型，而不仅是使用模型。）

## 文件结构

大多数仓库只有一个上下文：

```
/
├── CONTEXT.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

如果根目录存在 `CONTEXT-MAP.md`，则仓库包含多个上下文。该映射会指明每个上下文的位置：

```
/
├── CONTEXT-MAP.md
├── docs/
│   └── adr/                          ← system-wide decisions
├── src/
│   ├── ordering/
│   │   ├── CONTEXT.md
│   │   └── docs/adr/                 ← context-specific decisions
│   └── billing/
│       ├── CONTEXT.md
│       └── docs/adr/
```

惰性创建文件，只在确实有内容可写时创建。如果 `CONTEXT.md` 不存在，在第一个术语被确定时创建它。如果 `docs/adr/` 不存在，在需要第一份 ADR 时创建它。

## 会话期间

### 对照术语表提出质疑

当用户使用的术语与 `CONTEXT.md` 中已有语言冲突时，立即指出：“你的术语表把‘cancellation’定义为 X，但你现在似乎表达的是 Y，究竟是哪一个？”

### 打磨模糊语言

当用户使用模糊或含义重叠的术语时，提出精确的规范术语：“你说的是‘account’，你指的是 Customer 还是 User？它们是两回事。”

### 讨论具体场景

讨论领域关系时，用具体场景进行压力测试。构造能够探查边界情况的场景，迫使用户精确定义概念之间的界限。

### 与代码交叉核对

当用户陈述某个机制如何工作时，检查代码是否一致。如果发现矛盾，明确指出：“你的代码会取消整个 Order，但你刚才说可以部分取消，哪一个才是正确的？”

### 即时更新 CONTEXT.md

术语一旦确定，就立即更新 `CONTEXT.md`。不要批量处理，应在术语明确时当场记录。使用 [CONTEXT-FORMAT.zh-CN.md](./CONTEXT-FORMAT.zh-CN.md) 中的格式。

`CONTEXT.md` 必须完全不包含实现细节。不要把 `CONTEXT.md` 当作规格、草稿区或实现决策仓库。它只是术语表，除此之外什么都不是。

### 谨慎提议 ADR

只有在以下三个条件全部满足时，才提议创建 ADR：

1. **难以逆转**：日后改变主意的成本很高
2. **缺少上下文时令人意外**：未来读者会疑惑“为什么要这样做？”
3. **真实权衡的结果**：确实存在多个可行方案，而你基于具体理由选择了其中一个

只要缺少任一条件，就跳过 ADR。使用 [ADR-FORMAT.zh-CN.md](./ADR-FORMAT.zh-CN.md) 中的格式。
