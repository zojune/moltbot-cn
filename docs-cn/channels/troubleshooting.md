---
summary: "特定渠道的故障排除快捷方式（Discord/Telegram/WhatsApp）"
read_when:
  - 渠道已连接但消息不流动
  - 调查渠道配置错误（intents、权限、隐私模式）
---
# 渠道故障排除

从以下开始：

```bash
moltbot doctor
moltbot channels status --probe
```

`channels status --probe` 在检测到常见渠道配置错误时打印警告，并包括小型实时检查（凭据、某些权限/成员资格）。

## 渠道
- Discord：[/channels/discord#troubleshooting](/channels/discord#troubleshooting)
- Telegram：[/channels/telegram#troubleshooting](/channels/telegram#troubleshooting)
- WhatsApp：[/channels/whatsapp#troubleshooting-quick](/channels/whatsapp#troubleshooting-quick)

## Telegram 快速修复
- 日志显示 `HttpError: Network request for 'sendMessage' failed` 或 `sendChatAction` → 检查 IPv6 DNS。如果 `api.telegram.org` 首先解析为 IPv6 并且主机缺乏 IPv6 出站，请强制使用 IPv4 或启用 IPv6。请参阅 [/channels/telegram#troubleshooting](/channels/telegram#troubleshooting)。
- 日志显示 `setMyCommands failed` → 检查到 `api.telegram.org` 的出站 HTTPS 和 DNS 可达性（在锁定 VPS 或代理上常见）。
