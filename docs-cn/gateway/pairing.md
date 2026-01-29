---
summary: "Gateway 拥有的节点配对（选项 B），用于 iOS 和其他远程节点"
read_when:
  - 在没有 macOS UI 的情况下实现节点配对批准
  - 为批准远程节点添加 CLI 流程
  - 使用节点管理扩展 gateway 协议
---
# Gateway 拥有的配对（选项 B）

在 Gateway 拥有的配对中，**Gateway** 是允许哪些节点加入的单一事实来源。
UI（macOS 应用、未来的客户端）只是批准或拒绝待处理请求的前端。

**重要：** WS 节点在 `connect` 期间使用**设备配对**（角色 `node`）。
`node.pair.*` 是一个单独的配对存储，**不**限制 WS 握手。
只有显式调用 `node.pair.*` 的客户端才使用此流程。

## 概念

- **待处理请求**：节点请求加入；需要批准。
- **已配对节点**：具有已颁发认证令牌的已批准节点。
- **传输**：Gateway WS 端点转发请求但不决定成员身份。（不推荐/已移除旧的 TCP 网桥支持。）

## 配对如何工作

1. 节点连接到 Gateway WS 并请求配对。
2. Gateway 存储**待处理请求**并发出 `node.pair.requested`。
3. 您批准或拒绝请求（CLI 或 UI）。
4. 批准后，Gateway 颁发一个**新令牌**（令牌在重新配对时轮换）。
5. 节点使用令牌重新连接，现在"已配对"。

待处理请求在 **5 分钟**后自动过期。

## CLI 工作流（无头友好）

```bash
moltbot nodes pending
moltbot nodes approve <requestId>
moltbot nodes reject <requestId>
moltbot nodes status
moltbot nodes rename --node <id|name|ip> --name "Living Room iPad"
```

`nodes status` 显示已配对/已连接的节点及其功能。

## API 表面（gateway 协议）

事件：
- `node.pair.requested` — 创建新的待处理请求时发出。
- `node.pair.resolved` — 请求被批准/拒绝/过期时发出。

方法：
- `node.pair.request` — 创建或重用待处理请求。
- `node.pair.list` — 列出待处理 + 已配对的节点。
- `node.pair.approve` — 批准待处理请求（颁发令牌）。
- `node.pair.reject` — 拒绝待处理请求。
- `node.pair.verify` — 验证 `{ nodeId, token }`。

注意事项：
- `node.pair.request` 对于每个节点是幂等的：重复调用返回相同的待处理请求。
- 批准**总是**生成新令牌；`node.pair.request` 从不返回令牌。
- 请求可能包含 `silent: true` 作为自动批准流程的提示。

## 自动批准（macOS 应用）

如果满足以下条件，macOS 应用可以选择尝试**静默批准**：
- 请求被标记为 `silent`，并且
- 应用可以使用同一用户验证到 gateway 主机的 SSH 连接。

如果静默批准失败，它将回退到正常的"批准/拒绝"提示。

## 存储（本地、私有）

配对状态存储在 Gateway 状态目录下（默认 `~/.clawdbot`）：

- `~/.clawdbot/nodes/paired.json`
- `~/.clawdbot/nodes/pending.json`

如果您覆盖 `CLAWDBOT_STATE_DIR`，`nodes/` 文件夹也会随之移动。

安全注意事项：
- 令牌是秘密；将 `paired.json` 视为敏感信息。
- 轮换令牌需要重新批准（或删除节点条目）。

## 传输行为

- 传输是**无状态的**；它不存储成员身份。
- 如果 Gateway 离线或配对被禁用，节点无法配对。
- 如果 Gateway 处于远程模式，配对仍然针对远程 Gateway 的存储进行。
