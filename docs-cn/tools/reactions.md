---
summary: "跨频道共享的反应语义"
read_when:
  - 在任何频道中使用反应
---
# 反应工具

跨频道共享反应语义：

- 添加表情符号时需要 `emoji`。
- 如果支持，`emoji=""` 删除机器人的表情符号。
- 如果支持，`remove: true` 删除指定的表情符号（需要 `emoji`）。

频道说明：

- **Discord/Slack**：空 `emoji` 删除机器人在消息上的所有反应；`remove: true` 仅删除该表情符号。
- **Google Chat**：空 `emoji` 删除应用在消息上的反应；`remove: true` 仅删除该表情符号。
- **Telegram**：空 `emoji` 删除机器人的反应；`remove: true` 也删除反应，但仍需要非空的 `emoji` 用于工具验证。
- **WhatsApp**：空 `emoji` 删除机器人反应；`remove: true` 映射到空表情符号（仍然需要 `emoji`）。
- **Signal**：当启用 `channels.signal.reactionNotifications` 时，入站反应通知会发出系统事件。
