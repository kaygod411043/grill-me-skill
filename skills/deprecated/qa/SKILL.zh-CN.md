---
name: qa
description: 进行交互式 QA 会话，由用户以对话方式报告 bug 或问题，代理负责创建 GitHub issue。后台探索代码库以获取上下文和领域语言。当用户想要报告 bug、进行 QA、以对话方式创建 issue，或提到“QA 会话”时使用。
---

# QA 会话

运行交互式 QA 会话。用户描述他们遇到的问题。你负责澄清问题、探索代码库以获取上下文，并创建持久、以用户为中心且使用项目领域语言的 GitHub issue。

## 对用户提出的每个问题

### 1. 倾听并适度澄清

让用户用自己的话描述问题。提出**最多 2 到 3 个简短的澄清问题**，重点关注：

- 他们预期发生什么，而实际发生了什么
- 复现步骤（如果并不明显）
- 问题是稳定出现还是偶发

不要过度追问。如果描述已经足以创建 issue，就继续下一步。

### 2. 在后台探索代码库

与用户交谈时，在后台启动一个 Agent（`subagent_type=Explore`），以了解相关区域。目标**不是**寻找修复方法，而是：

- 学习该区域使用的领域语言（检查 `UBIQUITOUS_LANGUAGE.md`）
- 理解该功能本应做什么
- 确定面向用户的行为边界

这些上下文有助于你写出更好的 issue，但 issue 本身**不应**引用具体文件、行号或内部实现细节。

### 3. 评估范围：单个 issue 还是拆分？

创建 issue 前，先判断这是一个**单独的 issue**，还是需要**拆分**成多个 issue。

在以下情况中拆分：

- 修复跨越多个彼此独立的区域（例如，“表单验证错误、成功消息缺失，而且重定向也坏了”）
- 存在可以由不同人员并行处理、明确可分离的关注点
- 用户描述的情况包含多个不同的失败模式或症状

在以下情况中保留为单个 issue：

- 只有一个位置上的一种行为出错
- 所有症状都源于同一个根本行为

### 4. 创建 GitHub issue

使用 `gh issue create` 创建 issue。不要先要求用户审核，直接创建并分享 URL。

Issue 必须**持久有效**，即使经过大规模重构也仍然有意义。要从用户视角来写。

#### 单个 issue

使用以下模板：

```
## What happened

[Describe the actual behavior the user experienced, in plain language]

## What I expected

[Describe the expected behavior]

## Steps to reproduce

1. [Concrete, numbered steps a developer can follow]
2. [Use domain terms from the codebase, not internal module names]
3. [Include relevant inputs, flags, or configuration]

## Additional context

[Any extra observations from the user or from codebase exploration that help frame the issue — e.g. "this only happens when using the Docker layer, not the filesystem layer" — use domain language but don't cite files]
```

#### 拆分（多个 issue）

按依赖顺序创建 issue（阻塞项优先），以便引用真实的 issue 编号。

每个子 issue 使用以下模板：

```
## Parent issue

#<parent-issue-number> (if you created a tracking issue) or "Reported during QA session"

## What's wrong

[Describe this specific behavior problem — just this slice, not the whole report]

## What I expected

[Expected behavior for this specific slice]

## Steps to reproduce

1. [Steps specific to THIS issue]

## Blocked by

- #<issue-number> (if this issue can't be fixed until another is resolved)

Or "None — can start immediately" if no blockers.

## Additional context

[Any extra observations relevant to this slice]
```

进行拆分时：

- **宁可创建多个轻量 issue，也不要创建少数厚重 issue**，每个 issue 都应能够独立修复和验证
- **如实标明阻塞关系**，如果 issue B 确实要等 issue A 修复后才能测试，就明确说明；如果它们彼此独立，就都标为 `None — can start immediately`
- **按依赖顺序创建 issue**，这样就能在 `Blocked by` 中引用真实的 issue 编号
- **最大化并行度**，目标是让多人（或多个代理）能够同时接手不同的 issue

#### 所有 issue 正文的规则

- **不要包含文件路径或行号**，这些信息会过时
- **使用项目的领域语言**（如果存在 `UBIQUITOUS_LANGUAGE.md`，请检查它）
- **描述行为，而不是代码**，例如写“同步服务未能应用补丁”，而不是“`applyPatch()` 在第 42 行抛出异常”
- **必须提供复现步骤**，如果无法确定，就询问用户
- **保持简洁**，开发者应能在 30 秒内读完 issue

创建后，输出所有 issue 的 URL（并总结阻塞关系），然后询问：“下一个问题，还是就到这里？”

### 5. 继续会话

持续进行，直到用户表示结束。每个 issue 都相互独立，不要批量处理。
