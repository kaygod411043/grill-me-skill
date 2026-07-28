# Issue tracker: GitHub

此仓库的 issue 和 PRD 以 GitHub issue 形式存放。所有操作都使用 `gh` CLI。

## 约定

- **创建 issue**：`gh issue create --title "..." --body "..."`。多行 body 使用 heredoc。
- **读取 issue**：`gh issue view <number> --comments`，使用 `jq` 过滤 comment，并同时获取 label。
- **列出 issue**：`gh issue list --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'`，按需添加 `--label` 和 `--state` 过滤器。
- **评论 issue**：`gh issue comment <number> --body "..."`
- **添加/删除 label**：`gh issue edit <number> --add-label "..."` / `--remove-label "..."`
- **关闭**：`gh issue close <number> --comment "..."`

从 `git remote -v` 推断仓库；在 clone 内运行时，`gh` 会自动完成此操作。

## 把 Pull request 作为 triage surface

**PRs as a request surface: no.** _（如果此仓库把外部 PR 当作 feature request，则设为 `yes`；`/triage` 会读取此标志。）_

设为 `yes` 时，PR 使用与 issue 相同的 label 和状态，并使用对应的 `gh pr` 命令：

- **读取 PR**：`gh pr view <number> --comments`，并使用 `gh pr diff <number>` 获取 diff。
- **列出用于 triage 的外部 PR**：`gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments`，然后仅保留 `authorAssociation` 为 `CONTRIBUTOR`、`FIRST_TIME_CONTRIBUTOR` 或 `NONE` 的项（删除 `OWNER`/`MEMBER`/`COLLABORATOR`）。
- **评论/设置 label/关闭**：`gh pr comment`、`gh pr edit --add-label`/`--remove-label`、`gh pr close`。

GitHub 的 issue 和 PR 共用同一个编号空间，因此单独的 `#42` 可能指任何一种。先用 `gh pr view 42` 解析，失败时回退到 `gh issue view 42`。

## 当 skill 要求“publish to the issue tracker”时

创建 GitHub issue。

## 当 skill 要求“fetch the relevant ticket”时

运行 `gh issue view <number> --comments`。

## Wayfinding 操作

供 `/wayfinder` 使用。**map** 是单个 issue，**child** issue 作为 ticket。

- **Map**：一个带有 `wayfinder:map` label 的 issue，body 包含 Notes / Decisions-so-far / Fog。`gh issue create --label wayfinder:map`。
- **Child ticket**：以 GitHub sub-issue 形式链接到 map 的 issue（通过 sub-issues endpoint 调用 `gh api`）。如果未启用 sub-issue，则把 child 加入 map body 中的 task list，并在 child body 顶部写入 `Part of #<map>`。Labels：`wayfinder:<type>`（`research`/`prototype`/`grilling`/`task`）。ticket 被认领后，将其分配给负责推进的开发者。
- **Blocking**：GitHub 的**原生 issue dependencies**，它是规范且在 UI 中可见的表示。使用 `gh api --method POST repos/<owner>/<repo>/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>` 添加边，其中 `<blocker-db-id>` 是 blocker 的数字 **database id**（`gh api repos/<owner>/<repo>/issues/<n> --jq .id`），*不是* `#number` 或 `node_id`。GitHub 通过 `issue_dependencies_summary.blocked_by` 报告依赖（仅包含未关闭 blocker，即实时 gate）。如果 dependencies 不可用，则回退到 child body 顶部的 `Blocked by: #<n>, #<n>` 行。当所有 blocker 均已关闭时，ticket 才解除阻塞。
- **Frontier query**：列出 map 中未关闭的 child（`gh issue list --state open`，范围限定为 map 的 sub-issue/task list），删除存在未关闭 blocker（`issue_dependencies_summary.blocked_by > 0`，或 `Blocked by` 行中的 issue 尚未关闭）或已有 assignee 的项；按 map 顺序取第一个。
- **Claim**：`gh issue edit <n> --add-assignee @me`，这是会话中的首次写操作。
- **Resolve**：先执行 `gh issue comment <n> --body "<answer>"`，再执行 `gh issue close <n>`，然后向 map 的 Decisions-so-far 追加上下文链接（gist + link）。
