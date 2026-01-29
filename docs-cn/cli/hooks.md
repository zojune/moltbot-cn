---
summary: "`moltbot hooks` CLI 参考(agent hooks)"
read_when:
  - 您想管理 agent hooks
  - 您想安装或更新 hooks
---

# `moltbot hooks`

管理 agent hooks(事件驱动的自动化,用于 `/new`、`/reset` 等命令以及 gateway 启动)。

相关:
- Hooks: [Hooks](/hooks)
- 插件 hooks: [Plugins](/plugin#plugin-hooks)

## 列出所有 Hooks

```bash
moltbot hooks list
```

列出从工作区、托管和捆绑目录中发现的所有 hooks。

**选项:**
- `--eligible`:仅显示符合条件的 hooks(满足要求)
- `--json`:输出为 JSON
- `-v, --verbose`:显示详细信息,包括缺失的要求

**示例输出:**

```
Hooks (4/4 ready)

Ready:
  🚀 boot-md ✓ - Run BOOT.md on gateway startup
  📝 command-logger ✓ - Log all command events to a centralized audit file
  💾 session-memory ✓ - Save session context to memory when /new command is issued
  😈 soul-evil ✓ - Swap injected SOUL content during a purge window or by random chance
```

**示例(详细):**

```bash
moltbot hooks list --verbose
```

显示不符合条件的 hooks 的缺失要求。

**示例(JSON):**

```bash
moltbot hooks list --json
```

返回用于程序化使用的结构化 JSON。

## 获取 Hook 信息

```bash
moltbot hooks info <name>
```

显示特定 hook 的详细信息。

**参数:**
- `<name>`:Hook 名称(例如 `session-memory`)

**选项:**
- `--json`:输出为 JSON

**示例:**

```bash
moltbot hooks info session-memory
```

**输出:**

```
💾 session-memory ✓ Ready

Save session context to memory when /new command is issued

Details:
  Source: moltbot-bundled
  Path: /path/to/moltbot/hooks/bundled/session-memory/HOOK.md
  Handler: /path/to/moltbot/hooks/bundled/session-memory/handler.ts
  Homepage: https://docs.molt.bot/hooks#session-memory
  Events: command:new

Requirements:
  Config: ✓ workspace.dir
```

## 检查 Hooks 资格

```bash
moltbot hooks check
```

显示 hook 资格状态摘要(准备就绪与未就绪的数量)。

**选项:**
- `--json`:输出为 JSON

**示例输出:**

```
Hooks Status

Total hooks: 4
Ready: 4
Not ready: 0
```

## 启用 Hook

```bash
moltbot hooks enable <name>
```

通过将其添加到您的配置(`~/.clawdbot/config.json`)来启用特定的 hook。

**注意:**插件管理的 hooks 在 `moltbot hooks list` 中显示 `plugin:<id>`,不能在此处启用/禁用。改为启用/禁用插件。

**参数:**
- `<name>`:Hook 名称(例如 `session-memory`)

**示例:**

```bash
moltbot hooks enable session-memory
```

**输出:**

```
✓ Enabled hook: 💾 session-memory
```

**它的作用:**
- 检查 hook 是否存在并符合条件
- 在配置中更新 `hooks.internal.entries.<name>.enabled = true`
- 将配置保存到磁盘

**启用后:**
- 重启 gateway 以重新加载 hooks(macOS 上的菜单栏应用重启,或在开发中重启您的 gateway 进程)。

## 禁用 Hook

```bash
moltbot hooks disable <name>
```

通过更新您的配置来禁用特定的 hook。

**参数:**
- `<name>`:Hook 名称(例如 `command-logger`)

**示例:**

```bash
moltbot hooks disable command-logger
```

**输出:**

```
⏸ Disabled hook: 📝 command-logger
```

**禁用后:**
- 重启 gateway 以重新加载 hooks

## 安装 Hooks

```bash
moltbot hooks install <path-or-spec>
```

从本地文件夹/归档文件或 npm 安装 hook 包。

**它的作用:**
- 将 hook 包复制到 `~/.clawdbot/hooks/<id>`
- 在 `hooks.internal.entries.*` 中启用已安装的 hooks
- 在 `hooks.internal.installs` 下记录安装

**选项:**
- `-l, --link`:链接本地目录而不是复制(将其添加到 `hooks.internal.load.extraDirs`)

**支持的归档文件:**`.zip`、`.tgz`、`.tar.gz`、`.tar`

**示例:**

```bash
# 本地目录
moltbot hooks install ./my-hook-pack

# 本地归档文件
moltbot hooks install ./my-hook-pack.zip

# NPM 包
moltbot hooks install @moltbot/my-hook-pack

# 链接本地目录而不复制
moltbot hooks install -l ./my-hook-pack
```

## 更新 Hooks

```bash
moltbot hooks update <id>
moltbot hooks update --all
```

更新已安装的 hook 包(仅 npm 安装)。

**选项:**
- `--all`:更新所有跟踪的 hook 包
- `--dry-run`:显示将要更改的内容而不写入

## 捆绑的 Hooks

### session-memory

当您发出 `/new` 时,将会话上下文保存到内存。

**启用:**

```bash
moltbot hooks enable session-memory
```

**输出:**`~/clawd/memory/YYYY-MM-DD-slug.md`

**参见:**[session-memory 文档](/hooks#session-memory)

### command-logger

将所有命令事件记录到集中式审计文件。

**启用:**

```bash
moltbot hooks enable command-logger
```

**输出:**`~/.clawdbot/logs/commands.log`

**查看日志:**

```bash
# 最近的命令
tail -n 20 ~/.clawdbot/logs/commands.log

# 漂亮打印
cat ~/.clawdbot/logs/commands.log | jq .

# 按操作过滤
grep '"action":"new"' ~/.clawdbot/logs/commands.log | jq .
```

**参见:**[command-logger 文档](/hooks#command-logger)

### soul-evil

在清除窗口期间或随机机会,将注入的 `SOUL.md` 内容与 `SOUL_EVIL.md` 交换。

**启用:**

```bash
moltbot hooks enable soul-evil
```

**参见:**[SOUL Evil Hook](/hooks/soul-evil)

### boot-md

当 gateway 启动时运行 `BOOT.md`(频道启动后)。

**事件**:`gateway:startup`

**启用**:

```bash
moltbot hooks enable boot-md
```

**参见:**[boot-md 文档](/hooks#boot-md)
