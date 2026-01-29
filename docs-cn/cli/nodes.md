---
summary: "`moltbot nodes` CLI 参考(列表/状态/批准/调用、相机/画布/屏幕)"
read_when:
  - 您正在管理已配对的 nodes(相机、屏幕、画布)
  - 您需要批准请求或调用 node 命令
---

# `moltbot nodes`

管理已配对的 nodes(设备)并调用 node 功能。

相关:
- Nodes 概述: [Nodes](/nodes)
- 相机: [Camera nodes](/nodes/camera)
- 图像: [Image nodes](/nodes/images)

常用选项:
- `--url`、`--token`、`--timeout`、`--json`

## 常用命令

```bash
moltbot nodes list
moltbot nodes list --connected
moltbot nodes list --last-connected 24h
moltbot nodes pending
moltbot nodes approve <requestId>
moltbot nodes status
moltbot nodes status --connected
moltbot nodes status --last-connected 24h
```

`nodes list` 打印待处理/已配对的表格。已配对的行包括最近连接的年龄(Last Connect)。
使用 `--connected` 仅显示当前连接的 nodes。使用 `--last-connected <duration>` 过滤
在持续时间内连接的 nodes(例如 `24h`、`7d`)。

## 调用 / 运行

```bash
moltbot nodes invoke --node <id|name|ip> --command <command> --params <json>
moltbot nodes run --node <id|name|ip> <command...>
moltbot nodes run --raw "git status"
moltbot nodes run --agent main --node <id|name|ip> --raw "git status"
```

调用标志:
- `--params <json>`:JSON 对象字符串(默认 `{}`)。
- `--invoke-timeout <ms>`:node 调用超时(默认 `15000`)。
- `--idempotency-key <key>`:可选的幂等性键。

### 执行样式默认值

`nodes run` 镜像模型的执行行为(默认值 + 审批):

- 读取 `tools.exec.*`(加上 `agents.list[].tools.exec.*` 覆盖)。
- 在调用 `system.run` 之前使用执行审批(`exec.approval.request`)。
- 当设置了 `tools.exec.node` 时,可以省略 `--node`。
- 需要一个通告 `system.run` 的 node(macOS 伴随应用或无头 node 主机)。

标志:
- `--cwd <path>`:工作目录。
- `--env <key=val>`:环境覆盖(可重复)。
- `--command-timeout <ms>`:命令超时。
- `--invoke-timeout <ms>`:node 调用超时(默认 `30000`)。
- `--needs-screen-recording`:需要屏幕录制权限。
- `--raw <command>`:运行 shell 字符串(`/bin/sh -lc` 或 `cmd.exe /c`)。
- `--agent <id>`:agent 作用域的审批/允许列表(默认为配置的 agent)。
- `--ask <off|on-miss|always>`、`--security <deny|allowlist|full>`:覆盖。
