# 范围外知识库

仓库中的 `.out-of-scope/` 目录保存被拒绝功能请求的持久记录。它有两个用途：

1. **组织记忆**：记录功能被拒绝的原因，避免 issue 关闭后丢失决策理由
2. **去重**：当新 issue 与之前的拒绝项匹配时，技能可以指出先前决策，而不是重新争论

## 目录结构

```
.out-of-scope/
├── dark-mode.md
├── plugin-system.md
└── graphql-api.md
```

每个**概念**一个文件，而不是每个 issue 一个文件。请求相同内容的多个 issue 归入同一个文件。

## 文件格式

文件应采用轻松易读的风格，更像一份简短设计文档，而不是数据库记录。使用段落、代码示例和实例，让第一次接触这项决策的人也能清晰理解理由。

```markdown
# Dark Mode

本项目不支持 dark mode 或面向用户的主题设置。

## Why this is out of scope

渲染管线假设只有一套由 `ThemeConfig` 定义的配色。
支持多主题需要：

- 包裹整个组件树的 theme context provider
- 每个组件都能感知主题的样式解析
- 保存用户主题偏好的持久化层

这是一项重大的架构变更，与项目专注内容创作的目标不一致。
主题设置应由嵌入或重新分发输出的下游消费者负责。

```ts
// The current ThemeConfig interface is not designed for runtime switching:
interface ThemeConfig {
  colors: ColorPalette; // single palette, resolved at build time
  fonts: FontStack;
}
```

## Prior requests

- #42 — "Add dark mode support"
- #87 — "Night theme for accessibility"
- #134 — "Dark theme option"
```

### 命名文件

为概念使用简短、描述性的 kebab-case 名称，例如 `dark-mode.md`、`plugin-system.md`、`graphql-api.md`。名称应足够容易辨认，使浏览目录的人无需打开文件就能理解被拒绝的内容。

### 编写理由

理由必须充实，不能只写“我们不想要”，而要解释原因。好的理由会引用：

- 项目范围或理念（“本项目专注 X；主题设置是下游关注点”）
- 技术约束（“支持这一点需要 Y，而 Y 与我们的 Z 架构冲突”）
- 战略决策（“我们选择 A 而不是 B，因为……”）

理由应长期有效。避免引用临时情况（“我们现在太忙”），那不是真正的拒绝，而是延期。

## 何时检查 `.out-of-scope/`

在分诊期间（步骤 1：收集上下文）读取 `.out-of-scope/` 中的所有文件。评估新 issue 时：

- 检查请求是否匹配现有的范围外概念
- 按概念相似度而不是关键词匹配，例如 “night theme” 匹配 `dark-mode.md`
- 如果匹配，向维护者指出：“这与 `.out-of-scope/dark-mode.md` 相似；我们之前因为 [reason] 拒绝过它。你现在仍然持相同看法吗？”

维护者可以：

- **确认**：将新 issue 添加到现有文件的 “Prior requests” 列表，然后关闭
- **重新考虑**：删除或更新范围外文件，让 issue 进入正常分诊流程
- **不同意**：这些 issue 有关但不同，继续正常分诊

## 何时写入 `.out-of-scope/`

只有当一个 **enhancement**（不是 bug）以 `wontfix` 被_拒绝_时才写入。此规则同样适用于 enhancement PR 和 issue：记录被拒绝的 PR，避免同一请求以后以全新代码的形式再次出现。

不要在某项内容因**已经实现**而以 `wontfix` 关闭时写入。那是已构建的功能，不是被拒绝的功能；记录它会用错误的拒绝项污染去重检查。关闭评论应改为指出功能现有的位置。

流程：

1. 维护者决定一项功能请求不在范围内
2. 检查是否已经存在匹配的 `.out-of-scope/` 文件
3. 如果存在：将新 issue 追加到 “Prior requests” 列表
4. 如果不存在：以概念名创建新文件，写入决策、理由和第一条先前请求
5. 在 issue 上发布评论，解释决策并提及 `.out-of-scope/` 文件
6. 使用 `wontfix` label 关闭 issue

## 更新或删除范围外文件

如果维护者改变了对先前被拒绝概念的看法：

- 删除 `.out-of-scope/` 文件
- 本技能无需重新打开旧 issue，它们是历史记录
- 触发重新考虑的新 issue 进入正常分诊流程
