# Claude Code 自动跳过确认弹窗

> 用 Claude Code 写代码时，每执行一个命令都要手动输 `y` 确认，太烦了！看完这篇，5 分钟搞定。

---

## 先搞懂：为什么 Claude Code 老弹窗？

Claude Code 的设计理念是「安全第一」，每次执行以下操作都会让你确认：

| 操作类型 | 例子 | 默认行为 |
|---|---|---|
| 运行命令 | `npm install`、`git push` | 每次弹窗问 `y/n` |
| 编辑文件 | 修改 `.ts`、`.py` 文件 | 每个文件首次改时弹窗 |
| 网络请求 | `curl`、下载文件 | 弹窗 |
| 读取文件 | `cat xxx.ts` | 不弹窗（只读比较安全） |

弹窗长这样：

```
⏺ Do you want to proceed? (y/n) ▏
```

这就是我们要消灭的东西。

---

## 三步解决

### 第一步：找到配置文件

Claude Code 的配置存在一个叫 `settings.json` 的文件里。打开方式：

**方法 A — 在终端里敲一行（推荐）**

打开你的终端（CMD / PowerShell / VSCode 内置终端），输入：

```bash
claude /config
```

这会直接打开 Claude Code 的设置界面，里面能看到所有配置项。

**方法 B — 直接找文件**

| 你的系统 | 文件路径 |
|---|---|
| Windows | `C:\Users\你的用户名\.claude\settings.json` |
| Mac | `/Users/你的用户名/.claude/settings.json` |
| Linux | `~/.claude/settings.json` |

> **Windows 用户**：`你的用户名` 就是你开机登录那个名字。不知道的话，在终端敲 `echo %USERPROFILE%` 就能看到完整路径。

用任意编辑器打开这个文件（记事本 / VSCode / Notepad++ 都行）。

---

### 第二步：添加配置

找到你的 `settings.json`，里面大概是这样的：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-xxxx",
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic"
  },
  "model": "deepseek-v4-pro"
}
```

**在最后一个 `}` 前面，添加 `permissions` 段**。改完后长这样：

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

> ⚠️ **注意**：JSON 格式很严格！每个键值对后面要有逗号（最后一个除外），大括号要对齐。不确定的话直接复制下面的完整版。

---

### 第三步：保存，完事

保存文件，关了重开 Claude Code，或者直接在运行中的 Claude Code 里敲：

```
/clear
```

然后试试让 Claude Code 帮你跑个命令，比如 `ls` 或 `npm install`，你会发现弹窗消失了，直接执行。

---

## 复制即用的完整配置

以下是可直接使用的完整 `settings.json`（DeepSeek 用户专用版）：

```json
{
  "env": {
    "ANTHROPIC_AUTH_TOKEN": "sk-换成你的API-Key",
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

> 如果你用的是 Anthropic 官方 API，把 `ANTHROPIC_BASE_URL` 删掉就行。API Key 记得换成你自己的。

---

## 三种模式的通俗解释

| 模式 | 一句话解释 | 什么时候用 |
|---|---|---|
| `bypassPermissions` | 什么都自动同意，不问你 | 自己写个人项目时 |
| `acceptEdits` | 改文件自动同意，跑命令还是要问 | 和同事合作的项目 |
| `default` | 一切照旧，该问就问 | 不确定就别改 |

**建议**：自己电脑上的个人项目用 `bypassPermissions`，公司的用 `acceptEdits`。

---

## 安全吗？

`bypassPermissions` 跳过了确认弹窗，但 Claude Code 有两道底线：

1. **`rm -rf /`** — 删除整个磁盘 → 依然拦截
2. **`rm -rf ~`** — 删除你的用户目录 → 依然拦截

所以不至于把你的电脑搞崩。但 Claude Code 仍然会直接改你的代码文件，所以：

- 用了 Git 的项目随便开（改坏了能回滚）
- 没做备份的生产服务器别开
- 先用 `/config` 在项目级的 `.claude/settings.json` 里改，别改全局的

---

## 高级玩法：只跳过某些命令

如果你只想自动同意 `npm`、`git`，但 `rm`、`curl` 还是要问：

```json
{
  "permissions": {
    "allow": [
      "Bash(npm *)",
      "Bash(git *)",
      "Bash(pip *)",
      "Bash(go *)",
      "Edit",
      "Read"
    ],
    "deny": [
      "Bash(rm *)",
      "Bash(curl *)",
      "Bash(wget *)"
    ]
  }
}
```

这样 `npm install` 自动过，`rm -rf node_modules` 还是会弹窗问你。

---

## 常见问题

**Q: 改了没生效？**
A: 在 Claude Code 里敲 `/clear`，或者直接关掉重开。

**Q: JSON 报错说格式不对？**
A: 多半是少了逗号或者多了逗号。复制上面的「完整配置」直接覆盖最省事。

**Q: VSCode 里的 Claude Code 怎么配置？**
A: 一样的，Claude Code 在 VSCode 里就是个终端，配同一个 `settings.json` 就行。

**Q: 我只想让某个项目自动确认，其他项目照旧？**
A: 在项目根目录建 `.claude/settings.json`，只写 `permissions` 段，不影响其他项目。

---

## 相关资源

- [Claude Code 官方权限文档](https://code.claude.com/docs/en/permissions)
- [Claude Code 官方配置文档](https://code.claude.com/docs/en/settings)
- [Claude Code + DeepSeek 400 错误修复](https://github.com/foweh/claude-deepseek-fix)
