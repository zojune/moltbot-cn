---
summary: "WhatsApp 组消息处理的行为和配置（mentionPatterns 在所有表面之间共享）"
read_when:
  - 更改组消息规则或提及
---
# 组消息（WhatsApp web 频道）

目标：让 Clawd 坐在 WhatsApp 组中，仅在收到 ping 时唤醒，并使该线程与个人 DM 会话分开。

注意：`agents.list[].groupChat.mentionPatterns` 现在也被 Telegram/Discord/Slack/iMessage 使用；本文档侧重于 WhatsApp 特定的行为。对于多 agent 设置，请为每个 agent 设置 `agents.list[].groupChat.mentionPatterns`（或使用 `messages.groupChat.mentionPatterns` 作为全局回退）。

## 已实现的内容（2025-12-03）
- 激活模式：`mention`（默认）或 `always`。`mention` 需要 ping（通过 `mentionedJids` 的真实 WhatsApp @提及、正则表达式模式或文本中任何位置的 bot E.164）。`always` 在每条消息上唤醒 agent，但它应该只在能增加有意义的价值时回复；否则它返回静默令牌 `NO_REPLY`。可以在配置中设置默认值（`channels.whatsapp.groups`）并通过 `/activation` 按组覆盖。当设置 `channels.whatsapp.groups` 时，它也充当组允许列表（包括 `"*"` 以允许所有）。
- 组策略：`channels.whatsapp.groupPolicy` 控制是否接受组消息（`open|disabled|allowlist`）。`allowlist` 使用 `channels.whatsapp.groupAllowFrom`（回退：显式 `channels.whatsapp.allowFrom`）。默认是 `allowlist`（在您添加发件人之前被阻止）。
- 每组会话：会话密钥看起来像 `agent:<agentId>:whatsapp:group:<jid>`，因此 `/verbose on` 或 `/think high` 等命令（作为独立消息发送）的范围是该组；个人 DM 状态不受影响。跳过组线程的心跳。
- 上下文注入：**仅待处理**组消息（默认 50）*未*触发运行的在前缀 `[Chat messages since your last reply - for context]` 下，触发行在 `[Current message - respond to this]` 下。会话中已有的消息不会重新注入。
- 发件人显示：每个组批次现在以 `[from: Sender Name (+E164)]` 结尾，以便 Pi 知道谁在说话。
- 临时/查看一次：我们在提取文本/提及之前展开它们，因此它们内部的 ping 仍会触发。
- 组系统提示：在组会话的第一轮（以及每当 `/activation` 更改模式时），我们在系统提示中注入一个简短的简介，如 `You are replying inside the WhatsApp group "<subject>". Group members: Alice (+44...), Bob (+43...), … Activation: trigger-only … Address the specific sender noted in the message context.` 如果元数据不可用，我们仍告诉 agent 它是组聊天。

## 配置示例（WhatsApp）

将 `groupChat` 块添加到 `~/.clawdbot/moltbot.json`，以便显示名称 ping 在 WhatsApp 从文本正文中剥离视觉 `@` 时也能工作：

```json5
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          historyLimit: 50,
          mentionPatterns: [
            "@?moltbot",
            "\+?15555550123"
          ]
        }
      }
    ]
  }
}
```

注意：
- 正则表达式不区分大小写；它们涵盖像 `@moltbot` 这样的显示名称 ping 以及带或不带 `+`/空格的原始号码。
- 当有人点击联系人时，WhatsApp 仍通过 `mentionedJids` 发送规范提及，因此很少需要号码回退，但它是一个有用的安全网。

### 激活命令（仅所有者）

使用组聊天命令：
- `/activation mention`
- `/activation always`

只有所有者号码（来自 `channels.whatsapp.allowFrom`，或未设置时 bot 自己的 E.164）可以更改此设置。在组中作为独立消息发送 `/status` 以查看当前激活模式。

## 如何使用
1) 将您的 WhatsApp 帐户（运行 Moltbot 的那个）添加到组。
2) 说 `@moltbot …`（或包含号码）。除非您设置 `groupPolicy: "open"`，否则只有允许的发件人可以触发它。
3) agent 提示将包括最近的组上下文以及尾随的 `[from: …]` 标记，以便它可以称呼正确的人。
4) 会话级指令（`/verbose on`、`/think high`、`/new` 或 `/reset`、`/compact`）仅适用于该组的会话；将它们作为独立消息发送，以便它们注册。您的个人 DM 会话保持独立。

## 测试/验证
- 手动冒烟：
  - 在组中发送 `@clawd` ping 并确认引用发件人姓名的回复。
  - 发送第二个 ping 并验证历史记录块在下一轮被包含然后清除。
- 检查 gateway 日志（使用 `--verbose` 运行）以查看 `inbound web message` 条目，显示 `from: <groupJid>` 和 `[from: …]` 后缀。

## 已知注意事项
- 跳过组的心跳以避免嘈杂的广播。
- 回显抑制使用组合批次字符串；如果您在没有提及的情况下发送相同的文本两次，只有第一个会得到响应。
- 会话存储条目将在会话存储中显示为 `agent:<agentId>:whatsapp:group:<jid>`（默认 `~/.clawdbot/agents/<agentId>/sessions/sessions.json`）；缺少的条目只是意味着组尚未触发运行。
- 组中的输入指示器遵循 `agents.defaults.typingMode`（默认：未提及时为 `message`）。
