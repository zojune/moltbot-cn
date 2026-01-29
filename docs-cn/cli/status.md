---
summary: "`moltbot status` CLI 参考(诊断、探测、使用快照)"
read_when:
  - 您想对频道健康状况 + 最近的会话收件人进行快速诊断
  - 您想要一个可粘贴的"全部"状态用于调试
---

# `moltbot status`

频道 + 会话的诊断。

```bash
moltbot status
moltbot status --all
moltbot status --deep
moltbot status --usage
```

注意事项:
- `--deep` 运行实时探测(WhatsApp Web + Telegram + Discord + Google Chat + Slack + Signal)。
- 当配置多个 agent 时,输出包括每个 agent 的会话存储。
- 概述包括 Gateway + node 主机服务安装/运行时状态(如果可用)。
- 概述包括更新频道 + git SHA(用于源代码检出)。
- 更新信息显示在概述中;如果有可用更新,状态会打印提示以运行 `moltbot update`(参见 [Updating](/install/updating))。
