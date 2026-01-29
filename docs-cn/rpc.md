---
summary: "外部 CLI（signal-cli、imsg）的 RPC 适配器和网关模式"
read_when:
  - 添加或更改外部 CLI 集成
  - 调试 RPC 适配器（signal-cli、imsg）
---
# RPC 适配器

Moltbot 通过 JSON-RPC 集成外部 CLI。目前使用两种模式。

## 模式 A：HTTP 守护进程（signal-cli）
- `signal-cli` 作为守护进程运行，通过 HTTP 进行 JSON-RPC。
- 事件流是 SSE（`/api/v1/events`）。
- 健康探测：`/api/v1/check`。
- 当 `channels.signal.autoStart=true` 时，Moltbot 拥有生命周期。

有关设置和端点，请参阅 [Signal](/channels/signal)。

## 模式 B：stdio 子进程（imsg）
- Moltbot 生成 `imsg rpc` 作为子进程。
- JSON-RPC 是通过 stdin/stdout 逐行分隔的（每行一个 JSON 对象）。
- 无 TCP 端口，无需守护进程。

使用的核心方法：
- `watch.subscribe` → 通知（`method: "message"`）
- `watch.unsubscribe`
- `send`
- `chats.list`（探测/诊断）

有关设置和寻址（`chat_id` 优先），请参阅 [iMessage](/channels/imessage)。

## 适配器指南
- 网关拥有进程（启动/停止与提供商生命周期相关）。
- 保持 RPC 客户端弹性：超时，退出时重启。
- 优先使用稳定的 ID（例如 `chat_id`）而不是显示字符串。
