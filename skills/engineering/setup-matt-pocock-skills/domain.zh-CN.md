# 领域文档

工程 skill 在探索代码库时应如何使用此仓库的领域文档。

## 探索前读取这些内容

- 仓库根目录的 **`CONTEXT.md`**，或者
- 如果根目录存在 **`CONTEXT-MAP.md`**，则读取它；该文件指向每个上下文的一个 `CONTEXT.md`。读取与当前主题相关的每一个文件。
- **`docs/adr/`**：读取与你即将处理的区域相关的 ADR。在多上下文仓库中，也要检查 `src/<context>/docs/adr/` 中上下文范围内的决策。

如果其中任何文件不存在，**静默继续**。不要指出文件缺失，也不要建议预先创建。`/domain-modeling` skill（通过 `/grill-with-docs` 和 `/improve-codebase-architecture` 使用）会在术语或决策真正确定时惰性创建它们。

## 文件结构

单上下文仓库（大多数仓库）：

```
/
├── CONTEXT.md
├── docs/adr/
│   ├── 0001-event-sourced-orders.md
│   └── 0002-postgres-for-write-model.md
└── src/
```

多上下文仓库（根目录存在 `CONTEXT-MAP.md`）：

```
/
├── CONTEXT-MAP.md
├── docs/adr/                          ← system-wide decisions
└── src/
    ├── ordering/
    │   ├── CONTEXT.md
    │   └── docs/adr/                  ← context-specific decisions
    └── billing/
        ├── CONTEXT.md
        └── docs/adr/
```

## 使用术语表词汇

当输出中提到领域概念时（例如 issue 标题、重构提案、假设、测试名称），使用 `CONTEXT.md` 中定义的术语。不要偏移到术语表明确要求避免的同义词。

如果需要的概念还不在术语表中，这就是一个信号：要么你正在创造项目中并未使用的语言（重新考虑），要么确实存在空缺（记录下来，交给 `/domain-modeling`）。

## 标出 ADR 冲突

如果输出与现有 ADR 冲突，应明确指出，而不是静默覆盖：

> _与 ADR-0007（event-sourced orders）冲突，但值得重新讨论，因为……_
