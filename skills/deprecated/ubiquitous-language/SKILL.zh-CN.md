---
name: ubiquitous-language
description: 从当前对话中提取 DDD 风格的通用语言词汇表，标记歧义并提出规范术语。保存到 UBIQUITOUS_LANGUAGE.md。当用户想要定义领域术语、构建词汇表、强化术语体系、创建通用语言，或提到“领域模型”或“DDD”时使用。
disable-model-invocation: true
---

# 通用语言

从当前对话中提取领域术语并将其正式化为一致的词汇表，保存到本地文件。

## 流程

1. **扫描对话**，找出与领域相关的名词、动词和概念
2. **识别问题**：
   - 同一个词用于不同概念（歧义）
   - 不同的词用于同一个概念（同义词）
   - 含糊或含义过载的术语
3. **提出规范词汇表**，明确选择推荐术语
4. **写入工作目录下的 `UBIQUITOUS_LANGUAGE.md`**，使用下方格式
5. **在对话中直接输出摘要**

## 输出格式

写入一个具有以下结构的 `UBIQUITOUS_LANGUAGE.md` 文件：

```md
# Ubiquitous Language

## Order lifecycle

| Term        | Definition                                              | Aliases to avoid      |
| ----------- | ------------------------------------------------------- | --------------------- |
| **Order**   | A customer's request to purchase one or more items      | Purchase, transaction |
| **Invoice** | A request for payment sent to a customer after delivery | Bill, payment request |

## People

| Term         | Definition                                  | Aliases to avoid       |
| ------------ | ------------------------------------------- | ---------------------- |
| **Customer** | A person or organization that places orders | Client, buyer, account |
| **User**     | An authentication identity in the system    | Login, account         |

## Relationships

- An **Invoice** belongs to exactly one **Customer**
- An **Order** produces one or more **Invoices**

## Example dialogue

> **Dev:** "When a **Customer** places an **Order**, do we create the **Invoice** immediately?"
> **Domain expert:** "No — an **Invoice** is only generated once a **Fulfillment** is confirmed. A single **Order** can produce multiple **Invoices** if items ship in separate **Shipments**."
> **Dev:** "So if a **Shipment** is cancelled before dispatch, no **Invoice** exists for it?"
> **Domain expert:** "Exactly. The **Invoice** lifecycle is tied to the **Fulfillment**, not the **Order**."

## Flagged ambiguities

- "account" was used to mean both **Customer** and **User** — these are distinct concepts: a **Customer** places orders, while a **User** is an authentication identity that may or may not represent a **Customer**.
```

## 规则

- **明确表态。** 当同一个概念存在多个说法时，选出最合适的一个，并将其他说法列为应避免的别名。
- **明确标记冲突。** 如果某个术语在对话中存在歧义，要在 `Flagged ambiguities` 一节指出，并给出清晰建议。
- **只收录与领域专家相关的术语。** 跳过模块名或类名，除非它们在领域语言中具有含义。
- **保持定义紧凑。** 最多一句话。定义它**是什么**，而不是它做什么。
- **展示关系。** 使用粗体术语名称，并在显而易见时表达基数关系。
- **只收录领域术语。** 跳过通用编程概念（数组、函数、端点），除非它们具有领域特定含义。
- 当自然形成不同分组时，**将术语分到多个表格中**（例如按子领域、生命周期或参与者分组）。每组使用自己的标题和表格。如果所有术语都属于同一个内聚领域，一个表格就足够，不要强行分组。
- **编写一段示例对话。** 由开发者和领域专家进行 3 到 5 轮简短交流，展示这些术语如何自然地相互作用。对话应澄清相关概念之间的边界，并展示术语的精确用法。

<example>

## 示例对话

> **开发者：**“如何在没有 Docker 的情况下测试**同步服务**？”

> **领域专家：**“提供**文件系统层**而不是 **Docker 层**。它实现同一个**沙箱服务**接口，但使用本地目录作为**沙箱**。”

> **开发者：**“那么 **sync-in** 仍然会创建一个**包**并解压吗？”

> **领域专家：**“没错。**同步服务**并不知道自己正在与哪个层通信。它会调用 `exec` 和 `copyIn`，而**文件系统层**只是将它们作为本地 shell 命令运行。”

</example>

## 再次运行

在同一对话中再次调用时：

1. 读取现有的 `UBIQUITOUS_LANGUAGE.md`
2. 纳入后续讨论中的所有新术语
3. 如果理解有所演进，更新定义
4. 重新标记所有新出现的歧义
5. 重写示例对话，以纳入新术语
