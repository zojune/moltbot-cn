---
summary: "WebSocket gateway 架构、组件和客户端流程"
read_when:
  - 处理 gateway 协议、客户端或传输
---
# Gateway 架构

最后更新：2026-01-22

## 概述

- 单个长寿命的 **Gateway** 拥有所有消息传递表面（WhatsApp 通过
  Baileys、Telegram 通过 grammY、Slack、Discord、Signal、iMessage、WebChat）。
- 控制平面客户端（macOS 应用、CLI、Web UI、自动化）通过配置的绑定主机上的 **WebSocket** 连接到 Gateway（默认
  `127.0.0.1:18789`）。
- **节点**（macOS/iOS/Android/headless）也通过 **WebSocket** 连接，但
  声明 `role: node` 并具有明确的 caps/命令。
- 每个主机一个 Gateway；它是打开 WhatsApp 会话的唯一地方。
- **canvas 主机**（默认 `18793`）提供 agent 可编辑的 HTML 和 A2UI。

## 组件和流程

### Gateway（守护进程）
- 维护提供程序连接。
- 公开类型化的 WS API（请求、响应、服务器推送事件）。
- 根据 JSON Schema 验证入站帧。
- 发出 `agent`、`chat`、`presence`、`health`、`heartbeat`、`cron` 等事件。

### 客户端（mac 应用 / CLI / Web 管理）
- 每个客户端一个 WS 连接。
- 发送请求（`health`、`status`、`send`、`agent`、`system-presence`）。
- 订阅事件（`tick`、`agent`、`presence`、`shutdown`）。

### 节点（macOS / iOS / Android / headless）
- 使用 `role: node` 连接到**相同的 WS 服务器**。
- 在 `connect` 中提供设备身份；配对是**基于设备的**（角色 `node`），批准
  存在于设备配对存储中。
- 公开 `canvas.*`、`camera.*`、`screen.record`、`location.get` 等命令。

协议详情：
- [Gateway protocol](/gateway/protocol)

### WebChat
- 静态 UI，使用 Gateway WS API 进行聊天历史记录和发送。
- 在远程设置中，通过与其他客户端相同的 SSH/Tailscale 隧道连接。

## 连接生命周期（单个客户端）

```
Client                    Gateway
  |                          |
  |---- req:connect -------->|
  |<------ res (ok) ---------|   （或 res error + close）
  |   (payload=hello-ok 携带快照：presence + health)
  |                          |
  |<------ event:presence ---|
  |<------ event:tick -------|
  |                          |
  |------- req:agent ------->|
  |<------ res:agent --------|   (ack: {runId,status:"accepted"})
  |<------ event:agent ------|   （流式传输）
  |<------ res:agent --------|   (final: {runId,status,summary})
  |                          |
```

## 线路协议（摘要）

- 传输：WebSocket，带有 JSON 负载的文本帧。
- 第一帧**必须**是 `connect`。
- 握手后：
  - 请求：`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  - 事件：`{type:"event", event, payload, seq?, stateVersion?}`
- 如果设置了 `CLAWDBOT_GATEWAY_TOKEN`（或 `--token`），`connect.params.auth.token`
  必须匹配，否则套接字关闭。
- 副作用方法（`send`、`agent`）需要幂等性密钥才能安全重试；服务器保持短期去重缓存。
- 节点必须在 `connect` 中包含 `role: "node"` 以及 caps/命令/权限。

## 配对 + 本地信任

- 所有 WS 客户端（操作员 + 节点）在 `connect` 上包含**设备身份**。
- 新设备 ID 需要配对批准；Gateway 为后续连接颁发**设备令牌**。
- **本地**连接（环回或 gateway 主机自己的 tailnet 地址）可以自动批准，以保持同主机 UX 流畅。
- **非本地**连接必须签署 `connect.challenge` nonce 并需要明确批准。
- Gateway 认证（`gateway.auth.*`）仍然适用于**所有**连接，本地或远程。

详情：[Gateway protocol](/gateway/protocol)、[Pairing](/start/pairing)、
[Security](/gateway/security)。

## 协议类型和代码生成

- TypeBox schemas 定义协议。
- JSON Schema 从这些 schemas 生成。
- Swift 模型从 JSON Schema 生成。

## 远程访问

- 首选：Tailscale 或 VPN。
- 替代方案：SSH 隧道
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
- 通过隧道应用相同的握手 + 认证令牌。
- 可以为远程设置中的 WS 启用 TLS + 可选的固定。

## 操作快照

- 启动：`moltbot gateway`（前台，日志记录到 stdout）。
- 运行状况：通过 WS 进行 `health`（也包含在 `hello-ok` 中）。
- 监督：launchd/systemd 用于自动重启。

## 不变量

- 恰好一个 Gateway 控制每个主机单个 Baileys 会话。
- 握手是强制性的；任何非 JSON 或非连接的第一帧都是硬关闭。
- 事件不会重放；客户端必须在间隙时刷新。
