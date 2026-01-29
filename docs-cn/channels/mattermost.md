---
summary: "Mattermost 机器人设置和 Moltbot 配置"
read_when:
  - 设置 Mattermost
  - 调试 Mattermost 路由
---

# Mattermost (插件)

状态: 通过插件支持(bot 令牌 + WebSocket 事件)。支持频道、群组和私信。Mattermost 是一个可自托管的消息平台；有关产品详细信息和下载，请访问官方网站 [mattermost.com](https://mattermost.com)。

## 需要插件
Mattermost 作为插件提供，不包含在核心安装中。

通过 CLI 安装(npm 注册表):
```bash
moltbot plugins install @moltbot/mattermost
```

本地检出(从 git 仓库运行时):
```bash
moltbot plugins install ./extensions/mattermost
```

如果您在配置/入职过程中选择了 Mattermost，并且检测到 git 检出，Moltbot 将自动提供本地安装路径。

详情: [插件](/plugin)

## 快速设置
1) 安装 Mattermost 插件。
2) 创建 Mattermost 机器人账户并复制 **bot 令牌**。
3) 复制 Mattermost **基础 URL**(例如 `https://chat.example.com`)。
4) 配置 Moltbot 并启动网关。

最小配置:
```json5
{
  channels: {
    mattermost: {
      enabled: true,
      botToken: "mm-token",
      baseUrl: "https://chat.example.com",
      dmPolicy: "pairing"
    }
  }
}
```

## 环境变量(默认账户)
如果您更喜欢环境变量，请在网关主机上设置这些:

- `MATTERMOST_BOT_TOKEN=...`
- `MATTERMOST_URL=https://chat.example.com`

环境变量仅适用于**默认**账户(`default`)。其他账户必须使用配置值。

## 聊天模式
Mattermost 自动响应私信。频道行为由 `chatmode` 控制:

- `oncall`(默认): 仅在频道中被 @提及时响应。
- `onmessage`: 响应每条频道消息。
- `onchar`: 当消息以触发前缀开头时响应。

配置示例:
```json5
{
  channels: {
    mattermost: {
      chatmode: "onchar",
      oncharPrefixes: [">", "!"]
    }
  }
}
```

注意事项:
- `onchar` 仍然响应显式 @提及。
- `channels.mattermost.requireMention` 适用于旧配置，但首选 `chatmode`。

## 访问控制(私信)
- 默认: `channels.mattermost.dmPolicy = "pairing"`(未知发送者会收到配对代码)。
- 通过以下方式批准:
  - `moltbot pairing list mattermost`
  - `moltbot pairing approve mattermost <CODE>`
- 公开私信: `channels.mattermost.dmPolicy="open"` 加上 `channels.mattermost.allowFrom=["*"]`。

## 频道(群组)
- 默认: `channels.mattermost.groupPolicy = "allowlist"`(提及限制)。
- 使用 `channels.mattermost.groupAllowFrom` 允许列表发送者(用户 ID 或 `@username`)。
- 公开频道: `channels.mattermost.groupPolicy="open"`(提及限制)。

## 出站传递的目标
使用 `moltbot message send` 或 cron/webhooks 的这些目标格式:

- `channel:<id>` 用于频道
- `user:<id>` 用于私信
- `@username` 用于私信(通过 Mattermost API 解析)

裸 ID 被视为频道。

## 多账户
Mattermost 支持在 `channels.mattermost.accounts` 下的多个账户:

```json5
{
  channels: {
    mattermost: {
      accounts: {
        default: { name: "Primary", botToken: "mm-token", baseUrl: "https://chat.example.com" },
        alerts: { name: "Alerts", botToken: "mm-token-2", baseUrl: "https://alerts.example.com" }
      }
    }
  }
}
```

## 故障排除
- 频道中没有回复: 确保机器人在频道中并提及它(oncall)，使用触发前缀(onchar)，或设置 `chatmode: "onmessage"`。
- 身份验证错误: 检查 bot 令牌、基础 URL 以及账户是否已启用。
- 多账户问题: 环境变量仅适用于 `default` 账户。
