---
summary: "通过 imsg 实现 iMessage 支持（通过 stdio 的 JSON-RPC）、设置和 chat_id 路由"
read_when:
  - 设置 iMessage 支持
  - 调试 iMessage 发送/接收
---
# iMessage (imsg)


状态：外部 CLI 集成。网关生成 `imsg rpc`（通过 stdio 的 JSON-RPC）。

## 快速设置（初学者）
1) 确保 Messages 在此 Mac 上已登录。
2) 安装 `imsg`：
   - `brew install steipete/tap/imsg`
3) 使用 `channels.imessage.cliPath` 和 `channels.imessage.dbPath` 配置 Moltbot。
4) 启动网关并批准任何 macOS 提示（Automation + Full Disk Access）。

最小配置：
```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "/usr/local/bin/imsg",
      dbPath: "/Users/<you>/Library/Messages/chat.db"
    }
  }
}
```

## 它是什么
- 通过 macOS 上的 `imsg` 支持的 iMessage 渠道。
- 确定性路由：回复始终返回到 iMessage。
- DM 共享 agent 的主会话；群组被隔离（`agent:<agentId>:imessage:group:<chat_id>`）。
- 如果多参与者线程以 `is_group=false` 到达，您仍然可以通过 `chat_id` 使用 `channels.imessage.groups` 隔离它（请参阅下面的"类群组线程"）。

## 配置写入
默认情况下，iMessage 被允许写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。

禁用：
```json5
{
  channels: { imessage: { configWrites: false } }
}
```

## 要求
- 已登录 Messages 的 macOS。
- Moltbot + `imsg` 的完全磁盘访问权限（Messages DB 访问）。
- 发送时的 Automation 权限。
- `channels.imessage.cliPath` 可以指向代理 stdin/stdout 的任何命令（例如，一个包装脚本，SSH 到另一个 Mac 并运行 `imsg rpc`）。

## 设置（快速路径）
1) 确保 Messages 在此 Mac 上已登录。
2) 配置 iMessage 并启动网关。

### 专用机器人 macOS 用户（用于隔离身份）
如果您希望机器人从**单独的 iMessage 身份**发送（并保持您的个人 Messages 整洁），请使用专用的 Apple ID + 专用的 macOS 用户。

1. 创建一个专用的 Apple ID（例如，`my-cool-bot@icloud.com`）。
   - Apple 可能需要电话号码进行验证/2FA。
2. 创建一个 macOS 用户（例如，`clawdshome`）并登录。
3. 在该 macOS 用户中打开 Messages 并使用机器人 Apple ID 登录 iMessage。
4. 启用远程登录（系统设置 → 常规 → 共享 → 远程登录）。
5. 安装 `imsg`：
   - `brew install steipete/tap/imsg`
6. 设置 SSH 以便 `ssh <bot-macos-user>@localhost true` 无需密码即可工作。
7. 将 `channels.imessage.accounts.bot.cliPath` 指向一个运行 `imsg` 作为机器人用户的 SSH 包装器。

首次运行注意：在*机器人 macOS 用户*中，发送/接收可能需要 GUI 批准（Automation + Full Disk Access）。如果 `imsg rpc` 看起来卡住或退出，请登录该用户（屏幕共享有帮助），运行一次 `imsg chats --limit 1` / `imsg send ...`，批准提示，然后重试。

示例包装器（`chmod +x`）。将 `<bot-macos-user>` 替换为您的实际 macOS 用户名：
```bash
#!/usr/bin/env bash
set -euo pipefail

# 首先运行一次交互式 SSH 以接受主机密钥：
#   ssh <bot-macos-user>@localhost true
exec /usr/bin/ssh -o BatchMode=yes -o ConnectTimeout=5 -T <bot-macos-user>@localhost \
  "/usr/local/bin/imsg" "$@"
```

示例配置：
```json5
{
  channels: {
    imessage: {
      enabled: true,
      accounts: {
        bot: {
          name: "Bot",
          enabled: true,
          cliPath: "/path/to/imsg-bot",
          dbPath: "/Users/<bot-macos-user>/Library/Messages/chat.db"
        }
      }
    }
  }
}
```

对于单账户设置，请使用平面选项（`channels.imessage.cliPath`、`channels.imessage.dbPath`）而不是 `accounts` 映射。

### 远程/SSH 变体（可选）
如果您希望 iMessage 在另一台 Mac 上，请将 `channels.imessage.cliPath` 设置为通过 SSH 在远程 macOS 主机上运行 `imsg` 的包装器。Moltbot 仅需要 stdio。

示例包装器：
```bash
#!/usr/bin/env bash
exec ssh -T gateway-host imsg "$@"
```

**远程附件**：当 `cliPath` 通过 SSH 指向远程主机时，Messages 数据库中的附件路径引用远程机器上的文件。通过设置 `channels.imessage.remoteHost`，Moltbot 可以自动通过 SCP 获取这些文件：

```json5
{
  channels: {
    imessage: {
      cliPath: "~/imsg-ssh",                     // 到远程 Mac 的 SSH 包装器
      remoteHost: "user@gateway-host",           // 用于 SCP 文件传输
      includeAttachments: true
    }
  }
}
```

如果未设置 `remoteHost`，Moltbot 会尝试通过解析包装脚本中的 SSH 命令来自动检测它。为了可靠性，建议进行显式配置。

#### 通过 Tailscale 的远程 Mac（示例）
如果网关在 Linux 主机/VM 上运行，但 iMessage 必须在 Mac 上运行，Tailscale 是最简单的桥梁：网关通过 tailnet 与 Mac 通信，通过 SSH 运行 `imsg`，并将 SCP 附件传回。

架构：
```
┌──────────────────────────────┐          SSH (imsg rpc)          ┌──────────────────────────┐
│ Gateway host (Linux/VM)      │──────────────────────────────────▶│ Mac with Messages + imsg │
│ - moltbot gateway           │          SCP (attachments)        │ - Messages signed in     │
│ - channels.imessage.cliPath  │◀──────────────────────────────────│ - Remote Login enabled   │
└──────────────────────────────┘                                   └──────────────────────────┘
              ▲
              │ Tailscale tailnet (hostname or 100.x.y.z)
              ▼
        user@gateway-host
```

具体配置示例（Tailscale 主机名）：
```json5
{
  channels: {
    imessage: {
      enabled: true,
      cliPath: "~/.clawdbot/scripts/imsg-ssh",
      remoteHost: "bot@mac-mini.tailnet-1234.ts.net",
      includeAttachments: true,
      dbPath: "/Users/bot/Library/Messages/chat.db"
    }
  }
}
```

示例包装器（`~/.clawdbot/scripts/imsg-ssh`）：
```bash
#!/usr/bin/env bash
exec ssh -T bot@mac-mini.tailnet-1234.ts.net imsg "$@"
```

注意：
- 确保 Mac 已登录 Messages，并且启用了远程登录。
- 使用 SSH 密钥，以便 `ssh bot@mac-mini.tailnet-1234.ts.net` 无需提示即可工作。
- `remoteHost` 应该与 SSH 目标匹配，以便 SCP 可以获取附件。

多账户支持：使用 `channels.imessage.accounts` 和每个账户的配置以及可选的 `name`。请参阅 [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) 以了解共享模式。不要提交 `~/.clawdbot/moltbot.json`（它通常包含令牌）。

## 访问控制（DM + 群组）
DM：
- 默认：`channels.imessage.dmPolicy = "pairing"`。
- 未知发送者收到配对码；在批准之前消息将被忽略（配对码在 1 小时后过期）。
- 通过以下方式批准：
  - `moltbot pairing list imessage`
  - `moltbot pairing approve imessage <CODE>`
- 配对是 iMessage DM 的默认令牌交换方式。详情：[配对](/start/pairing)

群组：
- `channels.imessage.groupPolicy = open | allowlist | disabled`。
- `channels.imessage.groupAllowFrom` 控制在设置为 `allowlist` 时谁可以在群组中触发。
- 提及门控使用 `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`），因为 iMessage 没有原生提及元数据。
- 多 agent 覆盖：在 `agents.list[].groupChat.mentionPatterns` 上设置每个 agent 的模式。

## 工作原理（行为）
- `imsg` 流式传输消息事件；网关将它们规范化为共享频道信封。
- 回复始终路由回相同的聊天 id 或句柄。

## 类群组线程（`is_group=false`）
一些 iMessage 线程可以有多个参与者，但根据 Messages 存储聊天标识符的方式，仍然以 `is_group=false` 到达。

如果您在 `channels.imessage.groups` 下显式配置了 `chat_id`，Moltbot 会将该线程视为"群组"用于：
- 会话隔离（单独的 `agent:<agentId>:imessage:group:<chat_id>` 会话密钥）
- 群组白名单/提及门控行为

示例：
```json5
{
  channels: {
    imessage: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "42": { "requireMention": false }
      }
    }
  }
}
```
当您想要为特定线程使用隔离的个性/模型时，这很有用（请参阅 [多 agent 路由](/concepts/multi-agent)）。对于文件系统隔离，请参阅 [沙盒](/gateway/sandboxing)。

## 媒体 + 限制
- 通过 `channels.imessage.includeAttachments` 可选附件摄取。
- 通过 `channels.imessage.mediaMaxMb` 设置媒体上限。

## 限制
- 出站文本被分块为 `channels.imessage.textChunkLimit`（默认 4000）。
- 可选的换行符分块：设置 `channels.imessage.chunkMode="newline"` 在长度分块之前在空行（段落边界）处拆分。
- 媒体上传受 `channels.imessage.mediaMaxMb` 限制（默认 16）。

## 寻址 / 投递目标
首选 `chat_id` 以进行稳定路由：
- `chat_id:123`（首选）
- `chat_guid:...`
- `chat_identifier:...`
- 直接句柄：`imessage:+1555` / `sms:+1555` / `user@example.com`

列出聊天：
```
imsg chats --limit 20
```

## 配置参考（iMessage）
完整配置：[配置](/gateway/configuration)

提供者选项：
- `channels.imessage.enabled`：启用/禁用渠道启动。
- `channels.imessage.cliPath`：`imsg` 的路径。
- `channels.imessage.dbPath`：Messages DB 路径。
- `channels.imessage.remoteHost`：当 `cliPath` 指向远程 Mac 时用于 SCP 附件传输的 SSH 主机（例如，`user@gateway-host`）。如果未设置，则从 SSH 包装器自动检测。
- `channels.imessage.service`：`imessage | sms | auto`。
- `channels.imessage.region`：SMS 区域。
- `channels.imessage.dmPolicy`：`pairing | allowlist | open | disabled`（默认：pairing）。
- `channels.imessage.allowFrom`：DM 白名单（句柄、电子邮件、E.164 号码或 `chat_id:*`）。`open` 需要 `"*"`。iMessage 没有用户名；使用句柄或聊天目标。
- `channels.imessage.groupPolicy`：`open | allowlist | disabled`（默认：allowlist）。
- `channels.imessage.groupAllowFrom`：群组发送者白名单。
- `channels.imessage.historyLimit` / `channels.imessage.accounts.*.historyLimit`：要包括为上下文的最大群组消息数（0 禁用）。
- `channels.imessage.dmHistoryLimit`：以用户回合为单位的 DM 历史限制。每个用户覆盖：`channels.imessage.dms["<handle>"].historyLimit`。
- `channels.imessage.groups`：每个群组默认值 + 白名单（使用 `"*"` 表示全局默认值）。
- `channels.imessage.includeAttachments`：将附件摄取到上下文中。
- `channels.imessage.mediaMaxMb`：入站/出站媒体上限（MB）。
- `channels.imessage.textChunkLimit`：出站分块大小（字符）。
- `channels.imessage.chunkMode`：`length`（默认）或 `newline` 在长度分块之前在空行（段落边界）处拆分。

相关全局选项：
- `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）。
- `messages.responsePrefix`。
