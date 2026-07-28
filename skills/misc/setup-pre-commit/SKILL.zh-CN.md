---
name: setup-pre-commit
description: 在当前仓库中设置 Husky pre-commit hook，并配置 lint-staged（Prettier）、类型检查和测试。当用户想要添加 pre-commit hook、设置 Husky、配置 lint-staged，或添加提交时的格式化、类型检查和测试时使用。
---

# 设置 Pre-Commit Hook

## 本技能会设置什么

- **Husky** pre-commit hook
- **lint-staged**，对所有已暂存文件运行 Prettier
- **Prettier** 配置（如果缺失）
- pre-commit hook 中的 **typecheck** 和 **test** script

## 步骤

### 1. 检测包管理器

检查 `package-lock.json`（npm）、`pnpm-lock.yaml`（pnpm）、`yarn.lock`（yarn）、`bun.lockb`（bun）。使用存在的那一个。如果无法确定，默认使用 npm。

### 2. 安装依赖

安装为 devDependencies：

```
husky lint-staged prettier
```

### 3. 初始化 Husky

```bash
npx husky init
```

这会创建 `.husky/` 目录，并将 `prepare: "husky"` 添加到 package.json。

### 4. 创建 `.husky/pre-commit`

写入以下文件（Husky v9+ 不需要 shebang）：

```
npx lint-staged
npm run typecheck
npm run test
```

**适配**：将 `npm` 替换为检测到的包管理器。如果仓库的 package.json 中没有 `typecheck` 或 `test` script，则省略相应行并告知用户。

### 5. 创建 `.lintstagedrc`

```json
{
  "*": "prettier --ignore-unknown --write"
}
```

### 6. 创建 `.prettierrc`（如果缺失）

只有在不存在 Prettier 配置时才创建。使用以下默认值：

```json
{
  "useTabs": false,
  "tabWidth": 2,
  "printWidth": 80,
  "singleQuote": false,
  "trailingComma": "es5",
  "semi": true,
  "arrowParens": "always"
}
```

### 7. 验证

- [ ] `.husky/pre-commit` 存在且可执行
- [ ] `.lintstagedrc` 存在
- [ ] package.json 中的 `prepare` script 为 `"husky"`
- [ ] `prettier` 配置存在
- [ ] 运行 `npx lint-staged` 验证其正常工作

### 8. 提交

暂存所有已更改或创建的文件，并使用以下消息提交：`Add pre-commit hooks (husky + lint-staged + prettier)`

这会经过新的 pre-commit hook，是一次很好的冒烟测试，可验证一切正常工作。

## 注意事项

- Husky v9+ 的 hook 文件不需要 shebang
- `prettier --ignore-unknown` 会跳过 Prettier 无法解析的文件（图像等）
- pre-commit 会先运行 lint-staged（速度快，仅处理已暂存文件），然后运行完整的类型检查和测试
