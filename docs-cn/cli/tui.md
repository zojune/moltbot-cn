---
summary: "`moltbot tui` CLI 参考(连接到 Gateway 的终端 UI)"
read_when:
  - 您需要 Gateway 的终端 UI(远程友好)
  - 您想从脚本传递 url/token/会话
---

# `moltbot tui`

打开连接到 Gateway 的终端 UI。

相关:
- TUI 指南: [TUI](/tui)

## 示例

```bash
moltbot tui
moltbot tui --url ws://127.0.0.1:18789 --token <token>
moltbot tui --session main --deliver
```
