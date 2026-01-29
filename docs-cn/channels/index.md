---
summary: "Moltbot 可以连接的消息传递平台"
read_when:
  - 您想为 Moltbot 选择聊天渠道
  - 您需要支持的消息传递平台的快速概述
---
# 聊天渠道

Moltbot 可以在您已经使用的任何聊天应用程序上与您交谈。每个渠道通过网关连接。
到处都支持文本；媒体和反应因渠道而异。

## 支持的渠道

- [WhatsApp](/channels/whatsapp) — 最受欢迎；使用 Baileys，需要 QR 配对。
- [Telegram](/channels/telegram) — 通过 grammY 的 Bot API；支持群组。
- [Discord](/channels/discord) — Discord Bot API + 网关；支持服务器、频道和 DM。
- [Slack](/channels/slack) — Bolt SDK；工作区应用程序。
- [Google Chat](/channels/googlechat) — 通过 HTTP webhook 的 Google Chat API 应用程序。
- [Mattermost](/channels/mattermost) — Bot API + WebSocket；频道、群组、DM（插件，单独安装）。
- [Signal](/channels/signal) — signal-cli；注重隐私。
- [BlueBubbles](/channels/bluebubbles) — **推荐用于 iMessage**；使用 BlueBubbles macOS 服务器 REST API，具有完整功能支持（编辑、撤回、效果、反应、群组管理 — 编辑目前在 macOS 26 Tahoe 上损坏）。
- [iMessage](/channels/imessage) — 仅 macOS；通过 imsg 的原生集成（旧版，新设置请考虑 BlueBubbles）。
- [Microsoft Teams](/channels/msteams) — Bot Framework；企业支持（插件，单独安装）。
- [LINE](/channels/line) — LINE Messaging API 机器人（插件，单独安装）。
- [Nextcloud Talk](/channels/nextcloud-talk) — 通过 Nextcloud Talk 的自托管聊天（插件，单独安装）。
- [Matrix](/channels/matrix) — Matrix 协议（插件，单独安装）。
- [Nostr](/channels/nostr) — 通过 NIP-04 的去中心化 DM（插件，单独安装）。
- [Tlon](/channels/tlon) — 基于 Urbit 的消息传递程序（插件，单独安装）。
- [Twitch](/channels/twitch) — 通过 IRC 连接的 Twitch 聊天（插件，单独安装）。
- [Zalo](/channels/zalo) — Zalo Bot API；越南流行的消息传递程序（插件，单独安装）。
- [Zalo Personal](/channels/zalouser) — 通过 QR 登录的 Zalo 个人帐户（插件，单独安装）。
- [WebChat](/web/webchat) — 通过 WebSocket 的网关 WebChat UI。

## 注意事项

- 渠道可以同时运行；配置多个渠道，Moltbot 将按聊天路由。
- 最快的设置通常是 **Telegram**（简单的机器人令牌）。WhatsApp 需要 QR 配对，并且在磁盘上存储更多状态。
- 群组行为因渠道而异；请参阅 [群组](/concepts/groups)。
- 出于安全考虑，强制执行 DM 配对和白名单；请参阅 [安全性](/gateway/security)。
- Telegram 内部：[grammY 说明](/channels/grammy)。
- 故障排除：[渠道故障排除](/channels/troubleshooting)。
- 模型提供者单独记录；请参阅 [模型提供者](/providers/models)。
