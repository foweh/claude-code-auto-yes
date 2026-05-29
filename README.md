# Claude Code 自动跳过权限确认（Auto Yes）

## 问题

Claude Code 每次执行 Bash 命令、编辑文件时都会弹窗让你确认：

```
Do you want to proceed? (y/n)
```

频繁打断，影响效率。本文教你如何让 Claude Code 自动跳过所有确认。

## 解决：修改 `settings.json`

### 配置文件位置

| 范围 | 路径 | 影响 |
|---|---|---|
| 全局 | `~/.claude/settings.json` | 所有项目生效 |
| 当前项目 | `项目根目录/.claude/settings.json` | 仅此项目 |

Windows 上 `~` = `C:\Users\<用户名>`。

### 添加权限配置

在 `settings.json` 中加 `permissions` 段：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-xxxx",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic"
  },
  "model": "deepseek-v4-pro",
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```

### 三种模式对比

| 模式 | 效果 | 适用场景 |
|---|---|---|
| `bypassPermissions` | 跳过**全部**确认弹窗 | 个人项目、隔离环境 |
| `acceptEdits` | 自动通过文件编辑，Bash 仍需确认 | 日常开发（推荐） |
| `default` | 每个新工具首次使用时确认 | 默认行为 |

### 精细化控制（可选）

如果不想全跳，可以按命令精确控制：

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",
      "Bash(git *)",
      "Bash(pip *)",
      "Bash(go *)",
      "Edit",
      "Read",
      "WebFetch"
    ],
    "deny": [
      "Bash(rm -rf *)",
      "Bash(curl *)"
    ]
  }
}
```

### 安全提醒

`bypassPermissions` 会让 Claude Code 直接执行所有操作。以下命令依然会拦截：

- `rm -rf /`（删除根目录）
- `rm -rf ~`（删除用户目录）

但仍建议：
- 不在生产服务器上使用 `bypassPermissions`
- 团队项目用 `acceptEdits` 更安全
- 善用 `deny` 规则阻止危险命令

## 完整配置示例

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-你的API-Key",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "deepseek-v4-flash",
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "deepseek-v4-pro",
    "ANTHROPIC_MODEL": "deepseek-v4-pro",
    "CLAUDE_CODE_DISABLE_AUTOUPDATER": "1"
  },
  "includeCoAuthoredBy": false,
  "model": "deepseek-v4-pro",
  "effortLevel": "xhigh",
  "permissions": {
    "defaultMode": "bypassPermissions"
  }
}
```

## 相关链接

- [Claude Code 权限文档](https://code.claude.com/docs/en/permissions)
- [Claude Code 设置文档](https://code.claude.com/docs/en/settings)
- [Claude Code + DeepSeek 代理修复](https://github.com/railfowehup/claude-deepseek-fix)
