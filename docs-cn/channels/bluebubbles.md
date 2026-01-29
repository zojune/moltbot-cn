---
summary: "通过 BlueBubbles macOS 服务器实现 iMessage（REST 发送/接收、输入状态、回应、配对、高级操作）"
read_when:
  - 设置 BlueBubbles 渠道
  - 排查 webhook 配对故障
  - 在 macOS 上配置 iMessage
---
# BlueBubbles (macOS REST)

状态：内置插件，通过 HTTP 与 BlueBubbles macOS 服务器通信。由于其 API 更丰富且设置更简单，相比旧的 imsg 渠道，**推荐用于 iMessage 集成**。

## 概述
- 通过 macOS 上的 BlueBubbles 辅助应用程序运行（[bluebubbles.app](https://bluebubbles.app)）。
- 推荐/测试版本：macOS Sequoia (15)。macOS Tahoe (26) 也可用；但在 Tahoe 上编辑功能目前不可用，群组图标更新可能报告成功但不同步。
- Moltbot 通过其 REST API 与之通信（`GET /api/v1/ping`、`POST /message/text`、`POST /chat/:id/*`）。
- 入站消息通过 webhook 到达；出站回复、输入指示器、已读回执和轻点反馈都是 REST 调用。
- 附件和贴纸作为入站媒体被摄取（并在可能时提供给 agent）。
- 配对/白名单的工作方式与其他渠道相同（`/start/pairing` 等），使用 `channels.bluebubbles.allowFrom` + 配对码。
- 回应作为系统事件显示，就像 Slack/Telegram 一样，以便 agent 可以在回复前"提及"它们。
- 高级功能：编辑、撤回、回复线程、消息效果、群组管理。

## 快速开始
1. 在 Mac 上安装 BlueBubbles 服务器（按照 [bluebubbles.app/install](https://bluebubbles.app/install) 上的说明进行操作）。
2. 在 BlueBubbles 配置中，启用 Web API 并设置密码。
3. 运行 `moltbot onboard` 并选择 BlueBubbles，或手动配置：
   ```json5
   {
     channels: {
       bluebubbles: {
         enabled: true,
         serverUrl: "http://192.168.1.100:1234",
         password: "example-password",
         webhookPath: "/bluebubbles-webhook"
       }
     }
   }
   ```
4. 将 BlueBubbles webhook 指向您的网关（例如：`https://your-gateway-host:3000/bluebubbles-webhook?password=<password>`）。
5. 启动网关；它将注册 webhook 处理程序并开始配对。

## 入门向导
BlueBubbles 可在交互式设置向导中使用：
```
moltbot onboard
```

向导会提示：
- **服务器 URL**（必需）：BlueBubbles 服务器地址（例如，`http://192.168.1.100:1234`）
- **密码**（必需）：来自 BlueBubbles 服务器设置的 API 密码
- **Webhook 路径**（可选）：默认为 `/bluebubbles-webhook`
- **DM 策略**：配对、白名单、开放或禁用
- **允许列表**：电话号码、电子邮件或聊天目标

您也可以通过 CLI 添加 BlueBubbles：
```
moltbot channels add bluebubbles --http-url http://192.168.1.100:1234 --password <password>
```

## 访问控制（DM + 群组）
私聊：
- 默认：`channels.bluebubbles.dmPolicy = "pairing"`。
- 未知发送者会收到配对码；在批准之前消息将被忽略（配对码在 1 小时后过期）。
- 通过以下方式批准：
  - `moltbot pairing list bluebubbles`
  - `moltbot pairing approve bluebubbles <CODE>`
- 配对是默认的令牌交换方式。详情：[配对](/start/pairing)

群组：
- `channels.bluebubbles.groupPolicy = open | allowlist | disabled`（默认：`allowlist`）。
- `channels.bluebubbles.groupAllowFrom` 控制在设置为 `allowlist` 时谁可以在群组中触发。

### 提及门控（群组）
BlueBubbles 支持群聊的提及门控，匹配 iMessage/WhatsApp 的行为：
- 使用 `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）来检测提及。
- 当为群组启用 `requireMention` 时，agent 仅在被提及时才响应。
- 来自授权发送者的控制命令绕过提及门控。

每个群组配置：
```json5
{
  channels: {
    bluebubbles: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true },  // 所有群组的默认设置
        "iMessage;-;chat123": { requireMention: false }  // 特定群组的覆盖
      }
    }
  }
}
```

### 命令门控
- 控制命令（例如，`/config`、`/model`）需要授权。
- 使用 `allowFrom` 和 `groupAllowFrom` 来确定命令授权。
- 授权的发送者即使在群组中未提及也可以运行控制命令。

## 输入状态 + 已读回执
- **输入指示器**：在响应生成之前和期间自动发送。
- **已读回执**：由 `channels.bluebubbles.sendReadReceipts` 控制（默认：`true`）。
- **输入指示器**：Moltbot 发送输入开始事件；BlueBubbles 在发送或超时时自动清除输入状态（通过 DELETE 手动停止不可靠）。

```json5
{
  channels: {
    bluebubbles: {
      sendReadReceipts: false  // 禁用已读回执
    }
  }
}
```

## 高级操作
BlueBubbles 支持在配置中启用时的高级消息操作：

```json5
{
  channels: {
    bluebubbles: {
      actions: {
        reactions: true,       // 轻点反馈（默认：true）
        edit: true,            // 编辑已发送的消息（macOS 13+，在 macOS 26 Tahoe 上损坏）
        unsend: true,          // 撤回消息（macOS 13+）
        reply: true,           // 通过消息 GUID 回复线程
        sendWithEffect: true,  // 消息效果（slam、loud 等）
        renameGroup: true,     // 重命名群聊
        setGroupIcon: true,    // 设置群聊图标/照片（在 macOS 26 Tahoe 上不稳定）
        addParticipant: true,  // 向群组添加参与者
        removeParticipant: true, // 从群组中移除参与者
        leaveGroup: true,      // 离开群聊
        sendAttachment: true   // 发送附件/媒体
      }
    }
  }
}
```

可用操作：
- **react**：添加/移除轻点反馈（`messageId`、`emoji`、`remove`）
- **edit**：编辑已发送的消息（`messageId`、`text`）
- **unsend**：撤回消息（`messageId`）
- **reply**：回复特定消息（`messageId`、`text`、`to`）
- **sendWithEffect**：发送带有 iMessage 效果的消息（`text`、`to`、`effectId`）
- **renameGroup**：重命名群聊（`chatGuid`、`displayName`）
- **setGroupIcon**：设置群聊的图标/照片（`chatGuid`、`media`）— 在 macOS 26 Tahoe 上不稳定（API 可能返回成功但图标不同步）
- **addParticipant**：向群组添加某人（`chatGuid`、`address`）
- **removeParticipant**：从群组中移除某人（`chatGuid`、`address`）
- **leaveGroup**：离开群聊（`chatGuid`）
- **sendAttachment**：发送媒体/文件（`to`、`buffer`、`filename`、`asVoice`）
  - 语音备忘录：将 `asVoice: true` 与 **MP3** 或 **CAF** 音频一起设置，以作为 iMessage 语音消息发送。BlueBubbles 在发送语音备忘录时将 MP3 转换为 CAF。

### 消息 ID（短 ID 与完整 ID）
为了节省令牌，Moltbot 可能会显示*短*消息 ID（例如，`1`、`2`）。
- `MessageSid` / `ReplyToId` 可以是短 ID。
- `MessageSidFull` / `ReplyToIdFull` 包含提供者的完整 ID。
- 短 ID 存储在内存中；它们可能在重启或缓存逐出时过期。
- 操作接受短 ID 或完整 `messageId`，但如果短 ID 不可用则会出错。

对于持久化的自动化和存储，请使用完整 ID：
- 模板：`{{MessageSidFull}}`、`{{ReplyToIdFull}}`
- 上下文：入站负载中的 `MessageSidFull` / `ReplyToIdFull`

请参阅 [配置](/gateway/configuration) 以了解模板变量。

## 阻止流式传输
控制响应是作为单个消息发送还是以块的形式流式传输：
```json5
{
  channels: {
    bluebubbles: {
      blockStreaming: true  // 启用块流式传输（默认行为）
    }
  }
}
```

## 媒体 + 限制
- 入站附件被下载并存储在媒体缓存中。
- 通过 `channels.bluebubbles.mediaMaxMb` 设置媒体上限（默认：8 MB）。
- 出站文本被分块为 `channels.bluebubbles.textChunkLimit`（默认：4000 个字符）。

## 配置参考
完整配置：[配置](/gateway/configuration)

提供者选项：
- `channels.bluebubbles.enabled`：启用/禁用渠道。
- `channels.bluebubbles.serverUrl`：BlueBubbles REST API 基础 URL。
- `channels.bluebubbles.password`：API 密码。
- `channels.bluebubbles.webhookPath`：Webhook 端点路径（默认：`/bluebubbles-webhook`）。
- `channels.bluebubbles.dmPolicy`：`pairing | allowlist | open | disabled`（默认：`pairing`）。
- `channels.bluebubbles.allowFrom`：DM 白名单（句柄、电子邮件、E.164 号码、`chat_id:*`、`chat_guid:*`）。
- `channels.bluebubbles.groupPolicy`：`open | allowlist | disabled`（默认：`allowlist`）。
- `channels.bluebubbles.groupAllowFrom`：群组发送者白名单。
- `channels.bluebubbles.groups`：每个群组配置（`requireMention` 等）。
- `channels.bluebubbles.sendReadReceipts`：发送已读回执（默认：`true`）。
- `channels.bluebubbles.blockStreaming`：启用块流式传输（默认：`true`）。
- `channels.bluebubbles.textChunkLimit`：出站分块大小（字符）（默认：4000）。
- `channels.bluebubbles.chunkMode`：`length`（默认）仅在超过 `textChunkLimit` 时拆分；`newline` 在长度分块之前在空行（段落边界）处拆分。
- `channels.bluebubbles.mediaMaxMb`：入站媒体上限（MB）（默认：8）。
- `channels.bluebubbles.historyLimit`：上下文的最大群组消息数（0 表示禁用）。
- `channels.bluebubbles.dmHistoryLimit`：DM 历史记录限制。
- `channels.bluebubbles.actions`：启用/禁用特定操作。
- `channels.bluebubbles.accounts`：多账户配置。

相关全局选项：
- `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）。
- `messages.responsePrefix`。

## 寻址 / 投递目标
首选 `chat_guid` 以进行稳定路由：
- `chat_guid:iMessage;-;+15555550123`（群组推荐）
- `chat_id:123`
- `chat_identifier:...`
- 直接句柄：`+15555550123`、`user@example.com`
  - 如果直接句柄没有现有的 DM 聊天，Moltbot 将通过 `POST /api/v1/chat/new` 创建一个。这需要启用 BlueBubbles Private API。

## 安全性
- Webhook 请求通过将 `guid`/`password` 查询参数或标头与 `channels.bluebubbles.password` 进行比较来验证。来自 `localhost` 的请求也被接受。
- 保持 API 密码和 webhook 端点保密（将它们视为凭据）。
- Localhost 信任意味着同主机反向代理可能会无意中绕过密码。如果您代理网关，请在代理处要求身份验证并配置 `gateway.trustedProxies`。请参阅 [网关安全性](/gateway/security#reverse-proxy-configuration)。
- 如果将 BlueBubbles 服务器暴露在 LAN 之外，请在 BlueBubbles 服务器上启用 HTTPS + 防火墙规则。

## 故障排除
- 如果输入/已读事件停止工作，请检查 BlueBubbles webhook 日志并验证网关路径与 `channels.bluebubbles.webhookPath` 匹配。
- 配对码在一小时后过期；使用 `moltbot pairing list bluebubbles` 和 `moltbot pairing approve bluebubbles <code>`。
- 回应需要 BlueBubbles private API（`POST /api/v1/message/react`）；确保服务器版本公开了它。
- 编辑/撤回需要 macOS 13+ 和兼容的 BlueBubbles 服务器版本。在 macOS 26 (Tahoe) 上，由于 private API 更改，编辑目前损坏。
- 在 macOS 26 (Tahoe) 上，群组图标更新可能不稳定：API 可能返回成功但新图标不同步。
- Moltbot 根据 BlueBubbles 服务器的 macOS 版本自动隐藏已知损坏的操作。如果在 macOS 26 (Tahoe) 上仍然显示编辑，请使用 `channels.bluebubbles.actions.edit=false` 手动禁用它。
- 有关状态/健康状况信息：`moltbot status --all` 或 `moltbot status --deep`。

有关一般渠道工作流程参考，请参阅 [渠道](/channels) 和 [插件](/plugins) 指南。
