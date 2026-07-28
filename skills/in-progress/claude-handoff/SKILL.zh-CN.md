---
name: claude-handoff
description: 将当前对话交接给一个新的后台代理，由它立即接手工作。
argument-hint: "下一次会话将用于什么？"
disable-model-invocation: true
---

为当前对话编写交接摘要，让新的代理能够继续工作。不要保存摘要，而要用它作为提示词启动一个后台代理：`claude --bg --name "<descriptive name>" "<handoff summary>"`。后台代理从当前工作目录启动并立即返回；用户通过 `claude agents` 管理它。

始终通过 `-n`/`--name` 传入描述性名称（例如 `--name "Fix login bug"`），它会设置任务列表、会话选择器和终端标题中显示的名称。

在摘要中加入“建议技能”章节，推荐代理应调用的技能。

不要重复其他工件（PRD、计划、ADR、Issue、提交、diff）中已经记录的内容，改为引用其路径或 URL。

移除 API 密钥、密码、个人身份信息等任何敏感信息，因为摘要将成为代理的提示词。

如果用户传入了参数，将其视为下一次会话的工作重点描述，并据此调整摘要。
