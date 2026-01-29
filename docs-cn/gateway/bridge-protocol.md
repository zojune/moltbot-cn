---
summary: "桥接协议(旧版节点):TCP JSONL、配对、作用域 RPC"
read_when:
  - 构建或调试节点客户端(iOS/Android/macOS 节点模式)
  - 调查配对或桥接认证失败
  - 审计 gateway 暴露的节点表面
---

# 桥接协议(旧版节点传输)

桥接协议是一种**旧版**节点传输(TCP JSONL)。新的节点客户端
应改用统一的 Gateway WebSocket 协议。

如果您正在构建操作员或节点客户端,请使用
[Gateway protocol](/gateway/protocol)。

**注意:**当前 Moltbot 构建版不再附带 TCP 桥接监听器;本文档仅供参考。
旧的 `bridge.*` 配置键不再属于配置架构的一部分。

## 为什么我们有两者

- **安全边界**:桥接暴露了一个小的允许列表,而不是
  完整的 gateway API 表面。
- **配对 + 节点身份**:节点准入由 gateway 拥有,并绑定
  到每个节点的令牌。
- **发现 UX**:节点可以通过 LAN 上的 Bonjour 发现 gateway,或通过 tailnet 直接连接。
- **环回 WS**:完整的 WS 控制平面保持本地,除非通过 SSH 隧道传输。

## 传输

- TCP,每行一个 JSON 对象(JSONL)。
- 可选 TLS(当 `bridge.tls.enabled` 为 true 时)。
- 旧的默认监听器端口是 `18790`(当前构建版不启动 TCP 桥接)。

当启用 TLS 时,发现 TXT 记录包括 `bridgeTls=1` 加上
`bridgeTlsSha256`,以便节点可以固定证书。

## 握手 + 配对

1) 客户端发送带有节点元数据 + 令牌(如果已配对)的 `hello`。
2) 如果未配对,gateway 回复 `error`(`NOT_PAIRED`/`UNAUTHORIZED`)。
3) 客户端发送 `pair-request`。
4) Gateway 等待批准,然后发送 `pair-ok` 和 `hello-ok`。

`hello-ok` 返回 `serverName`,并可能包括 `canvasHostUrl`。

## 帧

客户端 → Gateway:
- `req` / `res`:作用域 gateway RPC(chat、sessions、config、health、voicewake、skills.bins)
- `event`:节点信号(语音转录、代理请求、chat 订阅、exec 生命周期)

Gateway → 客户端:
- `invoke` / `invoke-res`:节点命令(`canvas.*`、`camera.*`、`screen.record`、
  `location.get`、`sms.send`)
- `event`:订阅会话的聊天更新
- `ping` / `pong`:保活

旧的允许列表强制执行位于 `src/gateway/server-bridge.ts` 中(已删除)。

## Exec 生命周期事件

节点可以发出 `exec.finished` 或 `exec.denied` 事件以显示 system.run 活动。
这些被映射到 gateway 中的系统事件。(旧节点可能仍发出 `exec.started`。)

负载字段(除非注明,均为可选):
- `sessionKey`(必需):接收系统事件的代理会话。
- `runId`:用于分组的唯一 exec id。
- `command`:原始或格式化的命令字符串。
- `exitCode`、`timedOut`、`success`、`output`:完成详细信息(仅完成)。
- `reason`:拒绝原因(仅拒绝)。

## Tailnet 使用

- 将桥接绑定到 tailnet IP:`~/.clawdbot/moltbot.json` 中的
  `bridge.bind: "tailnet"`。
- 客户端通过 MagicDNS 名称或 tailnet IP 连接。
- Bonjour **不**跨网络;使用手动主机/端口或在需要时使用广域 DNS-SD。

## 版本控制

桥接当前是**隐式 v1**(无最小/最大协商)。预期向后兼容;
在任何重大更改之前添加桥接协议版本字段。
