---
name: setup-ts-deep-modules
description: 将 dependency-cruiser 接入 TypeScript 仓库，使每个包都成为深模块：实现隐藏在子目录中，只能通过入口点文件访问。仅由用户调用。
disable-model-invocation: true
---

# 配置 TypeScript 深模块

让本仓库中的每个包都成为**深模块**：用小型接口承载大量行为。包的公开表面是它的**入口点**，即包根目录下的文件；所有子目录内容都保持隐藏。本技能安装 [dependency-cruiser](https://github.com/sverweij/dependency-cruiser)，配置只能通过入口点进入的规则，然后证明这些规则确实生效。

有关深模块、接口、接缝和深度等词汇，运行 `/codebase-design` 技能，并在整个过程中沿用其语言。

## 强制形成的结构

```
src/packages/
  <name>/
    index.ts        ← an entry point (public). Import this from outside.
    client.ts       ← another entry point. Packages may expose SEVERAL.
    lib/            ← implementation: hidden from outside, free to import each other.
    tests/          ← co-located tests + fixtures (a subfolder, so private).
```

公开表面是包的**根目录文件**，而不是某一个指定的 `index.ts`。按约定，实现放在 `lib/`，测试放在 `tests/`，让每个包都采用相同的双目录结构。但规则本身更通用：*任何*子目录中的*任何*内容都是私有的，因此无需为了新增目录扩展配置。

以下四条规则的级别全部为 `error`：

1. **入口点边界**：包外代码（应用代码或其他包）只能导入该包的入口点（根目录文件），绝不能导入其子目录中的内容。
2. **包内自由**：同一包内的文件可以自由相互导入。
3. **测试通过入口点访问**：`<pkg>/tests/` 下的文件可以导入任意包的入口点及自身的 `tests/` fixture，但绝不能导入任何包的子目录内部内容，连自身包也不例外。允许跨包集成测试，不允许深层导入。
4. **禁止循环依赖**：不允许依赖循环。

**入口点，而非 barrel。** 由于公开表面包含*所有*根目录文件，一个包可以暴露多个小型入口点（`index.ts`、`client.ts`、`server.ts`），无需把所有内容汇聚到一个巨型 `index.ts`。不鼓励重新导出整棵子树的 barrel 文件；保持入口点精简，并把实现隐藏在子目录中。

分层关系（哪些包可以依赖哪些包）是*另一个*问题，只在配置中保留注释 stub，供当前仓库自行填写。

## 步骤

### 1. 检测环境

- **包管理器**：`pnpm-lock.yaml` → pnpm，`yarn.lock` → yarn，`bun.lockb` → bun，否则使用 npm。后续所有命令都使用检测到的工具（`pnpm`/`yarn`/`npm run`/`bunx`）。
- **包根目录**：如果存在 `src/`，使用 `src/packages`；否则使用 `packages`。如果仓库已经采用另一种明显约定，向用户确认选择。
- **现有配置**：检查 `.dependency-cruiser.*` 文件。如果存在，绝不能覆盖；把四条规则和选项合并进去，并告诉用户增加了什么。

**完成判据：** 已知包管理器、包根目录和现有配置状态。

### 2. 安装 dependency-cruiser

使用检测到的包管理器，将 `dependency-cruiser` 安装为 devDependency。

**完成判据：** `dependency-cruiser` 已列入 `devDependencies`。

### 3. 编写配置

将 [`dependency-cruiser.config.cjs`](./dependency-cruiser.config.cjs) 复制到仓库根目录，命名为 `.dependency-cruiser.cjs`。把 `PACKAGES_ROOT` 设置为步骤 1 检测到的根目录。规则基于路径深度且与扩展名无关，无需调整其他内容。

**完成判据：** `.dependency-cruiser.cjs` 存在，`PACKAGES_ROOT` 正确，并且包含四条禁止规则。

### 4. 接入检查流程

- 添加 `lint:boundaries` 脚本：`depcruise <packages-root>`（或 `depcruise src`）。
- 将它并入仓库的总检查命令，即已经运行 typecheck 的命令（例如 `check` / `ci` / `validate` 脚本）。不要修改 `tsconfig`，也不要添加路径别名。
- 如果没有总检查脚本，添加 `lint:boundaries`，并告诉用户应将其纳入 CI。

**完成判据：** `lint:boundaries` 已存在，并与 typecheck 通过同一个命令运行。

### 5. 搭建示例包

创建并提交 `<packages-root>/example/`，作为可复制的模板：

- `index.ts`：一个入口点。导出一个委派给内部文件的函数，使这个包显然是*深*的，而非简单透传。
- `lib/impl.ts`：位于**子目录**中的内部文件，由 `index.ts` 导入，包外无法访问。
- `tests/example.test.ts`：**只**导入 `../index`（入口点），并针对公开函数断言。

告诉用户，这是一个可以复制或删除的起始模板。

**完成判据：** 示例包已经存在，通过根入口点暴露行为，并将 `impl` 隐藏在子目录中。

### 6. 证明规则确实生效

这是整个技能的完成判据；无法在违规时失败的配置毫无价值。

1. 运行 `lint:boundaries`。它必须在干净的示例上**通过**。
2. 临时向 `tests/example.test.ts` 添加深层导入（例如 `import { thing } from "../lib/impl"`）。再次运行 `lint:boundaries`，它必须以 `tests-through-entrypoints` **失败**。
3. 撤销该深层导入。再次运行，必须**通过**。

**完成判据：** 已亲眼观察到先通过、再因深层导入而失败、最后再次通过。如果步骤 2 没有失败，说明规则未正确接入，修复后再结束。

### 7. 记录约定

在包目录中编写一个 `README.md`（`<packages-root>/README.md`），放在它所约束的包旁边。内容包括：`src/packages/<name>/` 布局（入口点在根目录，`lib/` 存放实现，`tests/` 存放测试）、“只通过包的入口点（根目录文件）导入”，以及如何运行 `lint:boundaries`。明确**不鼓励 barrel 文件**：暴露多个小型入口点，不要通过一个 index 重新导出整棵子树。内容保持为可复制片段，再用四个段落分别说明四条规则。

然后在仓库的代理说明文件中加入指向它的**上下文指针**：优先使用已有的 `CLAUDE.md`，否则使用 `AGENTS.md`；如果两者都不存在，创建 `AGENTS.md`。一行即可，例如：`Packages are deep modules — see [src/packages/README.md](./src/packages/README.md) before adding or importing one.` 这样代理会主动发现边界规则，而不是在违反后才碰到它。

**完成判据：** `<packages-root>/README.md` 已存在并明确反对 barrel，且仓库的 `CLAUDE.md`/`AGENTS.md` 已链接到它。

## 备注

- 配置中的 `$1` 反向引用（dependency-cruiser 的分组匹配）让包内可以访问自身内部内容，同时阻止外部访问；不要把它们展开成每个包各自一套规则。
- 公开与私有由**深度**决定：包根目录文件是入口点，子目录中的任何内容都是私有的。约定子目录为 `lib/`（实现）和 `tests/`，但规则没有硬编码它们；任何子目录都属于私有，因此新增目录不需要修改配置。新增入口点只需新增根目录文件，无需 barrel。
- 包是**扁平的**：根目录下只有一层直接子包。包内部可以任意深度嵌套，但包内不能再包含另一个包。
- 使用 `.cjs` 而非 `.js`，确保即使仓库设有 `"type": "module"`，配置中的 `module.exports` 仍能工作。
