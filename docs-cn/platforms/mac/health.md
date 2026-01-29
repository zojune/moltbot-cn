---
summary: "macOS 应用如何报告网关/Baileys 健康状态"
read_when:
  - 调试 mac 应用健康指示器
---
# macOS 上的健康检查

如何从菜单栏应用查看链接的频道是否健康。

## 菜单栏
- 状态点现在反映 Baileys 健康状况：
  - 绿色：已链接 + socket 最近打开。
  - 橙色：正在连接/重试。
  - 红色：已注销或探测失败。
- 次要行显示"已链接 · 认证 12m"或显示失败原因。
- "运行健康检查"菜单项触发按需探测。

## 设置
- 常规选项卡获得一个健康卡，显示：已链接认证年龄、会话存储路径/计数、上次检查时间、上次错误/状态代码，以及运行健康检查/显示日志的按钮。
- 使用缓存的快照，以便 UI 瞬间加载并在离线时优雅地回退。
- **频道选项卡**显示频道状态 + WhatsApp/Telegram 的控制（登录 QR、注销、探测、上次断开连接/错误）。

## 探测如何工作
- 应用每约 60 秒并通过 `ShellExecutor` 按需运行 `moltbot health --json`。探测加载凭据并报告状态而不发送消息。
- 分别缓存最后一个良好的快照和最后一个错误，以避免闪烁；显示每个快照的时间戳。

## 如有疑问
- 您仍然可以使用 [网关健康](/gateway/health) 中的 CLI 流程（`moltbot status`、`moltbot status --deep`、`moltbot health --json`）并尾随 `/tmp/moltbot/moltbot-*.log` 以查找 `web-heartbeat` / `web-reconnect`。
