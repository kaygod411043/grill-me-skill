---
name: to-questionnaire
description: 把一个你无法独自回答的决策转化为问卷，交由其他人填写。
disable-model-invocation: true
---

把用户无法独自回答的问题转化为一份**问卷**：一份 Markdown 文档，可以异步交给某个人填写，也可以在会议中一起完成。接收者掌握用户缺少的知识，问卷负责将这些知识提取出来。

**追问发送方式，而非问题主题。** 只访谈用户必然能回答的*发送方式*：问卷交给谁，以及用户需要得到什么。文档中的问题再针对接收者所知与用户所需之间的**缺口**。

1. **要发给谁？** 在一次交流中询问接收者的角色、专业知识，以及与用户的关系。这决定问卷语气和需要携带多少背景。完成判据：清楚接收者是谁，以及他们掌握哪些用户不知道的知识。

2. **需要得到什么？** 在一次交流中询问用户无法独自解决、需要对方提供的具体决策或事实。完成判据：得到一份具体清单，列明用户最终必须能够采取哪些行动或做出哪些决策。

3. **编写问卷。** 按照下方“文档结构”，针对步骤 1–2 的缺口起草问题。将其写入当前目录下的 `to-questionnaire-<slug>.md`（slug 取自主题），并报告路径。完成判据：文件已存在，且用户在步骤 2 中列出的每一项都由一个问题覆盖。

## 文档结构

把文档定位为**探索问卷**：用户缺少背景，接收者掌握背景。按重要性从高到低排列问题，因为异步沟通可能只有一次机会；问题超过少量时，按主题归入 `##` 标题。使用以下模板编写。

<questionnaire-template>

# <Questionnaire title>

**Purpose:** why this questionnaire exists and the decision riding on it.

**From:** <the user> — **To:** <the recipient> — **How your answers will be used:** <where they go>

## Context

One paragraph orienting a recipient who wasn't in the user's head. Enough to answer well, not a page.

## How to answer

Deadline and rough effort. Partial answers and "I don't know" are useful — flag anything you're unsure of rather than skipping it.

## <Theme heading>

One `##` section per theme. Under each, its questions, most-important-first. Every question is one idea — never compound — with an answer stub directly beneath, and a one-line _why this matters_ only where the question could be misread or invite a throwaway answer.

<question-example>
### What load is the system expected to handle at launch?

_Why this matters: it decides whether we provision for burst traffic now or defer it._

>
</question-example>

## Anything else?

A closing catch-all: anything we didn't ask that we should know?

</questionnaire-template>
