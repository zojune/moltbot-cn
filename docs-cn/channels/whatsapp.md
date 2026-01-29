---
summary: "WhatsApp（web 通道）集成：登录、收件箱、回复、媒体和运维"
read_when:
  - 开发 WhatsApp/web 通道行为或收件箱路由时
---
# WhatsApp (web channel)


状态：仅支持通过 Baileys 的 WhatsApp Web。Gateway 拥有会话。

## 快速设置（初学者）
1) 如果可能，使用**单独的电话号码**（推荐）。
2) 在 `~/.clawdbot/moltbot.json` 中配置 WhatsApp。
3) 运行 `moltbot channels login` 扫描二维码（关联设备）。
4) 启动 gateway。

最小配置：
```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

## 目标
- 在一个 Gateway 进程中支持多个 WhatsApp 账户（多账户）。
- 确定性路由：回复返回到 WhatsApp，没有模型路由。
- 模型看到足够的上下文来理解引用回复。

## 配置写入
默认情况下，WhatsApp 允许写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。

禁用：
```json5
{
  channels: { whatsapp: { configWrites: false } }
}
```

## 架构（谁拥有什么）
- **Gateway** 拥有 Baileys socket 和收件箱循环。
- **CLI / macOS 应用**与 gateway 对话；不直接使用 Baileys。
- **主动监听器**是出站发送所必需的；否则发送快速失败。

## 获取电话号码（两种模式）

WhatsApp 需要真实的移动电话号码进行验证。VoIP 和虚拟号码通常被阻止。有两种支持的方式在 WhatsApp 上运行 Moltbot：

### 专用号码（推荐）
为 Moltbot 使用**单独的电话号码**。最佳用户体验，清晰的路由，没有自聊怪癖。理想设置：**备用/旧 Android 手机 + eSIM**。将其保持在 Wi‑Fi 和电源上，并通过二维码链接。

**WhatsApp Business：** 你可以在同一设备上使用不同号码的 WhatsApp Business。非常适合将你的个人 WhatsApp 分开 — 安装 WhatsApp Business 并在那里注册 Moltbot 号码。

**示例配置（专用号码，单用户许可列表）：**
```json5
{
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551234567"]
    }
  }
}
```

**配对模式（可选）：**
如果你想要配对而不是许可列表，将 `channels.whatsapp.dmPolicy` 设置为 `pairing`。未知发送者获得配对码；使用以下命令批准：
`moltbot pairing approve whatsapp <code>`

### 个人号码（回退）
快速回退：在**你自己的号码**上运行 Moltbot。向自己发送消息（WhatsApp "Message yourself"）进行测试，这样你就不会骚扰联系人。在设置和实验期间，预计在主手机上读取验证码。**必须启用自聊模式。**
当向导询问你的个人 WhatsApp 号码时，输入你将从中发送消息的电话（所有者/发送者），而不是助手号码。

**示例配置（个人号码，自聊）：**
```json
{
  "whatsapp": {
    "selfChatMode": true,
    "dmPolicy": "allowlist",
    "allowFrom": ["+15551234567"]
  }
}
```

如果设置了 `messages.responsePrefix`，自聊回复默认为 `[{identity.name}]`（否则 `[moltbot]`）。
如果未设置，显式设置它以自定义或禁用前缀（使用 `""` 删除它）。

### 号码来源提示
- **本地 eSIM** 来自你所在国家的移动运营商（最可靠）
  - 奥地利：[hot.at](https://www.hot.at)
  - 英国：[giffgaff](https://www.giffgaff.com) — 免费 SIM，无合同
- **预付费 SIM** — 便宜，只需要接收一条短信进行验证

**避免：** TextNow、Google Voice、大多数"免费短信"服务 — WhatsApp 会积极阻止这些。

**提示：** 号码只需要接收一条验证短信。之后，WhatsApp Web 会话通过 `creds.json` 持久化。

## 为什么不使用 Twilio？
- 早期的 Moltbot 构建支持 Twilio 的 WhatsApp Business 集成。
- WhatsApp Business 号码不适合个人助手。
- Meta 强制执行 24 小时回复窗口；如果你在过去 24 小时内没有回复，企业号码无法发起新消息。
- 高容量或"健谈"的使用会触发积极的阻止，因为企业账户不适合发送数十条个人助手消息。
- 结果：传递不可靠和频繁阻止，因此支持被移除。

## 登录 + 凭据
- 登录命令：`moltbot channels login`（通过关联设备的 QR）。
- 多账户登录：`moltbot channels login --account <id>`（`<id>` = `accountId`）。
- 默认账户（省略 `--account` 时）：如果存在则为 `default`，否则为第一个配置的账户 ID（排序）。
- 凭据存储在 `~/.clawdbot/credentials/whatsapp/<accountId>/creds.json` 中。
- 备份副本在 `creds.json.bak`（损坏时恢复）。
- 遗留兼容性：旧版本安装直接在 `~/.clawdbot/credentials/` 中存储 Baileys 文件。
- 注销：`moltbot channels logout`（或 `--account <id>`）删除 WhatsApp 身份验证状态（但保留共享的 `oauth.json`）。
- 注销的 socket => 指示重新链接的错误。

## 入站流程（私信 + 群组）
- WhatsApp 事件来自 `messages.upsert`（Baileys）。
- 收件箱监听器在关闭时分离，以避免在测试/重启中累积事件处理程序。
- 状态/广播聊天被忽略。
- 直接聊天使用 E.164；群组使用群组 JID。
- **私信策略**：`channels.whatsapp.dmPolicy` 控制直接聊天访问（默认：`pairing`）。
  - 配对：未知发送者获得配对码（通过 `moltbot pairing approve whatsapp <code>` 批准；配对码在 1 小时后过期）。
  - 开放：需要 `channels.whatsapp.allowFrom` 包含 `"*"`。
  - 自己的消息始终允许；"自聊模式"仍需要 `channels.whatsapp.allowFrom` 包含你自己的号码。

### 个人号码模式（回退）
如果你在**个人 WhatsApp 号码**上运行 Moltbot，启用 `channels.whatsapp.selfChatMode`（参见上面的示例）。

行为：
- 出站私信从不触发配对回复（防止骚扰联系人）。
- 入站未知发送者仍遵循 `channels.whatsapp.dmPolicy`。
- 自聊模式（allowFrom 包含你的号码）避免自动已读回执并忽略提及 JID。
- 为非自聊私信发送已读回执。

## 已读回执
默认情况下，gateway 在接受入站 WhatsApp 消息后将其标记为已读（蓝勾）。

全局禁用：
```json5
{
  channels: { whatsapp: { sendReadReceipts: false } }
}
```

每个账户禁用：
```json5
{
  channels: {
    whatsapp: {
      accounts: {
        personal: { sendReadReceipts: false }
      }
    }
  }
}
```

注意事项：
- 自聊模式始终跳过已读回执。

## WhatsApp 常见问题解答：发送消息 + 配对

**当我链接 WhatsApp 时，Moltbot 会给随机联系人发消息吗？**
不会。默认私信策略是**配对**，因此未知发送者只获得配对码，其消息**不会被处理**。Moltbot 只回复它接收到的聊天，或你明确触发的发送（agent/CLI）。

**WhatsApp 上的配对如何工作？**
配对是未知发送者的私信门槛：
- 来自新发送者的第一条私信返回一个短代码（消息不被处理）。
- 使用以下命令批准：`moltbot pairing approve whatsapp <code>`（使用 `moltbot pairing list whatsapp` 列出）。
- 配对码在 1 小时后过期；每个通道最多有 3 个待处理的请求。

**多个人可以在一个 WhatsApp 号码上使用不同的 Moltbot 吗？**
可以，通过 `bindings` 将每个发送者路由到不同的 agent（peer `kind: "dm"`，发送者 E.164 如 `+15551234567`）。回复仍来自**相同的 WhatsApp 账户**，直接聊天折叠到每个 agent 的主会话，因此**每人使用一个 agent**。私信访问控制（`dmPolicy`/`allowFrom`）是每个 WhatsApp 账户的全局控制。参见 [Multi-Agent Routing](/concepts/multi-agent)。

**向导为什么要问我的电话号码？**
向导使用它来设置你的**许可列表/所有者**，以便允许你自己的私信。它不用于自动发送。如果你在自己的个人 WhatsApp 号码上运行，请使用相同的号码并启用 `channels.whatsapp.selfChatMode`。

## 消息规范化（模型看到的内容）
- `Body` 是带有信封的当前消息正文。
- 引用回复上下文**始终附加**：
  ```
  [Replying to +1555 id:ABC123]
  <quoted text or <media:...>>
  [/Replying]
  ```
- 回复元数据也设置：
  - `ReplyToId` = stanzaId
  - `ReplyToBody` = 引用的正文或媒体占位符
  - `ReplyToSender` = 已知时为 E.164
- 仅媒体入站消息使用占位符：
  - `<media:image|video|audio|document|sticker>`

## 群组
- 群组映射到 `agent:<agentId>:whatsapp:group:<jid>` 会话。
- 群组策略：`channels.whatsapp.groupPolicy = open|disabled|allowlist`（默认 `allowlist`）。
- 激活模式：
  - `mention`（默认）：需要 @提及或正则匹配。
  - `always`：始终触发。
- `/activation mention|always` 仅限所有者，必须作为单独消息发送。
- 所有者 = `channels.whatsapp.allowFrom`（如果未设置，则为自己 E.164）。
- **历史注入**（仅待处理）：
  - 最近的*未处理*消息（默认 50）插入在：
    `[Chat messages since your last reply - for context]` 下（会话中已存在的消息不会被重新注入）
  - 当前消息在：
    `[Current message - respond to this]` 下
  - 发送者后缀附加：`[from: Name (+E164)]`
- 群组元数据缓存 5 分钟（主题 + 参与者）。

## 回复传递（线程化）
- WhatsApp Web 发送标准消息（当前 gateway 中没有引用回复线程化）。
- 此通道忽略回复标签。

## 确认反应（接收时自动反应）

WhatsApp 可以在接收入站消息时立即自动发送表情符号反应，在 bot 生成回复之前。这为用户提供了即时反馈，表明他们的消息已被接收。

**配置：**
```json
{
  "whatsapp": {
    "ackReaction": {
      "emoji": "👀",
      "direct": true,
      "group": "mentions"
    }
  }
}
```

**选项：**
- `emoji`（字符串）：用于确认的表情符号（例如，"👀"、"✅"、"📨"）。空或省略 = 功能已禁用。
- `direct`（布尔值，默认：`true`）：在直接/私信聊天中发送反应。
- `group`（字符串，默认：`"mentions"`）：群组聊天行为：
  - `"always"`：对所有群组消息做出反应（即使没有 @提及）
  - `"mentions"`：仅在 bot 被 @提及时做出反应
  - `"never"`：从不在群组中做出反应

**每个账户的覆盖：**
```json
{
  "whatsapp": {
    "accounts": {
      "work": {
        "ackReaction": {
          "emoji": "✅",
          "direct": false,
          "group": "always"
        }
      }
    }
  }
}
```

**行为注意事项：**
- 反应在消息接收时**立即**发送，在输入指示器或 bot 回复之前。
- 在具有 `requireMention: false`（激活：always）的群组中，`group: "mentions"` 将对所有消息做出反应（不仅仅是 @提及）。
- 即发即忘：反应失败被记录，但不会阻止 bot 回复。
- 参与者 JID 自动包含在群组反应中。
- WhatsApp 忽略 `messages.ackReaction`；使用 `channels.whatsapp.ackReaction` 代替。

## Agent 工具（反应）
- 工具：`whatsapp` 使用 `react` 操作（`chatJid`、`messageId`、`emoji`，可选 `remove`）。
- 可选：`participant`（群组发送者）、`fromMe`（对你自己的消息做出反应）、`accountId`（多账户）。
- 反应移除语义：参见 [/tools/reactions](/tools/reactions)。
- 工具限制：`channels.whatsapp.actions.reactions`（默认：启用）。

## 限制
- 出站文本被分块为 `channels.whatsapp.textChunkLimit`（默认 4000）。
- 可选换行分块：设置 `channels.whatsapp.chunkMode="newline"` 在长度分块之前按空行（段落边界）分割。
- 入站媒体保存受 `channels.whatsapp.mediaMaxMb` 限制（默认 50 MB）。
- 出站媒体项受 `agents.defaults.mediaMaxMb` 限制（默认 5 MB）。

## 出站发送（文本 + 媒体）
- 使用主动 web 监听器；如果 gateway 未运行则错误。
- 文本分块：每条消息最多 4k（可通过 `channels.whatsapp.textChunkLimit` 配置，可选 `channels.whatsapp.chunkMode`）。
- 媒体：
  - 支持图像/视频/音频/文档。
  - 音频作为 PTT 发送；`audio/ogg` => `audio/ogg; codecs=opus`。
  - 标题仅在第一个媒体项上。
  - 媒体获取支持 HTTP(S) 和本地路径。
  - 动画 GIF：WhatsApp 期望 MP4 并带有 `gifPlayback: true` 用于内联循环。
    - CLI：`moltbot message send --media <mp4> --gif-playback`
    - Gateway：`send` 参数包括 `gifPlayback: true`

## 语音消息（PTT 音频）
WhatsApp 将音频作为**语音消息**（PTT 气泡）发送。
- 最佳效果：OGG/Opus。Moltbot 将 `audio/ogg` 重写为 `audio/ogg; codecs=opus`。
- 对于 WhatsApp，`[[audio_as_voice]]` 被忽略（音频已作为语音消息发送）。

## 媒体限制 + 优化
- 默认出站上限：5 MB（每个媒体项）。
- 覆盖：`agents.defaults.mediaMaxMb`。
- 图像在限制下自动优化为 JPEG（调整大小 + 质量扫描）。
- 超大媒体 => 错误；媒体回复回退到文本警告。

## 心跳
- **Gateway 心跳**记录连接健康（`web.heartbeatSeconds`，默认 60s）。
- **Agent 心跳**可以每个 agent 配置（`agents.list[].heartbeat`）或全局配置
  通过 `agents.defaults.heartbeat`（当未设置每个 agent 条目时回退）。
  - 使用配置的心跳提示（默认：`Read HEARTBEAT.md if it exists (workspace context). Follow it strictly. Do not infer or repeat old tasks from prior chats. If nothing needs attention, reply HEARTBEAT_OK.`）+ `HEARTBEAT_OK` 跳过行为。
  - 传递默认为最后使用的通道（或配置的目标）。

## 重连行为
- 退避策略：`web.reconnect`：
  - `initialMs`、`maxMs`、`factor`、`jitter`、`maxAttempts`。
- 如果达到 maxAttempts，web 监控停止（降级）。
- 注销 => 停止并要求重新链接。

## 配置快速映射
- `channels.whatsapp.dmPolicy`（私信策略：pairing/allowlist/open/disabled）。
- `channels.whatsapp.selfChatMode`（同手机设置；bot 使用你的个人 WhatsApp 号码）。
- `channels.whatsapp.allowFrom`（私信许可列表）。WhatsApp 使用 E.164 电话号码（没有用户名）。
- `channels.whatsapp.mediaMaxMb`（入站媒体保存上限）。
- `channels.whatsapp.ackReaction`（消息接收时的自动反应：`{emoji, direct, group}`）。
- `channels.whatsapp.accounts.<accountId>.*`（每个账户的设置 + 可选的 `authDir`）。
- `channels.whatsapp.accounts.<accountId>.mediaMaxMb`（每个账户的入站媒体上限）。
- `channels.whatsapp.accounts.<accountId>.ackReaction`（每个账户的确认反应覆盖）。
- `channels.whatsapp.groupAllowFrom`（群组发送者许可列表）。
- `channels.whatsapp.groupPolicy`（群组策略）。
- `channels.whatsapp.historyLimit` / `channels.whatsapp.accounts.<accountId>.historyLimit`（群组历史上下文；`0` 禁用）。
- `channels.whatsapp.dmHistoryLimit`（私信历史限制，用户轮次）。每个用户覆盖：`channels.whatsapp.dms["<phone>"].historyLimit`。
- `channels.whatsapp.groups`（群组许可列表 + 提及门槛默认值；使用 `"*"` 允许所有）
- `channels.whatsapp.actions.reactions`（限制 WhatsApp 工具反应）。
- `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）
- `messages.groupChat.historyLimit`
- `channels.whatsapp.messagePrefix`（入站前缀；每个账户：`channels.whatsapp.accounts.<accountId>.messagePrefix`；已弃用：`messages.messagePrefix`）
- `messages.responsePrefix`（出站前缀）
- `agents.defaults.mediaMaxMb`
- `agents.defaults.heartbeat.every`
- `agents.defaults.heartbeat.model`（可选覆盖）
- `agents.defaults.heartbeat.target`
- `agents.defaults.heartbeat.to`
- `agents.defaults.heartbeat.session`
- `agents.list[].heartbeat.*`（每个账户覆盖）
- `session.*`（scope、idle、store、mainKey）
- `web.enabled`（当为 false 时禁用通道启动）
- `web.heartbeatSeconds`
- `web.reconnect.*`

## 日志 + 故障排除
- 子系统：`whatsapp/inbound`、`whatsapp/outbound`、`web-heartbeat`、`web-reconnect`。
- 日志文件：`/tmp/moltbot/moltbot-YYYY-MM-DD.log`（可配置）。
- 故障排除指南：[Gateway troubleshooting](/gateway/troubleshooting)。

## 故障排除（快速）

**未链接 / 需要二维码登录**
- 症状：`channels status` 显示 `linked: false` 或警告"Not linked"。
- 修复：在 gateway 主机上运行 `moltbot channels login` 并扫描二维码（WhatsApp → 设置 → 关联设备）。

**已链接但断开连接 / 重连循环**
- 症状：`channels status` 显示 `running, disconnected` 或警告"Linked but disconnected"。
- 修复：`moltbot doctor`（或重启 gateway）。如果持续存在，通过 `channels login` 重新链接并检查 `moltbot logs --follow`。

**Bun 运行时**
- **不推荐**使用 Bun。WhatsApp (Baileys) 和 Telegram 在 Bun 上不可靠。
  使用 **Node** 运行 gateway。（参见入门运行时说明）。
