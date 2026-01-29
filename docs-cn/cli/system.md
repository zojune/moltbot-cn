---
summary: "`moltbot system` CLI 参考(系统事件、心跳、存在)"
read_when:
  - 您想在无需创建 cron 任务的情况下将系统事件排入队列
  - 您需要启用或禁用心跳
  - 您想检查系统存在条目
---

# `moltbot system`

Gateway 的系统级助手:将系统事件排入队列、控制心跳并查看存在。

## 常用命令

```bash
moltbot system event --text "Check for urgent follow-ups" --mode now
moltbot system heartbeat enable
moltbot system heartbeat last
moltbot system presence
```

## `system event`

在**主**会话上将系统事件排入队列。下一次心跳将把它作为提示中的 `System:` 行注入。使用 `--mode now` 立即触发心跳;`next-heartbeat` 等待下一次计划的任务。

标志:
- `--text <text>`:必需的系统事件文本。
- `--mode <mode>`:`now` 或 `next-heartbeat`(默认)。
- `--json`:机器可读的输出。

## `system heartbeat last|enable|disable`

心跳控制:
- `last`:显示最后一次心跳事件。
- `enable`:重新打开心跳(如果它们被禁用,请使用此项)。
- `disable`:暂停心跳。

标志:
- `--json`:机器可读的输出。

## `system presence`

列出 Gateway 当前知道的系统存在条目(nodes、实例和类似的状态行)。

标志:
- `--json`:机器可读的输出。

## 注意事项

- 需要您的当前配置可以访问的运行中的 Gateway(本地或远程)。
- 系统事件是短暂的,不会在重启之间持久化。
