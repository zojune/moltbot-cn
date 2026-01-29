---
summary: "Loopback WebChat static 主机 and Gateway WS 用法 for chat UI"
read_when: 
  - Debugging or configuring WebChat access
---
# WebChat (Gateway WebSocket UI)


状态: the macOS/iOS SwiftUI chat UI talks directly to the Gateway WebSocket.

## What it is
- A native chat UI for the Gateway (no embedded browser and no local static 服务器).
- Uses the same 会话 and routing 规则 as other 渠道.
- Deterministic routing: replies always go back to WebChat.

## 快速开始
1) Start the Gateway.
2) Open the WebChat UI (macOS/iOS 应用) or the Control UI chat tab.
3) Ensure Gateway 认证 is configured (必需 默认情况下, even on loopback).

## 工作原理 (行为)
- The UI connects to the Gateway WebSocket and uses `chat.history`, `chat.send`, and `chat.inject`.
- `chat.inject` appends an assistant 注意 directly to the transcript and broadcasts it to the UI (no 代理 run).
- History is always fetched from the Gateway (no local 文件 watching).
- If the Gateway is unreachable, WebChat is read-only.

## Remote use
- Remote mode tunnels the Gateway WebSocket over SSH/Tailscale.
- You do not need to run a separate WebChat 服务器.

## 配置 参考 (WebChat)
Full 配置: [配置](/Gateway/配置)

渠道 选项:
- No dedicated `webchat.*` block. WebChat uses the Gateway 端点 + 认证 设置 below.

相关 global 选项:
- `gateway.port`, `gateway.bind`: WebSocket 主机/端口.
- `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket 认证.
- `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: remote Gateway target.
- `session.*`: 会话 存储 and main 键 defaults.
