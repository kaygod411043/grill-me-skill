---
name: git-guardrails-claude-code
description: 设置 Claude Code hook，在危险的 git 命令（push、reset --hard、clean、branch -D 等）执行前将其阻止。当用户想要防止破坏性 git 操作、添加 git 安全 hook，或在 Claude Code 中阻止 git push/reset 时使用。
---

# 设置 Git 防护栏

设置一个 `PreToolUse` hook，在 Claude 执行危险的 git 命令之前进行拦截和阻止。

## 会阻止哪些操作

- `git push`（包括 `--force` 在内的所有变体）
- `git reset --hard`
- `git clean -f` / `git clean -fd`
- `git branch -D`
- `git checkout .` / `git restore .`

命令被阻止时，Claude 会看到一条消息，告知它没有权限使用这些命令。

## 步骤

### 1. 询问作用范围

询问用户：只为**当前项目**（`.claude/settings.json`）安装，还是为**所有项目**（`~/.claude/settings.json`）安装？

### 2. 复制 hook 脚本

内置脚本位于：[scripts/block-dangerous-git.sh](scripts/block-dangerous-git.sh)

根据作用范围将其复制到目标位置：

- **项目**：`.claude/hooks/block-dangerous-git.sh`
- **全局**：`~/.claude/hooks/block-dangerous-git.sh`

使用 `chmod +x` 赋予其可执行权限。

### 3. 将 hook 添加到设置

添加到相应的设置文件：

**项目**（`.claude/settings.json`）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

**全局**（`~/.claude/settings.json`）：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "~/.claude/hooks/block-dangerous-git.sh"
          }
        ]
      }
    ]
  }
}
```

如果设置文件已经存在，将该 hook 合并到现有的 `hooks.PreToolUse` 数组中，不要覆盖其他设置。

### 4. 询问是否需要自定义

询问用户是否想在阻止列表中添加或移除任何模式。相应地编辑复制出的脚本。

### 5. 验证

运行一次快速测试：

```bash
echo '{"tool_input":{"command":"git push origin main"}}' | <path-to-script>
```

应以状态码 2 退出，并向 stderr 输出一条 BLOCKED 消息。
