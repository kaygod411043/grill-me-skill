# 编写代理简报

代理简报是在 GitHub issue 或 PR 转为 `ready-for-agent` 时发布的一条结构化评论。它是 AFK 代理执行工作的权威规格说明。原始正文和讨论只是上下文，代理简报才是契约。

简报说明**代理应做什么**，并覆盖两种入口：对于 issue，是从零构建改动；对于 PR，则是_在现有 diff 上_还需要完成什么，例如收尾、补齐缺口、处理审查意见。两种情况遵循相同原则；下方 PR 示例展示了差异。

## 原则

### 持久性优先于精确性

Issue 可能在 `ready-for-agent` 中停留数天或数周，期间代码库会发生变化。编写简报时，即使文件被重命名、移动或重构，它仍应有用。

- **要**描述接口、类型和行为契约
- **要**指出代理应查找或修改的具体类型、函数签名或配置结构
- **不要**引用文件路径，它们会过时
- **不要**引用行号
- **不要**假设当前实现结构会保持不变

### 描述行为，而非过程

描述系统**应该做什么**，而不是**如何**实现。代理会重新探索代码库，并自行作出实现决策。

- **好：**“`SkillConfig` 类型应接受一个 `CronExpression` 类型的可选 `schedule` 字段”
- **坏：**“打开 src/types/skill.ts，在第 42 行添加 schedule 字段”
- **好：**“用户不带参数运行 `/triage` 时，应看到需要关注的 issue 摘要”
- **坏：**“在主 handler 函数中添加 switch 语句”

### 完整的验收标准

代理需要知道何时完成。每份代理简报都必须包含具体且可测试的验收标准，每条标准都应能独立验证。

- **好：**“运行 `gh issue list --label needs-triage` 会返回已经过初步分类的 issue”
- **坏：**“分诊应该正常工作”

### 明确范围边界

说明哪些内容不在范围内。这可以防止代理过度打磨，或对相邻功能作出假设。

## 模板

```markdown
## Agent Brief

**Category:** bug / enhancement
**Summary:** 用一句话说明需要完成什么

**Current behavior:**
描述当前发生的行为。对于 bug，这是损坏的行为。
对于 enhancement，这是该功能将要扩展的现状。

**Desired behavior:**
描述代理完成工作后应该发生的行为。
具体说明边缘情况和错误条件。

**Key interfaces:**
- `TypeName`：需要改变什么，以及原因
- `functionName()` 返回类型：当前返回什么，应该返回什么
- Config shape：所需的任何新配置选项

**Acceptance criteria:**
- [ ] 具体且可测试的标准 1
- [ ] 具体且可测试的标准 2
- [ ] 具体且可测试的标准 3

**Out of scope:**
- 此 issue 不应改变或处理的内容
- 看似相关但应单独处理的相邻功能
```

## 示例

### 好的代理简报（bug）

```markdown
## Agent Brief

**Category:** bug
**Summary:** Skill description 截断会切断单词，产生损坏的输出

**Current behavior:**
当 skill description 超过 1024 个字符时，无论单词边界如何，都会在第 1024
个字符处直接截断。这会让 description 结束于半个单词
（例如 “Use when the user wants to confi”）。

**Desired behavior:**
截断应发生在 1024 个字符前的最后一个单词边界，
并追加 “...” 表示内容被截断。

**Key interfaces:**
- `SkillMetadata` 类型的 `description` 字段：无需改变类型，
  但填充该字段的验证或处理逻辑需要遵守单词边界
- 任何读取 SKILL.md frontmatter 并提取 description 的函数

**Acceptance criteria:**
- [ ] 少于 1024 个字符的 description 保持不变
- [ ] 超过 1024 个字符的 description 在 1024 个字符前的最后一个单词边界截断
- [ ] 被截断的 description 以 “...” 结尾
- [ ] 包含 “...” 在内的总长度不超过 1024 个字符

**Out of scope:**
- 改变 1024 个字符的限制本身
- 支持多行 description
```

### 好的代理简报（enhancement）

```markdown
## Agent Brief

**Category:** enhancement
**Summary:** 增加 `.out-of-scope/` 目录支持，用于跟踪被拒绝的功能请求

**Current behavior:**
功能请求被拒绝时，issue 会带着 `wontfix` label 和一条评论被关闭。
没有持久记录保存这项决策或理由。
未来遇到类似请求时，维护者必须回忆或搜索之前的讨论。

**Desired behavior:**
被拒绝的功能请求应记录在 `.out-of-scope/<concept>.md` 文件中，
其中包含决策、理由，以及请求过该功能的所有 issue 链接。
分诊新 issue 时，应检查这些文件是否存在匹配项。

**Key interfaces:**
- `.out-of-scope/` 中的 Markdown 文件格式：每个文件都应有
  `# Concept Name` 标题、`**Decision:**` 行、`**Reason:**` 行，
  以及包含 issue 链接的 `**Prior requests:**` 列表
- 分诊工作流应尽早读取所有 `.out-of-scope/*.md` 文件，
  并按概念相似度将新 issue 与它们匹配

**Acceptance criteria:**
- [ ] 将 enhancement 以 wontfix 关闭时，会在 `.out-of-scope/` 中创建或更新文件
- [ ] 文件包含决策、理由和已关闭 issue 的链接
- [ ] 如果匹配的 `.out-of-scope/` 文件已经存在，
      将新 issue 追加到其 “Prior requests” 列表，而不是创建重复文件
- [ ] 分诊期间会检查现有 `.out-of-scope/` 文件，
      并在新 issue 匹配先前拒绝项时指出它

**Out of scope:**
- 自动匹配（由人类确认匹配）
- 重新打开之前被拒绝的功能
- Bug 报告（只有被拒绝的 enhancement 才写入 `.out-of-scope/`）
```

### 好的代理简报（PR）

对于 PR，“Current behavior” 描述 diff 的当前状态，简报要求代理完成或修复它，而不是从零构建。

```markdown
## Agent Brief

**Category:** enhancement
**Summary:** 完成贡献者为 `triage list` 添加的 `--json` 输出 flag

**Current behavior:**
该 PR 添加了 `--json` flag，将 issue 列表序列化为 JSON。正常路径有效，
diff 也符合项目的命令结构。还剩两个缺口：错误仍然以人类文本打印（不是 JSON），
并且新 flag 没有测试覆盖。

**Desired behavior:**
使用 `--json` 时，包括错误在内的所有输出都应是 stdout 上格式正确的 JSON，
且命令的 exit code 保持不变。缺少该 flag 时，现有的人类可读输出完全不变。

**Key interfaces:**
- 命令的错误路径在 `--json` 下应输出 `{ "error": string }`，
  而不是纯文本错误
- 复用 PR 已添加的 serializer；不要引入第二个

**Acceptance criteria:**
- [ ] `triage list --json` 在成功和错误情况下都输出有效 JSON
- [ ] Exit code 与非 JSON 命令一致
- [ ] 测试覆盖 `--json` 成功输出和一个错误情况
- [ ] 默认（非 JSON）输出逐字节保持不变

**Out of scope:**
- 为任何其他命令添加 `--json`
- 改变 PR 已定义的成功 payload 的 JSON 结构
```

### 坏的代理简报

```markdown
## Agent Brief

**Summary:** 修复分诊 bug

**What to do:**
分诊相关内容损坏了。查看主文件并修复。
第 150 行附近的函数存在问题。

**Files to change:**
- src/triage/handler.ts (line 150)
- src/types.ts (line 42)
```

它很差，因为：

- 没有类别
- 描述含糊（“分诊相关内容损坏了”）
- 引用了会过时的文件路径和行号
- 没有验收标准
- 没有范围边界
- 没有描述当前行为与期望行为
