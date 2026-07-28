# HTML 报告格式

架构审查应呈现为操作系统临时目录中的单个自包含 HTML 文件。Tailwind 和 Mermaid 都来自 CDN。Mermaid 能可靠处理图结构的图表；手工 div 和内联 SVG 则用于更具编辑感的视觉效果（体量图、剖面图）。将两者混合使用，不要所有内容都依赖 Mermaid，否则看起来会很通用。

## 脚手架

```html
<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Architecture review — {{repo name}}</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      /* small custom layer for things Tailwind doesn't cover cleanly:
         dashed seam lines, hand-drawn-feeling arrow heads, etc. */
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
      .deep { background: linear-gradient(135deg, #0f172a, #1e293b); }
    </style>
  </head>
  <body class="bg-stone-50 text-slate-900 font-sans">
    <main class="max-w-5xl mx-auto px-6 py-12 space-y-12">
      <header>...</header>
      <section id="candidates" class="space-y-10">...</section>
      <section id="top-recommendation">...</section>
    </main>
  </body>
</html>
```

## 页头

显示仓库名称、日期和紧凑图例：实线框 = module，虚线 = seam，红色箭头 = leakage，深色粗框 = deep module。不写介绍段落，直接进入候选项。

## 候选项卡片

让图表承担主要表达任务。文字应稀少、直白，并自然使用 `/codebase-design` skill 中的术语表词汇。

每个候选项使用一个 `<article>`：

- **标题**：简短，点明深化动作，例如“Collapse the Order intake pipeline”。
- **徽章行**：推荐强度（`Strong` = emerald、`Worth exploring` = amber、`Speculative` = slate），以及依赖类别标签（`in-process`、`local-substitutable`、`ports & adapters`、`mock`）。
- **文件**：等宽字体列表，使用 `font-mono text-sm`。
- **之前/之后图**：核心内容。两列并排。参见下面的模式。
- **问题**：一句话说明痛点。
- **解决方案**：一句话说明变化。
- **收益**：项目符号，每项不超过 6 个词，例如“Tests hit one interface”“Pricing logic stops leaking”“Delete 4 shallow wrappers”。
- **ADR 提示**（如适用）：琥珀色背景框中的一行文字。

不要写解释段落。如果图表必须配上一段文字才能看懂，就重新绘制图表。

## 图表模式

选择适合候选项的模式，并混合使用。不要让所有图表看起来相同，多样性本身就是重点的一部分。

### Mermaid 图（依赖/调用流的主力）

当重点是“X 调用 Y、Y 调用 Z，看看有多混乱”时，使用 Mermaid `flowchart` 或 `graph`。用 Tailwind 样式卡片包裹，避免它看起来像突然插入的外来内容。使用 classDef 将泄漏边染成红色，把深层模块设为深色。对于“之前：6 次往返；之后：1 次”之类的对比，时序图效果很好。

```html
<div class="rounded-lg border border-slate-200 bg-white p-4">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### 手工框线图（当 Mermaid 布局与你对抗时）

用带边框和标签的 `<div>` 表示模块。在相对定位容器上方绝对定位内联 SVG `<line>` 或 `<path>` 元素作为箭头。当你希望“之后”图呈现为一个粗边框深层模块，内部元素则变灰时，使用这种方式；Mermaid 无法以合适的视觉重量实现它。

### 剖面图（适合展示分层的浅薄程度）

堆叠水平条带（`h-12 border-l-4`），展示一次调用经过的各层。之前：6 个几乎什么都不做的薄层。之后：1 个标有合并职责的厚条带。

### 体量图（适合展示“接口和实现一样宽”）

每个模块使用两个矩形，一个表示接口表面积，一个表示实现。之前：接口矩形几乎与实现矩形等高（浅层）。之后：接口矩形很矮，实现矩形很高（深层）。

### 调用图折叠

之前：用嵌套框渲染函数调用树。之后：把同一棵树折叠进一个框中，并在内部以淡化效果展示现已成为内部调用的部分。

## 样式指南

- 偏编辑风格，而非企业仪表板。使用充足留白。标题可选择衬线字体（`font-serif` 效果很好，适合 stone/slate）。
- 谨慎使用颜色：一种强调色（emerald 或 indigo），再加上用于泄漏的红色和用于警告的 amber。
- 图表高度保持在约 320px，使之前/之后图可以舒适地并排展示，无需滚动。
- 图表中的模块标签使用 `text-xs uppercase tracking-wider`，应读起来像示意图，而不是 UI。
- 仅使用 Tailwind CDN 和 Mermaid ESM import 两个脚本。报告其余部分都是静态内容，除了 Mermaid 自身渲染，不包含应用代码或交互。

## 首要建议章节

使用一张较大的卡片。包含候选项名称、一句原因和指向其卡片的锚点链接。仅此而已。

## 语气

使用简洁、直白的语言，但架构名词和动词必须直接来自 `/codebase-design` skill。简洁不是偏离术语的借口。

**严格使用：** module、interface、implementation、depth、deep、shallow、seam、adapter、leverage、locality。

**绝不替换为：** component、service、unit（代替 module）· API、signature（代替 interface）· boundary（代替 seam）· layer、wrapper（本意为 module 时）。

**符合该风格的表述：**

- “Order intake module 是 shallow 的，interface 几乎与 implementation 相当。”
- “Pricing 泄漏越过了 seam。”
- “深化：一个 interface，一个测试位置。”
- “两个 adapter 使 seam 合理：生产环境用 HTTP，测试用 in-memory。”

**收益项目符号**应使用术语表命名收益：*“locality：缺陷集中在一个 module”*、*“leverage：一个 interface，N 个调用点”*、*“interface 收缩，implementation 吸收 wrappers”*。不要写*“更易维护”*或*“代码更整洁”*，这些词不在术语表中，不值得占据位置。

不要含糊其辞，不要铺垫，不要写“值得注意的是……”。一句话能写成项目符号，就写成项目符号；一个项目符号能删除，就删除。如果某个术语不在 `/codebase-design` 术语表中，在创造新词之前先从术语表中寻找合适的词。
