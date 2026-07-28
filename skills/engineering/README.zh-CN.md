# 工程

我日常用于代码工作的技能。

## 用户调用

只有在你明确输入技能名时才可调用（Claude Code：`disable-model-invocation: true`；Codex：`agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

- **[ask-matt](./ask-matt/SKILL.zh-CN.md)** — 询问哪项技能或哪条流程适合你当前的情况。本仓库用户调用技能的路由器。
- **[grill-with-docs](./grill-with-docs/SKILL.zh-CN.md)** — 在追问会话中构建项目的领域模型，打磨术语，并同步更新 `CONTEXT.md` 和 ADR。
- **[triage](./triage/SKILL.zh-CN.md)** — 让 issue 在由多个 triage role 组成的状态机中流转。
- **[improve-codebase-architecture](./improve-codebase-architecture/SKILL.zh-CN.md)** — 扫描代码库以寻找深化机会，将它们呈现为可视化 HTML 报告，然后针对你选择的机会展开追问。
- **[setup-matt-pocock-skills](./setup-matt-pocock-skills/SKILL.zh-CN.md)** — 为 engineering 技能配置本仓库（issue tracker、triage label、领域文档布局）。每个仓库运行一次。
- **[to-spec](./to-spec/SKILL.zh-CN.md)** — 将当前对话转换为 spec 并发布到 issue tracker。
- **[to-tickets](./to-tickets/SKILL.zh-CN.md)** — 将任意计划、spec 或对话拆分为一组 tracer-bullet ticket，每个 ticket 都声明自己的阻塞边——可以写入本地文件，也可以在真实 tracker 上使用原生阻塞链接。
- **[implement](./implement/SKILL.zh-CN.md)** — 构建 spec 或一组 ticket 所描述的工作，在预先约定的 seam 上驱动 `/tdd`，并在提交前通过 `/code-review` 收尾。
- **[wayfinder](./wayfinder/SKILL.zh-CN.md)** — 将一项超出单个代理会话容量的庞大工作，规划为 issue tracker 上由决策 ticket 组成的共享地图，逐个解决，直到通往目标的道路清晰。

## 模型调用

模型或用户均可调用（使用丰富的触发描述，让模型能主动选择它们）。

- **[prototype](./prototype/SKILL.zh-CN.md)** — 构建一个一次性原型来回答设计问题：可以是验证状态或逻辑的可运行终端应用，也可以是多种可切换的 UI 变体。

- **[diagnosing-bugs](./diagnosing-bugs/SKILL.zh-CN.md)** — 针对棘手 bug 和性能 regression 的严格诊断循环：复现 → 最小化 → 提出假设 → 插桩 → 修复 → regression test。
- **[research](./research/SKILL.zh-CN.md)** — 根据高可信度的一手来源调查问题，并将发现记录为仓库中带引用的 Markdown 文件，由后台代理执行。
- **[tdd](./tdd/SKILL.zh-CN.md)** — 使用红-绿-重构循环进行测试驱动开发。每次完成一个垂直切片，用于构建功能或修复 bug。
- **[domain-modeling](./domain-modeling/SKILL.zh-CN.md)** — 主动构建并打磨项目的领域模型——挑战术语、用场景进行压力测试，并同步更新 `CONTEXT.md` 和 ADR。
- **[codebase-design](./codebase-design/SKILL.zh-CN.md)** — 用于设计深模块的共享原则与词汇：小接口、清晰 seam，并可通过接口测试。
- **[code-review](./code-review/SKILL.zh-CN.md)** — 从固定点开始对 diff 进行双轴审查：**标准（Standards）**检查是否遵循仓库编码标准及 Fowler 坏味道基线，**规格（Spec）**检查是否忠实实现原始 issue/PRD；两项审查由并行子代理运行。
- **[resolving-merge-conflicts](./resolving-merge-conflicts/SKILL.zh-CN.md)** — 逐个 hunk 处理进行中的 git merge 或 rebase 冲突，通过追溯双方的一手来源，按照意图解决冲突，然后完成操作——绝不使用 `--abort`。
