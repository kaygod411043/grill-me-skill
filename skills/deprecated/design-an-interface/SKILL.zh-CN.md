---
name: design-an-interface
description: 使用并行子代理为模块生成多个截然不同的接口设计。当用户想要设计 API、探索接口方案、比较模块形态，或提到“设计两次”时使用。
---

# 设计接口

本技能基于《软件设计的哲学》中的“设计两次”：你的第一个想法不太可能是最好的。先生成多个截然不同的设计，然后进行比较。

## 工作流

### 1. 收集需求

在设计之前，先弄清楚：

- [ ] 这个模块要解决什么问题？
- [ ] 调用方是谁？（其他模块、外部用户、测试）
- [ ] 关键操作有哪些？
- [ ] 有哪些约束？（性能、兼容性、现有模式）
- [ ] 哪些内容应隐藏在内部，哪些应暴露出来？

询问：“这个模块需要做什么？谁会使用它？”

### 2. 生成设计（并行子代理）

使用 Task 工具同时启动 3 个以上的子代理。每个子代理都必须给出一种**截然不同**的方案。

```
Prompt template for each sub-agent:

Design an interface for: [module description]

Requirements: [gathered requirements]

Constraints for this design: [assign a different constraint to each agent]
- Agent 1: "Minimize method count - aim for 1-3 methods max"
- Agent 2: "Maximize flexibility - support many use cases"
- Agent 3: "Optimize for the most common case"
- Agent 4: "Take inspiration from [specific paradigm/library]"

Output format:
1. Interface signature (types/methods)
2. Usage example (how caller uses it)
3. What this design hides internally
4. Trade-offs of this approach
```

### 3. 展示设计

展示每个设计时包括：

1. **接口签名** - 类型、方法、参数
2. **使用示例** - 调用方在实践中如何实际使用它
3. **隐藏的内容** - 保留在内部的复杂度

按顺序展示各个设计，让用户在比较之前能够充分理解每种方案。

### 4. 比较设计

展示完所有设计后，从以下维度进行比较：

- **接口简洁性**：方法更少、参数更简单
- **通用与专用**：灵活性与专注度之间的取舍
- **实现效率**：这种形态是否允许高效的内部实现？
- **深度**：小接口隐藏大量复杂度（好），还是大接口配上单薄实现（差）
- **易于正确使用**与**易于误用**

用文字讨论取舍，不要使用表格。重点指出各设计分歧最大的地方。

### 5. 综合

最佳设计往往会结合多个方案中的洞见。询问：

- “哪种设计最符合你的主要用例？”
- “其他设计中是否有值得纳入的元素？”

## 评估标准

以下标准来自《软件设计的哲学》：

**接口简洁性**：方法更少、参数更简单，意味着更容易学习和正确使用。

**通用性**：无需修改即可处理未来用例。但要警惕过度泛化。

**实现效率**：接口形态是否允许高效实现？还是会迫使内部实现变得别扭？

**深度**：小接口隐藏大量复杂度，代表深模块（好）。大接口配上单薄实现，代表浅模块（应避免）。

## 反模式

- 不要让子代理产出相似的设计，要强制它们形成根本差异
- 不要跳过比较，价值就在对照之中
- 不要实现，本技能只关注接口形态
- 不要根据实现工作量来评估设计
