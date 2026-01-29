---
summary: "每个频道的路由规则（WhatsApp、Telegram、Discord、Slack）和共享上下文"
read_when:
  - 更改频道路由或收件箱行为
---
# 频道和路由

Moltbot 将回复**路由回消息来源的频道**。模型不选择频道；路由是确定性的，由主机配置控制。

## 关键术语

- **频道**：`whatsapp`、`telegram`、`discord`、`slack`、`signal`、`imessage`、`webchat`。
- **AccountId**：每个频道的帐户实例（当支持时）。
- **AgentId**：隔离的工作区 + 会话存储（"大脑"）。
- **SessionKey**：用于存储上下文和控制并发的存储桶密钥。

## 会话密钥形状（示例）

直接聊天折叠到 agent 的**main** 会话：

- `agent:<agentId>:<mainKey>`（默认：`agent:main:main`）

组和频道按频道保持隔离：

- 组：`agent:<agentId>:<channel>:group:<id>`
- 频道/房间：`agent:<agentId>:<channel>:channel:<id>`

线程：

- Slack/Discord 线程将 `:thread:<threadId>` 附加到基本密钥。
- Telegram 论坛主题在组密钥中嵌入 `:topic:<topicId>`。

示例：

- `agent:main:telegram:group:-1001234567890:topic:42`
- `agent:main:discord:channel:123456:thread:987654`

## 路由规则（如何选择 agent）

路由为每条入站消息选择**一个 agent**：

1. **精确对等匹配**（带有 `peer.kind` + `peer.id` 的 `bindings`）。
2. **公会匹配**（Discord）通过 `guildId`。
3. **团队匹配**（Slack）通过 `teamId`。
4. **帐户匹配**（频道上的 `accountId`）。
5. **频道匹配**（该频道上的任何帐户）。
6. **默认 agent**（`agents.list[].default`，否则第一个列表条目，回退到 `main`）。

匹配的 agent 确定使用哪个工作区和会话存储。

## 广播组（运行多个 agents）

广播组允许您为同一个对等方运行**多个 agents**，**当 Moltbot 通常会回复时**（例如：在 WhatsApp 组中，在提及/激活门控之后）。

配置：

```json5
{
  broadcast: {
    strategy: "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"],
    "+15555550123": ["support", "logger"]
  }
}
```

请参阅：[Broadcast Groups](/broadcast-groups)。

## 配置概述

- `agents.list`：命名的 agent 定义（工作区、模型等）。
- `bindings`：将入站频道/帐户/对等方映射到 agents。

示例：

```json5
{
  agents: {
    list: [
      { id: "support", name: "Support", workspace: "~/clawd-support" }
    ]
  },
  bindings: [
    { match: { channel: "slack", teamId: "T123" }, agentId: "support" },
    { match: { channel: "telegram", peer: { kind: "group", id: "-100123" } }, agentId: "support" }
  ]
}
```

## 会话存储

会话存储位于状态目录下（默认 `~/.clawdbot`）：

- `~/.clawdbot/agents/<agentId>/sessions/sessions.json`
- JSONL 记录与存储相邻

您可以通过 `session.store` 和 `{agentId}` 模板覆盖存储路径。

## WebChat 行为

WebChat 附加到**选定的 agent**，默认为 agent 的主会话。因此，WebChat 允许您在一个地方查看该 agent 的跨频道上下文。

## 回复上下文

入站回复包括：
- `ReplyToId`、`ReplyToBody` 和 `ReplyToSender`（当可用时）。
- 引用的上下文作为 `[Replying to ...]` 块附加到 `Body`。

这在所有频道中都是一致的。
