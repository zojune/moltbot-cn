---
summary: "通过 grammY 的 Telegram Bot API 集成及设置说明"
read_when:
  - 处理 Telegram 或 grammY 通道
---
# grammY 集成（Telegram Bot API）


# 为什么使用 grammY
- 优先支持 TS 的 Bot API 客户端，具有内置的长轮询 + webhook 助手、中间件、错误处理、速率限制器。
- 比手动滚动 fetch + FormData 更干净的媒体助手；支持所有 Bot API 方法。
- 可扩展：通过自定义 fetch 支持代理、会话中间件（可选）、类型安全上下文。

# 我们提供的内容
- **单一客户端路径**：基于 fetch 的实现已删除；grammY 现在是唯一的 Telegram 客户端（发送 + 网关），默认启用 grammY 限流器。
- **网关**：`monitorTelegramProvider` 构建 grammY `Bot`，连接提及/白名单门控，通过 `getFile`/`download` 进行媒体下载，并使用 `sendMessage/sendPhoto/sendVideo/sendAudio/sendDocument` 传递回复。通过 `webhookCallback` 支持长轮询或 webhook。
- **代理**：可选的 `channels.telegram.proxy` 通过 grammY 的 `client.baseFetch` 使用 `undici.ProxyAgent`。
- **Webhook 支持**：`webhook-set.ts` 包装 `setWebhook/deleteWebhook`；`webhook.ts` 使用健康 + 优雅关闭托管回调。当设置 `channels.telegram.webhookUrl` 时，网关启用 webhook 模式（否则它会长轮询）。
- **会话**：直接聊天折叠到 agent 主会话（`agent:<agentId>:<mainKey>`）；群组使用 `agent:<agentId>:telegram:group:<chatId>`；回复路由回同一频道。
- **配置旋钮**：`channels.telegram.botToken`、`channels.telegram.dmPolicy`、`channels.telegram.groups`（白名单 + 提及默认值）、`channels.telegram.allowFrom`、`channels.telegram.groupAllowFrom`、`channels.telegram.groupPolicy`、`channels.telegram.mediaMaxMb`、`channels.telegram.linkPreview`、`channels.telegram.proxy`、`channels.telegram.webhookSecret`、`channels.telegram.webhookUrl`。
- **草稿流式传输**：可选的 `channels.telegram.streamMode` 在私人主题聊天中使用 `sendMessageDraft`（Bot API 9.3+）。这与渠道块流式传输是分开的。
- **测试**：grammy 模拟涵盖 DM + 群组提及门控和出站发送；仍然欢迎更多媒体/webhook 固定装置。

未决问题
- 如果我们遇到 Bot API 429，则使用可选的 grammY 插件（throttler）。
- 添加更多结构化的媒体测试（贴纸、语音笔记）。
- 使 webhook 监听端口可配置（目前固定为 8787，除非通过网关连接）。
