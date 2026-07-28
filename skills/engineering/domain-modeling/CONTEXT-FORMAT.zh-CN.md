# CONTEXT.md 格式

## 结构

```md
# {Context Name}

{One or two sentence description of what this context is and why it exists.}

## Language

**Order**:
{A one or two sentence description of the term}
_Avoid_: Purchase, transaction

**Invoice**:
A request for payment sent to a customer after delivery.
_Avoid_: Bill, payment request

**Customer**:
A person or organization that places orders.
_Avoid_: Client, buyer, account
```

## 规则

- **要有明确立场。** 当同一概念存在多个词汇时，选出最合适的词，并把其他词列在 `_Avoid_` 下。
- **保持定义紧凑。** 最多一两句话。定义它“是什么”，而不是“做什么”。
- **仅包含该项目上下文特有的术语。** 即使项目大量使用超时、错误类型、工具模式等通用编程概念，也不应把它们放进来。添加术语前先问：这是该上下文特有的概念，还是通用编程概念？只有前者可以加入。
- **自然形成术语簇时，按子标题分组。** 如果所有术语都属于同一个内聚区域，使用扁平列表即可。

## 单上下文与多上下文仓库

**单上下文（大多数仓库）：** 仓库根目录有一个 `CONTEXT.md`。

**多个上下文：** 仓库根目录的 `CONTEXT-MAP.md` 列出各个上下文、它们所在的位置和相互关系：

```md
# Context Map

## Contexts

- [Ordering](./src/ordering/CONTEXT.md) — receives and tracks customer orders
- [Billing](./src/billing/CONTEXT.md) — generates invoices and processes payments
- [Fulfillment](./src/fulfillment/CONTEXT.md) — manages warehouse picking and shipping

## Relationships

- **Ordering → Fulfillment**: Ordering emits `OrderPlaced` events; Fulfillment consumes them to start picking
- **Fulfillment → Billing**: Fulfillment emits `ShipmentDispatched` events; Billing consumes them to generate invoices
- **Ordering ↔ Billing**: Shared types for `CustomerId` and `Money`
```

该 skill 会推断应采用哪种结构：

- 如果 `CONTEXT-MAP.md` 存在，读取它以定位各个上下文
- 如果只有根目录的 `CONTEXT.md`，则为单上下文
- 如果两者都不存在，则在确定第一个术语时惰性创建根目录的 `CONTEXT.md`

存在多个上下文时，推断当前主题属于哪个上下文。如果不明确，向用户询问。
