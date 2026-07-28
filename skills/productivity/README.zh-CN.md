# 生产力

通用工作流工具，不局限于代码任务。

## 用户调用型

只有用户输入技能名时才能调用（Claude Code：`disable-model-invocation: true`；Codex：`agents/openai.yaml` 中的 `policy.allow_implicit_invocation: false`）。

- **[grill-me](./grill-me/SKILL.zh-CN.md)**：围绕计划或设计持续接受深入访谈，直到决策树的每个分支都得到解决。
- **[handoff](./handoff/SKILL.zh-CN.md)**：将当前对话压缩为交接文档，供另一个代理继续工作。
- **[teach](./teach/SKILL.zh-CN.md)**：通过多次会话向用户教授一项新技能或概念，并将当前目录作为有状态教学工作区。
- **[writing-great-skills](./writing-great-skills/SKILL.zh-CN.md)**：关于如何编写和编辑优秀技能的参考资料，涵盖使技能行为可预测的词汇与原则。

## 模型调用型

模型或用户均可调用（丰富的触发条件使模型能够主动选择它们）。

- **[grilling](./grilling/SKILL.zh-CN.md)**：围绕计划、决策或想法持续深入访谈用户，直到决策树的每个分支都得到解决。
