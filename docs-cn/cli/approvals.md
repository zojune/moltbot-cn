---
summary: "`moltbot approvals` CLI 参考(gateway 或 node 主机的执行审批)"
read_when:
  - 您想从 CLI 编辑执行审批
  - 您需要管理 gateway 或 node 主机上的允许列表
---

# `moltbot approvals`

为**本地主机**、**gateway 主机**或**node 主机**管理执行审批。
默认情况下,命令针对磁盘上的本地审批文件。使用 `--gateway` 定位 gateway,或使用 `--node` 定位特定的 node。

相关:
- 执行审批: [Exec approvals](/tools/exec-approvals)
- Nodes: [Nodes](/nodes)

## 常用命令

```bash
moltbot approvals get
moltbot approvals get --node <id|name|ip>
moltbot approvals get --gateway
```

## 从文件替换审批

```bash
moltbot approvals set --file ./exec-approvals.json
moltbot approvals set --node <id|name|ip> --file ./exec-approvals.json
moltbot approvals set --gateway --file ./exec-approvals.json
```

## 允许列表助手

```bash
moltbot approvals allowlist add "~/Projects/**/bin/rg"
moltbot approvals allowlist add --agent main --node <id|name|ip> "/usr/bin/uptime"
moltbot approvals allowlist add --agent "*" "/usr/bin/uname"

moltbot approvals allowlist remove "~/Projects/**/bin/rg"
```

## 注意事项

- `--node` 使用与 `moltbot nodes` 相同的解析器(id、name、ip 或 id 前缀)。
- `--agent` 默认为 `"*"`,适用于所有 agent。
- node 主机必须通告 `system.execApprovals.get/set`(macOS 应用或无头 node 主机)。
- 审批文件按主机存储在 `~/.clawdbot/exec-approvals.json`。
