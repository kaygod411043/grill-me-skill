# MISSION.md 格式

`MISSION.md` 位于工作区根目录。它记录用户学习该主题的*原因*。每一项教学决策，包括接下来教什么、提供哪些资源、设计哪些练习，都应追溯到此文档。

## 模板

```md
# Mission: {Topic}

## Why
{1-3 sentences. The concrete real-world goal the user is chasing. What changes in their life or work when they have this skill? Avoid abstract framings like "to understand X" — push for the underlying outcome.}

## Success looks like
- {A specific, observable thing the user will be able to do}
- {Another specific thing}
- {…}

## Constraints
- {Time, budget, prior commitments, learning preferences, anything that bounds the approach}

## Out of scope
- {Adjacent topics the user explicitly does not want to chase right now — protects the zone of proximal development}
```

## 规则

- **每个工作区只有一个使命。** 如果用户想学习两项不相关的内容，应建立两个工作区。
- **具体优先于抽象。** “十月前跑完半程马拉松”优于“变得更健康”；“向团队交付一个 Rust CLI”优于“学习 Rust”。
- **质疑含糊表达。** 如果用户无法说明原因，先访谈，再写入任何内容。糟糕的使命比没有使命更糟。
- **现实变化时修订。** 使命会变化。用户目标发生移动时，更新此文件，不要让过时使命继续引导后续会话。
- **保持简短。** 如果 `MISSION.md` 超过一屏，它就不再是指南针，而开始变成计划。
