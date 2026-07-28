---
name: setup-matt-pocock-skills
description: 为此仓库配置工程 skill，包括 issue tracker、triage label 词汇和领域文档布局。在首次使用其他工程 skill 之前运行一次。
disable-model-invocation: true
---

# 设置 Matt Pocock 的 Skills

搭建工程 skill 所依赖的仓库级配置：

- **Issue tracker**：issue 所在的位置（默认 GitHub；开箱即用地支持本地 Markdown）
- **Triage labels**：五个规范 triage role 所使用的字符串
- **领域文档**：`CONTEXT.md` 和 ADR 所在的位置，以及读取它们的使用规则

这是一个由提示驱动的 skill，而不是确定性脚本。先探索、展示发现、与用户确认，然后再写入。

## 流程

### 1. 探索

查看当前仓库，了解起始状态。读取实际存在的内容，不要作假设：

- `git remote -v` 和 `.git/config`：这是 GitHub 仓库吗？具体是哪一个？
- 仓库根目录的 `AGENTS.md` 和 `CLAUDE.md`：是否存在？其中是否已经包含 `## Agent skills` 章节？
- 仓库根目录的 `CONTEXT.md` 和 `CONTEXT-MAP.md`
- `docs/adr/` 以及任何 `src/*/docs/adr/` 目录
- `docs/agents/`：本 skill 以前的输出是否已经存在？
- `.scratch/`：本地 Markdown issue tracker 约定是否已经在使用的信号
- 是否安装了 `triage` skill？（本 skill 同级目录中存在 `triage` skill 文件夹，或可用 skills 中包含 `triage`。）这决定是否执行 Section B。
- Monorepo 信号：`pnpm-workspace.yaml`、`package.json` 中的 `workspaces` 字段，或者包含自身 `src/` 且有内容的 `packages/*`。这些信号只会出现在真正庞大的多包仓库中；缺少这些信号意味着单上下文，几乎所有仓库都是如此。

### 2. 展示发现并询问

总结已有内容和缺失内容。然后按顺序逐节处理：每一节只提出一个问题，获得答案后再进入下一节。

每节都先给出推荐答案，让用户可以用一个词接受。只有选择确实会产生分支时，才提供一行说明；如果探索结果已经确定答案，则完全跳过该节（未安装 `triage` 时跳过 Section B，没有 monorepo 时跳过 Section C）。

**Section A — Issue tracker。**

> 说明：“issue tracker”是此仓库中存放 issue 的地方。`to-tickets`、`triage`、`to-spec` 和 `qa` 等 skill 会从中读取并写入；它们必须知道应该调用 `gh issue create`、在 `.scratch/` 下写 Markdown 文件，还是遵循你描述的其他工作流。选择你实际用于跟踪此仓库工作的地方。

默认立场：这些 skill 是为 GitHub 设计的。如果 `git remote` 指向 GitHub，则建议使用 GitHub。如果 `git remote` 指向 GitLab（`gitlab.com` 或自行托管的主机），则建议使用 GitLab。否则，或用户有不同偏好时，提供：

- **GitHub**：issue 存放在仓库的 GitHub Issues 中（使用 `gh` CLI）
- **GitLab**：issue 存放在仓库的 GitLab Issues 中（使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI）
- **本地 Markdown**：issue 以文件形式存放在仓库的 `.scratch/<feature>/` 下（适合个人项目或没有 remote 的仓库）
- **其他**（Jira、Linear 等）：让用户用一段话描述工作流；skill 会将其记录为自由格式文本

把选择记录到 `docs/agents/issue-tracker.md`。GitHub 和 GitLab 模板都包含“PRs as a request surface”标志，默认为 **off**。保持关闭，不要主动提及；如果用户希望把外部 PR 纳入 triage queue，之后可以自行在文件中切换该标志。

**Section B — Triage label vocabulary。** 如果没有安装 `triage` skill（探索阶段已确定），则完全跳过这一节。未安装的 skill 不需要 label。

如果已安装，只问一个问题：

> 你想保留默认 triage labels 吗？（建议：**yes**）

默认值是五个规范 role，每个 label 字符串都等于其名称：`needs-triage`、`needs-info`、`ready-for-agent`、`ready-for-human`、`wontfix`。用户回答 **yes** 时，原样写入。只有用户回答 no 时才收集覆盖值，通常是因为 tracker 已使用其他名称（例如用 `bug:triage` 表示 `needs-triage`）；这样 `triage` 会应用现有 label，而不是创建重复项。

**Section C — Domain docs。** 默认为**单上下文**：仓库根目录有一个 `CONTEXT.md` 和 `docs/adr/`。这适合几乎所有仓库，直接写入，无需询问。

只有在探索发现 monorepo 信号时，才提供**多上下文**选项：根目录的 `CONTEXT-MAP.md` 指向每个上下文各自的 `CONTEXT.md` 文件。然后确认用户想要哪种布局。

### 3. 确认并编辑

向用户展示以下内容的草稿：

- 将添加到所选 `CLAUDE.md` / `AGENTS.md` 中的 `## Agent skills` 块（选择规则见第 4 步）
- `docs/agents/issue-tracker.md`、`docs/agents/domain.md` 和 `docs/agents/triage-labels.md` 的内容（最后一个仅在安装了 `triage` 时显示）

允许用户在写入前编辑。

### 4. 写入

**选择要编辑的文件：**

- 如果 `CLAUDE.md` 存在，则编辑它。
- 否则，如果 `AGENTS.md` 存在，则编辑它。
- 如果两者都不存在，询问用户要创建哪一个，不要替用户选择。

当 `CLAUDE.md` 已存在时，绝不创建 `AGENTS.md`，反之亦然。始终编辑已有文件。

如果所选文件已经包含 `## Agent skills` 块，就在原位置更新其内容，而不是追加重复块。不要覆盖用户对周边章节的编辑。

该块为：

```markdown
## Agent skills

### Issue tracker

[one-line summary of where issues are tracked]. See `docs/agents/issue-tracker.md`.

### Triage labels

[one-line summary of the label vocabulary]. See `docs/agents/triage-labels.md`.

### Domain docs

[one-line summary of layout — "single-context" or "multi-context"]. See `docs/agents/domain.md`.
```

只有在安装了 `triage` 且执行过 Section B 时，才包含 `### Triage labels` 子块并写入 `docs/agents/triage-labels.md`。否则，两者都省略。

然后以本 skill 文件夹中的 seed 模板为起点写入文档文件：

- [issue-tracker-github.zh-CN.md](./issue-tracker-github.zh-CN.md)：GitHub issue tracker
- [issue-tracker-gitlab.zh-CN.md](./issue-tracker-gitlab.zh-CN.md)：GitLab issue tracker
- [issue-tracker-local.zh-CN.md](./issue-tracker-local.zh-CN.md)：本地 Markdown issue tracker
- [triage-labels.zh-CN.md](./triage-labels.zh-CN.md)：label 映射（仅在安装了 `triage` 时）
- [domain.zh-CN.md](./domain.zh-CN.md)：领域文档使用规则和布局

对于“其他”issue tracker，根据用户描述从头编写 `docs/agents/issue-tracker.md`。

### 5. 完成

告知用户设置已完成，并说明哪些工程 skill 现在会读取这些文件。提醒用户以后可以直接编辑 `docs/agents/*.md`；只有想切换 issue tracker 或从头开始时，才需要重新运行本 skill。
