---
summary: "配对概述：批准谁可以 DM 你 + 哪些节点可以加入"
read_when:
  - 设置 DM 访问控制
  - 配对新的 iOS/Android 节点
  - 审查 Moltbot 安全姿态
---

# 配对

"配对"是 Moltbot 的明确**所有者批准**步骤。
它用于两个地方：

1) **DM 配对**（谁被允许与机器人交谈）
2) **节点配对**（哪些设备/节点被允许加入网关网络）

安全上下文：[安全](/gateway/security)

## 1) DM 配对（入站聊天访问）

当频道配置了 DM 策略 `pairing` 时，未知发送者会获得一个短代码，在您批准之前，他们的消息**不会被处理**。

默认 DM 策略记录在：[安全](/gateway/security)

配对代码：
- 8 个字符，大写，没有歧义字符（`0O1I`）。
- **1 小时后过期**。机器人仅在新请求创建时发送配对消息（每个发送者大约每小时一次）。
- 待处理的 DM 配对请求默认每个频道最多 **3 个**；额外的请求被忽略，直到一个过期或被批准。

### 批准发送者

```bash
moltbot pairing list telegram
moltbot pairing approve telegram <CODE>
```

支持的频道：`telegram`、`whatsapp`、`signal`、`imessage`、`discord`、`slack`。

### 状态存储位置

存储在 `~/.clawdbot/credentials/` 下：
- 待处理的请求：`<channel>-pairing.json`
- 已批准的允许列表存储：`<channel>-allowFrom.json`

将这些视为敏感信息（它们控制对你助手的访问）。


## 2) 节点设备配对（iOS/Android/macOS/无头节点）

节点作为**设备**连接到网关，具有 `role: node`。网关创建一个必须批准的设备配对请求。

### 批准节点设备

```bash
moltbot devices list
moltbot devices approve <requestId>
moltbot devices reject <requestId>
```

### 状态存储位置

存储在 `~/.clawdbot/devices/` 下：
- `pending.json`（短期；待处理的请求过期）
- `paired.json`（配对的设备 + 令牌）

### 注意事项

- 旧版 `node.pair.*` API（CLI：`moltbot nodes pending/approve`）是一个单独的网关拥有的配对存储。WS 节点仍然需要设备配对。


## 相关文档

- 安全模型 + 提示注入：[安全](/gateway/security)
- 安全更新（运行 doctor）：[更新](/install/updating)
- 频道配置：
  - Telegram：[Telegram](/channels/telegram)
  - WhatsApp：[WhatsApp](/channels/whatsapp)
  - Signal：[Signal](/channels/signal)
  - iMessage：[iMessage](/channels/imessage)
  - Discord：[Discord](/channels/discord)
  - Slack：[Slack](/channels/slack)
