---
summary: "LINE Messaging API 插件设置、配置和使用"
read_when:
  - 您想将 Moltbot 连接到 LINE
  - 您需要 LINE webhook + 凭据设置
  - 您想要 LINE 特定的消息选项
---

# LINE (plugin)

LINE 通过 LINE Messaging API 连接到 Moltbot。插件在网关上作为 webhook 接收器运行，并使用您的频道访问令牌 + 频道密钥进行身份验证。

状态：通过插件支持。支持直接消息、群组聊天、媒体、位置、Flex 消息、模板消息和快速回复。不支持反应和线程。

## 需要插件

安装 LINE 插件：

```bash
moltbot plugins install @moltbot/line
```

本地检出（从 git repo 运行时）：

```bash
moltbot plugins install ./extensions/line
```

## 设置

1) 创建一个 LINE Developers 帐户并打开控制台：
   https://developers.line.biz/console/
2) 创建（或选择）一个 Provider 并添加一个 **Messaging API** 频道。
3) 从频道设置中复制 **Channel access token** 和 **Channel secret**。
4) 在 Messaging API 设置中启用 **Use webhook**。
5) 将 webhook URL 设置为您的网关端点（需要 HTTPS）：

```
https://gateway-host/line/webhook
```

网关响应 LINE 的 webhook 验证（GET）和入站事件（POST）。
如果您需要自定义路径，请设置 `channels.line.webhookPath` 或
`channels.line.accounts.<id>.webhookPath` 并相应地更新 URL。

## 配置

最小配置：

```json5
{
  channels: {
    line: {
      enabled: true,
      channelAccessToken: "LINE_CHANNEL_ACCESS_TOKEN",
      channelSecret: "LINE_CHANNEL_SECRET",
      dmPolicy: "pairing"
    }
  }
}
```

环境变量（仅默认帐户）：

- `LINE_CHANNEL_ACCESS_TOKEN`
- `LINE_CHANNEL_SECRET`

令牌/密钥文件：

```json5
{
  channels: {
    line: {
      tokenFile: "/path/to/line-token.txt",
      secretFile: "/path/to/line-secret.txt"
    }
  }
}
```

多个帐户：

```json5
{
  channels: {
    line: {
      accounts: {
        marketing: {
          channelAccessToken: "...",
          channelSecret: "...",
          webhookPath: "/line/marketing"
        }
      }
    }
  }
}
```

## 访问控制

直接消息默认为配对。未知发送者获得配对码，其消息在批准之前被忽略。

```bash
moltbot pairing list line
moltbot pairing approve line <CODE>
```

白名单和策略：

- `channels.line.dmPolicy`：`pairing | allowlist | open | disabled`
- `channels.line.allowFrom`：DM 的白名单 LINE 用户 ID
- `channels.line.groupPolicy`：`allowlist | open | disabled`
- `channels.line.groupAllowFrom`：群组的白名单 LINE 用户 ID
- 每个群组覆盖：`channels.line.groups.<groupId>.allowFrom`

LINE ID 区分大小写。有效 ID 如下所示：

- 用户：`U` + 32 个十六进制字符
- 群组：`C` + 32 个十六进制字符
- 房间：`R` + 32 个十六进制字符

## 消息行为

- 文本在 5000 个字符处分块。
- Markdown 格式被剥离；代码块和表格在可能的情况下转换为 Flex 卡片。
- 流式响应被缓冲；LINE 在 agent 工作时接收带有加载动画的完整块。
- 媒体下载受 `channels.line.mediaMaxMb` 限制（默认 10）。

## 频道数据（富消息）

使用 `channelData.line` 发送快速回复、位置、Flex 卡片或模板消息。

```json5
{
  text: "给您",
  channelData: {
    line: {
      quickReplies: ["状态", "帮助"],
      location: {
        title: "办公室",
        address: "123 Main St",
        latitude: 35.681236,
        longitude: 139.767125
      },
      flexMessage: {
        altText: "状态卡片",
        contents: { /* Flex payload */ }
      },
      templateMessage: {
        type: "confirm",
        text: "继续？",
        confirmLabel: "是",
        confirmData: "yes",
        cancelLabel: "否",
        cancelData: "no"
      }
    }
  }
}
```

LINE 插件还提供用于 Flex 消息预设的 `/card` 命令：

```
/card info "欢迎" "感谢加入！"
```

## 故障排除

- **Webhook 验证失败**：确保 webhook URL 是 HTTPS 并且 `channelSecret` 与 LINE 控制台匹配。
- **没有入站事件**：确认 webhook 路径与 `channels.line.webhookPath` 匹配，并且网关可从 LINE 访问。
- **媒体下载错误**：如果媒体超过默认限制，请提高 `channels.line.mediaMaxMb`。
