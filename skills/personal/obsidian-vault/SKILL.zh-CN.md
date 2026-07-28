---
name: obsidian-vault
description: 使用 wikilink 和索引笔记，在 Obsidian vault 中搜索、创建和管理笔记。当用户想要在 Obsidian 中查找、创建或整理笔记时使用。
---

# Obsidian Vault

## Vault 位置

`/mnt/d/Obsidian Vault/AI Research/`

根目录基本采用扁平结构。

## 命名约定

- **索引笔记**：聚合相关主题（例如 `Ralph Wiggum Index.md`、`Skills Index.md`、`RAG Index.md`）
- 所有笔记名称使用 **Title Case**
- 不使用文件夹进行组织，改用链接和索引笔记

## 链接

- 使用 Obsidian 的 `[[wikilinks]]` 语法：`[[Note Title]]`
- 笔记在底部链接到依赖笔记或相关笔记
- 索引笔记只是 `[[wikilinks]]` 列表

## 工作流

### 搜索笔记

```bash
# Search by filename
find "/mnt/d/Obsidian Vault/AI Research/" -name "*.md" | grep -i "keyword"

# Search by content
grep -rl "keyword" "/mnt/d/Obsidian Vault/AI Research/" --include="*.md"
```

或者直接在 vault 路径上使用 Grep/Glob 工具。

### 创建新笔记

1. 文件名使用 **Title Case**
2. 按照 vault 规则，将内容写成一个学习单元
3. 在底部添加指向相关笔记的 `[[wikilinks]]`
4. 如果属于编号序列，使用分层编号方案

### 查找相关笔记

在整个 vault 中搜索 `[[Note Title]]` 以查找反向链接：

```bash
grep -rl "\\[\\[Note Title\\]\\]" "/mnt/d/Obsidian Vault/AI Research/"
```

### 查找索引笔记

```bash
find "/mnt/d/Obsidian Vault/AI Research/" -name "*Index*"
```
