---
summary: "`moltbot agent` CLI 参考(通过 Gateway 发送单个 agent 轮次)"
read_when:
  - 您想从脚本运行单个 agent 轮次(可选择传递回复)
---

# `moltbot agent`

通过 Gateway 运行一个 agent 轮次(使用 `--local` 进行嵌入式运行)。
使用 `--agent <id>` 直接定位配置的 agent。

相关:
- Agent 发送工具: [Agent send](/tools/agent-send)

## 示例

```bash
moltbot agent --to +15555550123 --message "status update" --deliver
moltbot agent --agent ops --message "Summarize logs"
moltbot agent --session-id 1234 --message "Summarize inbox" --thinking medium
moltbot agent --agent ops --message "Generate report" --deliver --reply-channel slack --reply-to "#reports"
```
