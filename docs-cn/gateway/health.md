---
summary: "频道连接的健康检查步骤"
read_when:
  - 诊断 WhatsApp 频道健康状况
---
# 健康检查(CLI)

验证频道连接的简短指南,无需猜测。

## 快速检查
- `moltbot status` — 本地摘要:gateway 可达性/模式、更新提示、已链接频道认证时间、会话 + 最近活动。
- `moltbot status --all` — 完整的本地诊断(只读、彩色、可安全粘贴以进行调试)。
- `moltbot status --deep` — 还探测运行中的 Gateway(支持时进行每个频道的探测)。
- `moltbot health --json` — 向运行的 Gateway 请求完整的健康快照(仅 WS;无直接 Baileys 套接字)。
- 在 WhatsApp/WebChat 中将 `/status` 作为独立消息发送,以获取状态回复而不调用代理。
- 日志:tail `/tmp/moltbot/moltbot-*.log` 并筛选 `web-heartbeat`、`web-reconnect`、`web-auto-reply`、`web-inbound`。

## 深度诊断
- 磁盘上的凭据:`ls -l ~/.clawdbot/credentials/whatsapp/<accountId>/creds.json`(mtime 应该是最近的)。
- 会话存储:`ls -l ~/.clawdbot/agents/<agentId>/sessions/sessions.json`(可以在配置中覆盖路径)。计数和最近的收件人通过 `status` 显示。
- 重新链接流程:当日志中出现状态代码 409–515 或 `loggedOut` 时,使用 `moltbot channels logout && moltbot channels login --verbose`。(注意:在配对后,状态 515 的 QR 登录流程会自动重启一次)。

## 当出现故障时
- `logged out` 或状态 409–515 → 使用 `moltbot channels logout` 然后使用 `moltbot channels login` 重新链接。
- Gateway 无法访问 → 启动它:`moltbot gateway --port 18789`(如果端口繁忙,使用 `--force`)。
- 没有收到入站消息 → 确认已链接的电话在线并且发件人被允许(`channels.whatsapp.allowFrom`);对于群聊,确保允许列表 + 提及规则匹配(`channels.whatsapp.groups`、`agents.list[].groupChat.mentionPatterns`)。

## 专用的"health"命令
`moltbot health --json` 向运行的 Gateway 询问其健康快照(CLI 无直接频道套接字)。它在可用时报告已链接的凭据/认证时间、每个频道的探测摘要、会话存储摘要以及探测持续时间。如果 Gateway 无法访问或探测失败/超时,它将以非零退出。使用 `--timeout <ms>` 覆盖 10 秒的默认值。
