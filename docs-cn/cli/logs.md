---
summary: "`moltbot logs` CLI 参考(通过 RPC 跟踪 gateway 日志)"
read_when:
  - 您需要远程跟踪 Gateway 日志(无需 SSH)
  - 您想要用于工具的 JSON 日志行
---

# `moltbot logs`

通过 RPC 跟踪 Gateway 文件日志(在远程模式下工作)。

相关:
- 日志记录概述: [Logging](/logging)

## 示例

```bash
moltbot logs
moltbot logs --follow
moltbot logs --json
moltbot logs --limit 500
```
