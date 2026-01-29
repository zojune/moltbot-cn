---
summary: "设置指南：保持你的 Moltbot 设置定制化，同时保持最新"
read_when:
  - 设置新机器
  - 你想要"最新 + 最强大"而不会破坏你的个人设置
---

# 设置

最后更新：2026-01-01

## TL;DR
- **定制化在仓库外：**`~/clawd`（工作区）+ `~/.clawdbot/moltbot.json`（配置）。
- **稳定工作流：**安装 macOS 应用；让它运行捆绑的网关。
- **前沿工作流：**通过 `pnpm gateway:watch` 自己运行网关，然后让 macOS 应用以本地模式附加。

## 前提条件（从源代码）
- Node `>=22`
- `pnpm`
- Docker（可选；仅用于容器化设置/e2e — 参见 [Docker](/install/docker)）

## 定制策略（因此更新不会伤害你）

如果你想要"100% 为我定制"*并且*易于更新，请将你的定制保留在：
- **配置：**`~/.clawdbot/moltbot.json`（JSON/JSON5-like）
- **工作区：**`~/clawd`（技能、提示、记忆；使其成为私有 git 仓库）

引导一次：

```bash
moltbot setup
```

从此仓库内部，使用本地 CLI 入口：

```bash
moltbot setup
```

如果你还没有全局安装，请通过 `pnpm moltbot setup` 运行它。

## 稳定工作流（macOS 应用优先）

1) 安装 + 启动 **Moltbot.app**（菜单栏）。
2) 完成入门/权限检查清单（TCC 提示）。
3) 确保网关是**本地**的并且正在运行（应用管理它）。
4) 链接表面（示例：WhatsApp）：

```bash
moltbot channels login
```

5) 完整性检查：

```bash
moltbot health
```

如果入门在您的构建中不可用：
- 运行 `moltbot setup`，然后 `moltbot channels login`，然后手动启动网关（`moltbot gateway`）。

## 前沿工作流（网关在终端中）

目标：在 TypeScript 网关上工作，获得热重载，保持 macOS 应用 UI 附加。

### 0)（可选）也从源代码运行 macOS 应用

如果你也想要边缘的 macOS 应用：

```bash
./scripts/restart-mac.sh
```

### 1) 启动开发网关

```bash
pnpm install
pnpm gateway:watch
```

`gateway:watch` 在监视模式下运行网关，并在 TypeScript 更改时重新加载。

### 2) 将 macOS 应用指向你运行的网关

在 **Moltbot.app** 中：

- 连接模式：**本地**
应用将附加到配置端口上运行的网关。

### 3) 验证

- 应用内网关状态应显示 **"Using existing gateway …"**
- 或通过 CLI：

```bash
moltbot health
```

### 常见陷阱
- **错误的端口：**网关 WS 默认为 `ws://127.0.0.1:18789`；保持应用 + CLI 在同一端口上。
- **状态存储位置：**
  - 凭据：`~/.clawdbot/credentials/`
  - 会话：`~/.clawdbot/agents/<agentId>/sessions/`
  - 日志：`/tmp/moltbot/`

## 凭据存储地图

调试认证或决定备份什么时使用此：

- **WhatsApp**：`~/.clawdbot/credentials/whatsapp/<accountId>/creds.json`
- **Telegram 机器人令牌**：配置/env 或 `channels.telegram.tokenFile`
- **Discord 机器人令牌**：配置/env（尚不支持令牌文件）
- **Slack 令牌**：配置/env（`channels.slack.*`）
- **配对允许列表**：`~/.clawdbot/credentials/<channel>-allowFrom.json`
- **模型认证配置文件**：`~/.clawdbot/agents/<agentId>/agent/auth-profiles.json`
- **旧版 OAuth 导入**：`~/.clawdbot/credentials/oauth.json`
更多详情：[安全](/gateway/security#credential-storage-map)。

## 更新（不会破坏你的设置）

- 保持 `~/clawd` 和 `~/.clawdbot/` 为"你的东西"；不要将个人提示/配置放入 `moltbot` 仓库。
- 更新源代码：`git pull` + `pnpm install`（当 lockfile 更改时）+ 继续使用 `pnpm gateway:watch`。

## Linux（systemd 用户服务）

Linux 安装使用 systemd **用户**服务。默认情况下，systemd 在注销/空闲时停止用户服务，这会杀死网关。入门尝试为你启用 lingering（可能会提示 sudo）。如果仍然关闭，请运行：

```bash
sudo loginctl enable-linger $USER
```

对于始终在线或多用户服务器，考虑使用**系统**服务而不是用户服务（不需要 lingering）。参见 [网关运行手册](/gateway) 了解 systemd 说明。

## 相关文档

- [网关运行手册](/gateway)（标志、监管、端口）
- [网关配置](/gateway/configuration)（配置架构 + 示例）
- [Discord](/channels/discord) 和 [Telegram](/channels/telegram)（回复标签 + replyToMode 设置）
- [Moltbot 助手设置](/start/clawd)
- [macOS 应用](/platforms/macos)（网关生命周期）
