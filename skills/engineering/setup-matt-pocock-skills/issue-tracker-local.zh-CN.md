# Issue tracker: Local Markdown

此仓库的 issue 和 spec（你也可能把 spec 称为 PRD）以 `.scratch/` 中的 Markdown 文件形式存放。

## 约定

- 每个 feature 使用一个目录：`.scratch/<feature-slug>/`
- spec 位于 `.scratch/<feature-slug>/spec.md`
- 每个实现 issue 使用一个文件：`.scratch/<feature-slug>/issues/<NN>-<slug>.md`，从 `01` 开始编号，绝不使用单个合并 tickets 文件
- triage 状态记录为每个 issue 文件顶部附近的一行 `Status:`（role 字符串见 `triage-labels.zh-CN.md`）
- comment 和对话历史追加到文件底部的 `## Comments` 标题下

## 当 skill 要求“publish to the issue tracker”时

在 `.scratch/<feature-slug>/` 下创建新文件（需要时创建目录）。

## 当 skill 要求“fetch the relevant ticket”时

读取所引用路径下的文件。用户通常会直接传入路径或 issue 编号。

## Wayfinding 操作

供 `/wayfinder` 使用。**map** 是一个文件，每个 **child** ticket 使用一个文件。

- **Map**：`.scratch/<effort>/map.md`，body 包含 Notes / Decisions-so-far / Fog。
- **Child ticket**：`.scratch/<effort>/issues/NN-<slug>.md`，从 `01` 开始编号，问题写在 body 中。`Type:` 行记录 ticket 类型（`research`/`prototype`/`grilling`/`task`），`Status:` 行记录 `claimed`/`resolved`。
- **Blocking**：顶部附近的 `Blocked by: NN, NN` 行。当列出的每个文件均为 `resolved` 时，ticket 才解除阻塞。
- **Frontier**：扫描 `.scratch/<effort>/issues/`，寻找 open、unblocked 且 unclaimed 的文件；按编号取第一个。
- **Claim**：在开始任何工作前，设置 `Status: claimed` 并保存。
- **Resolve**：在 `## Answer` 标题下追加答案，设置 `Status: resolved`，然后向 `map.md` 中 map 的 Decisions-so-far 追加上下文链接（gist + link）。
