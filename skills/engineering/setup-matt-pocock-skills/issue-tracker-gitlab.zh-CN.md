# Issue tracker: GitLab

此仓库的 issue 和 PRD 以 GitLab issue 形式存放。所有操作都使用 [`glab`](https://gitlab.com/gitlab-org/cli) CLI。

## 约定

- **创建 issue**：`glab issue create --title "..." --description "..."`。多行 description 使用 heredoc。传入 `--description -` 以打开编辑器。
- **读取 issue**：`glab issue view <number> --comments`。使用 `-F json` 获取机器可读输出。
- **列出 issue**：`glab issue list -F json`，按需添加 `--label` 过滤器。
- **评论 issue**：`glab issue note <number> --message "..."`。GitLab 把 comment 称为“note”。
- **添加/删除 label**：`glab issue update <number> --label "..."` / `--unlabel "..."`。多个 label 可以用逗号分隔，也可以重复传入该 flag。
- **关闭**：`glab issue close <number>`。`glab issue close` 不接受关闭 comment，因此先用 `glab issue note <number> --message "..."` 发布说明，再关闭。
- **Merge request**：GitLab 把 PR 称为“merge request”。使用 `glab mr create`、`glab mr view`、`glab mr note` 等命令；整体结构与 `gh pr ...` 相同，只需用 `mr` 替换 `pr`，用 `note`/`--message` 替换 `comment`/`--body`。

从 `git remote -v` 推断仓库；在 clone 内运行时，`glab` 会自动完成此操作。

## 把 Merge request 作为 triage surface

**MRs as a request surface: no.** _（如果此仓库把外部 merge request 当作 feature request，则设为 `yes`；`/triage` 会读取此标志。）_

设为 `yes` 时，MR 使用与 issue 相同的 label 和状态，并使用对应的 `glab mr` 命令：

- **读取 MR**：`glab mr view <number> --comments`，并使用 `glab mr diff <number>` 获取 diff。
- **列出用于 triage 的外部 MR**：`glab mr list -F json`，然后只保留作者不是项目 member/owner 的 MR（贡献者的 MR，而非维护者正在进行的工作）。
- **评论/设置 label/关闭**：`glab mr note`、`glab mr update --label`/`--unlabel`、`glab mr close`。

与 GitHub 不同，GitLab 的 issue 和 MR 分别编号，因此一旦知道维护者所指的 surface，`#42` 就没有歧义。

## 当 skill 要求“publish to the issue tracker”时

创建 GitLab issue。

## 当 skill 要求“fetch the relevant ticket”时

运行 `glab issue view <number> --comments`。

## Wayfinding 操作

供 `/wayfinder` 使用。**map** 是单个 issue，**child** issue 作为 ticket。

- **Map**：一个带有 `wayfinder:map` label 的 issue，body 包含 Notes / Decisions-so-far / Fog。`glab issue create --label wayfinder:map`。（在支持原生 epic 的 GitLab tier 中，也可以用 epic 承载 map；带 label 的 issue 在所有 tier 都可用。）
- **Child ticket**：description 顶部带有 `Part of #<map>`，并带有 `wayfinder:<type>` label（`research`/`prototype`/`grilling`/`task`）的 issue。ticket 被认领后，将其分配给负责推进的开发者。
- **Blocking**：GitLab 的**原生 blocking link**，它是规范且在 UI 中可见的表示。通过 note 发布 `/blocked_by #<n>` quick action 来添加（`glab issue note <child> --message "/blocked_by #<blocker>"`）。原生 blocking link 是 Premium/Ultimate 功能；在 free tier 或功能不可用时，回退到 description 顶部的 `Blocked by: #<n>, #<n>` 行。当所有 blocker 均已关闭时，ticket 才解除阻塞。
- **Frontier query**：运行 `glab issue list -F json`，将范围限定为 map 的 child，删除存在未关闭 blocker（指向未关闭 issue 的原生 `blocked_by` link，可通过 `glab api projects/:id/issues/:iid/links` 查询；或 `Blocked by` 行中的 issue 尚未关闭）或已有 assignee 的项；按 map 顺序取第一个。
- **Claim**：`glab issue update <n> --assignee @me`，这是会话中的首次写操作。
- **Resolve**：先执行 `glab issue note <n> --message "<answer>"`，再执行 `glab issue close <n>`，然后向 map 的 Decisions-so-far 追加上下文链接（gist + link）。
