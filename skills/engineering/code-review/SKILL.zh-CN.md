---
name: code-review
description: 从一个固定点（commit、branch、tag 或 merge-base）起，沿两个轴审查变更——标准（Standards，即代码是否遵循本仓库记录的编码标准）和规格（Spec，即代码是否符合原始 issue/PRD 的要求）。在并行子代理中运行两项审查，并并排报告结果。当用户想要审查 branch、PR、进行中的变更，或要求“审查从 X 开始的变更”时使用。
---

从用户提供的固定点到 `HEAD` 之间，对 diff 进行双轴审查：

- **标准（Standards）** — 代码是否符合本仓库记录的编码标准？
- **规格（Spec）** — 代码是否忠实实现了原始 issue / PRD / spec？

两个轴都在**并行子代理**中运行，避免彼此污染上下文，然后由本技能汇总发现。

Issue tracker 应已提供给你；如果缺少 `docs/agents/issue-tracker.md`，请运行 `/setup-matt-pocock-skills`。

## 流程

### 1. 固定比较点

用户所说的任何内容都是固定点——commit SHA、branch name、tag、`main`、`HEAD~5` 等。如果用户没有指定，就询问他们。

一次性确定 diff 命令：`git diff <fixed-point>...HEAD`（三点语法，因此会相对于 merge-base 进行比较）。同时通过 `git log <fixed-point>..HEAD --oneline` 记录 commit 列表。

继续之前，确认固定点能够解析（`git rev-parse <fixed-point>`），并且 diff 非空。错误的 ref 或空 diff 应在这里失败，而不是在两个并行子代理内部失败。

### 2. 确定 spec 来源

按以下顺序查找原始 spec：

1. Commit message 中的 issue 引用（`#123`、`Closes #45`、GitLab `!67` 等），按照 `docs/agents/issue-tracker.md` 中的工作流获取。
2. 用户作为参数传入的路径。
3. `docs/`、`specs/` 或 `.scratch/` 下与 branch name 或 feature 匹配的 PRD/spec 文件。
4. 如果什么也没找到，询问用户 spec 在哪里。如果他们表示没有 spec，**规格（Spec）**子代理将跳过，并报告 "no spec available"。

### 3. 确定标准来源

仓库中记录代码应如何编写的任何内容，例如 `CODING_STANDARDS.md` 或 `CONTRIBUTING.md`。

除仓库自身记录的内容外，标准（Standards）轴始终携带下面的**坏味道基线**——一组固定的 Fowler 代码坏味道（《Refactoring》第 3 章），即使仓库没有任何文档也同样适用。它受两条规则约束：

- **仓库规则优先。** 仓库中记录的标准始终具有优先权；如果它认可某种会被基线标记的做法，则不要报告该坏味道。
- **始终需要判断。** 每种坏味道都是带标签的启发式判断（例如“可能存在依恋情结（Feature Envy）”），绝不是硬性违规；而且与这里的任何标准一样，跳过工具已经强制检查的内容。

每种坏味道都按照*它是什么* → *如何修复*来描述；将其与 diff 进行匹配：

- **神秘命名（Mysterious Name）** — 函数、变量或类型的名称没有揭示它做什么或保存什么。→ 重命名；如果无法找到一个诚实的名字，说明设计仍然模糊。
- **重复代码（Duplicated Code）** — 相同的逻辑形态出现在本次变更的多个 hunk 或文件中。→ 提取共享形态，并从两处调用。
- **依恋情结（Feature Envy）** — 一个方法访问另一个对象的数据多于访问自身数据。→ 将该方法移动到它所依恋的数据上。
- **数据泥团（Data Clumps）** — 同样的几个字段或参数总是一起传递（一个尚未诞生的类型）。→ 将它们组合成一个类型，并传递该类型。
- **基本类型偏执（Primitive Obsession）** — 使用 primitive 或 string 代替一个值得拥有独立类型的领域概念。→ 为该概念创建自己的小型类型。
- **重复条件分支（Repeated Switches）** — 针对同一类型的相同 `switch`/`if` 级联在变更中反复出现。→ 使用多态替换，或让两处共享同一个 map。
- **霰弹式修改（Shotgun Surgery）** — 一项逻辑变更迫使 diff 中许多文件发生分散修改。→ 将共同变化的内容集中到一个模块中。
- **发散式变化（Divergent Change）** — 一个文件或模块因多个不相关的原因而被修改。→ 进行拆分，让每个模块只因一个原因而变化。
- **夸夸其谈通用性（Speculative Generality）** — 为 spec 中不存在的需求添加 abstraction、parameter 或 hook。→ 删除它们；在真实需求出现前内联回去。
- **消息链（Message Chains）** — 调用方依赖了本不该了解的长链式导航 `a.b().c().d()`。→ 将遍历过程隐藏在第一个对象的单一方法后面。
- **中间人（Middle Man）** — class 或 function 主要只是继续向下委派。→ 移除它，直接调用真正的目标。
- **被拒绝的遗赠（Refused Bequest）** — subclass 或 implementer 忽略或覆盖了其继承内容中的大部分。→ 放弃 inheritance，改用 composition。

### 4. 并行启动两个子代理

发送一条包含两个 `Agent` 工具调用的消息。两者都使用 `general-purpose` subagent。

**标准（Standards）子代理 prompt** — 包括：

- 完整的 diff 命令和 commit 列表。
- 在步骤 3 中找到的标准来源文件列表，**以及步骤 3 中的完整坏味道基线**——子代理没有其他途径访问它。
- 任务说明：“按相关文件或 hunk 报告：(a) diff 中违反已记录标准的每个位置，并引用该标准（文件 + 规则）；(b) 发现的任何基线坏味道，指出其名称并引用 hunk。区分硬性违规和判断项——违反已记录标准可以是硬性违规，但基线坏味道始终是判断项，而且仓库中记录的标准优先于基线。跳过工具已经强制检查的内容。不超过 400 words；中文输出按与 400 个英文单词等量的篇幅控制。”

**规格（Spec）子代理 prompt** — 包括：

- diff 命令和 commit 列表。
- spec 的路径或获取到的内容。
- 任务说明：“报告：(a) spec 要求但缺失或只部分实现的需求；(b) diff 中未经要求的行为（scope creep）；(c) 看似已实现、但实现方式似乎错误的需求。每个发现都引用对应的 spec 行。不超过 400 words；中文输出按与 400 个英文单词等量的篇幅控制。”

如果缺少 spec，跳过规格（Spec）子代理，并在最终报告中注明。

### 5. 汇总

将两份报告原样或略作清理后，分别放在 `## Standards` 和 `## Spec` 标题下。**不要**合并或重新排序各项发现——两个轴是刻意分开的（参见*为什么使用两个轴*）。

最后用一行进行总结：给出每个轴的发现总数，以及每个轴*内部*最严重的问题（如有）。不要在两个轴之间选出唯一最严重的问题，因为这种重新排序正是分离两个轴所要避免的。

## 为什么使用两个轴

一项变更可能通过其中一个轴，却未通过另一个轴：

- 代码遵循所有标准，却实现了错误的内容 → **标准（Standards）通过，规格（Spec）失败。**
- 代码完全实现了 issue 的要求，却破坏了项目约定 → **规格（Spec）通过，标准（Standards）失败。**

分开报告可以防止一个轴掩盖另一个轴。
