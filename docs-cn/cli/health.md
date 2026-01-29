---
summary: "`moltbot health` CLI 参考(通过 RPC 获取 gateway 健康端点)"
read_when:
  - 您想快速检查运行的 Gateway 的健康状况
---

# `moltbot health`

从运行的 Gateway 获取健康状况。

```bash
moltbot health
moltbot health --json
moltbot health --verbose
```

注意事项:
- `--verbose` 运行实时探测,并在配置多个账户时打印每个账户的时间。
- 当配置多个 agent 时,输出包括每个 agent 的会话存储。
