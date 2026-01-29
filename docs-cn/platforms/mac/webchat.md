---
summary: "mac 应用如何嵌入网关 WebChat 以及如何调试它"
read_when:
  - 调试 mac WebChat 视图或环回端口
---
# WebChat（macOS 应用）

macOS 菜单栏应用将 WebChat UI 嵌入为本机 SwiftUI 视图。它
连接到网关并默认为所选
代理的**主会话**（以及其他会话的会话切换器）。

- **本地模式：** 直接连接到本地网关 WebSocket。
- **远程模式：** 通过 SSH 转发网关控制端口并使用该
  隧道作为数据平面。

## 启动和调试

- 手动：Lobster 菜单 → "打开聊天"。
- 自动打开以进行测试：
  ```bash
  dist/Moltbot.app/Contents/MacOS/Moltbot --webchat
  ```
- 日志：`./scripts/clawlog.sh`（子系统 `bot.molt`，类别 `WebChatSwiftUI`）。

## 连接方式

- 数据平面：网关 WS 方法 `chat.history`、`chat.send`、`chat.abort`、
  `chat.inject` 和事件 `chat`、`agent`、`presence`、`tick`、`health`。
- 会话：默认为主会话（`main`，或当作用域为全局时为 `global`）。UI 可以在会话之间切换。
- 入门使用专用会话，以使首次运行设置保持分离。

## 安全表面

- 远程模式仅通过 SSH 转发网关 WebSocket 控制端口。

## 已知限制

- UI 针对聊天会话进行了优化（不是完整的浏览器沙箱）。
