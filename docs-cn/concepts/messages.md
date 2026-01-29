---
summary: "消息流、会话、排队和推理可见性"
read_when:
  - 解释入站消息如何成为回复
  - 阐明会话、排队模式或流式传输行为
  - 记录推理可见性和使用影响
---
# 消息

此页面将 Moltbot 如何处理入站消息、会话、排队、流式传输和推理可见性联系在一起。

## 消息流（高级）

```
入站消息
  -> 路由/绑定 -> 会话键
  -> 队列（如果运行处于活动状态）
  -> 代理运行（流式传输 + 工具）
  -> 出站回复（通道限制 + 分块）
```

关键旋钮位于配置中：
- `messages.*` 用于前缀、排队和群组行为。
- `agents.defaults.*` 用于块流式传输和分块默认值。
- 通道覆盖（`channels.whatsapp.*`、`channels.telegram.*` 等）用于上限和流式传输开关。

有关完整架构，请参见[配置](/gateway/configuration)。

## 入站去重

通道可以在重新连接后重新传递相同的消息。Moltbot 保留一个短期缓存，键为通道/账户/对等/会话/消息 ID，因此重复传递不会触发另一个代理运行。

## 入站防抖

来自**同一发送者**的快速连续消息可以通过 `messages.inbound` 批处理到单个代理轮次中。防抖范围为每个通道 + 对话，并使用最新消息进行回复线程/ID。

配置（全局默认值 + 每个通道覆盖）：
```json5
{
  messages: {
    inbound: {
      debounceMs: 2000,
      byChannel: {
        whatsapp: 5000,
        slack: 1500,
        discord: 1500
      }
    }
  }
}
```

注意事项：
- 防抖仅适用于**纯文本**消息；媒体/附件立即刷新。
- 控制命令绕过防抖，因此它们保持独立。

## 会话和设备

会话由网关拥有，而不是由客户端拥有。
- 直接聊天折叠到代理主会话键。
- 群组/频道获得自己的会话键。
- 会话存储和记录存在于网关主机上。

多个设备/通道可以映射到同一个会话，但历史记录不会完全同步回每个客户端。建议：使用一个主设备进行长对话以避免上下文分歧。Control UI 和 TUI 始终显示网关支持的会话记录，因此它们是真实来源。

详情：[会话管理](/concepts/session)。

## 入站主体和历史上下文

Moltbot 将**提示主体**与**命令主体**分开：
- `Body`：发送到代理的提示文本。这可能包括通道信封和可选的历史包装器。
- `CommandBody`：用于指令/命令解析的原始用户文本。
- `RawBody`：`CommandBody` 的遗留别名（为了兼容性而保留）。

当通道提供历史时，它使用共享包装器：
- `[Chat messages since your last reply - for context]`
- `[Current message - respond to this]`

对于**非直接聊天**（群组/频道/房间），**当前消息主体**以发送者标签为前缀（与历史条目使用的样式相同）。这将实时和排队/历史消息保持在代理提示中一致。

历史缓冲区是**仅限待处理**：它们包括*未*触发的群组消息（例如，提及门控的消息）并**排除**已在会话记录中的消息。

指令剥离仅适用于**当前消息**部分，因此历史保持完整。包装历史的通道应将 `CommandBody`（或 `RawBody`）设置为原始消息文本，并将 `Body` 保留为组合提示。历史缓冲区可通过 `messages.groupChat.historyLimit`（全局默认值）和每个通道的覆盖（如 `channels.slack.historyLimit` 或 `channels.telegram.accounts.<id>.historyLimit`）配置（设置为 `0` 以禁用）。

## 排队和后续

如果运行已经处于活动状态，入站消息可以排队、引导到当前运行或为后续轮次收集。

- 通过 `messages.queue`（和 `messages.queue.byChannel`）配置。
- 模式：`interrupt`、`steer`、`followup`、`collect`，加上积压变体。

详情：[排队](/concepts/queue)。

## 流式传输、分块和批处理

块流式传输在模型生成文本块时发送部分回复。分块尊重通道文本限制并避免分割围栏代码。

关键设置：
- `agents.defaults.blockStreamingDefault`（`on|off`，默认关闭）
- `agents.defaults.blockStreamingBreak`（`text_end|message_end`）
- `agents.defaults.blockStreamingChunk`（`minChars|maxChars|breakPreference`）
- `agents.defaults.blockStreamingCoalesce`（基于空闲的批处理）
- `agents.defaults.humanDelay`（块回复之间类似人类的暂停）
- 通道覆盖：`*.blockStreaming` 和 `*.blockStreamingCoalesce`（非 Telegram 通道需要显式 `*.blockStreaming: true`）

详情：[流式传输 + 分块](/concepts/streaming)。

## 推理可见性和令牌

Moltbot 可以暴露或隐藏模型推理：
- `/reasoning on|off|stream` 控制可见性。
- 推理内容仍然计入令牌使用（由模型生成时）。
- Telegram 支持推理流式传输到草稿气泡。

详情：[思考 + 推理指令](/tools/thinking)和[令牌使用](/token-use)。

## 前缀、线程和回复

出站消息格式在 `messages` 中集中：
- `messages.responsePrefix`（出站前缀）和 `channels.whatsapp.messagePrefix`（WhatsApp 入站前缀）
- 通过 `replyToMode` 和每个通道的默认值进行回复线程

详情：[配置](/gateway/configuration#messages)和通道文档。
