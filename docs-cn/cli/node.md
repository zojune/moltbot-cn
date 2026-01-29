---
summary: "`moltbot node` CLI 参考(无头 node 主机)"
read_when:
  - 运行无头 node 主机
  - 为 system.run 配对非 macOS node
---

# `moltbot node`

运行**无头 node 主机**,连接到 Gateway WebSocket 并在此机器上暴露
`system.run` / `system.which`。

## 为什么使用 node 主机?

当您希望 agent **在其他机器上运行命令**而不在那里安装完整的 macOS 伴随应用时,请使用 node 主机。

常见用例:
- 在远程 Linux/Windows 机器上运行命令(构建服务器、实验室机器、NAS)。
- 在 gateway 上保持执行**沙盒化**,但将批准的运行委托给其他主机。
- 为自动化或 CI 节点提供轻量级、无头的执行目标。

执行仍由 node 主机上的**执行审批**和每个 agent 的允许列表保护,因此您可以保持命令访问作用域明确。

## 浏览器代理(零配置)

如果在 node 上未禁用 `browser.enabled`,node 主机会自动通告浏览器代理。这允许 agent 在该 node 上使用浏览器自动化,而无需额外配置。

如果需要,在 node 上禁用它:

```json5
{
  nodeHost: {
    browserProxy: {
      enabled: false
    }
  }
}
```

## 运行(前台)

```bash
moltbot node run --host <gateway-host> --port 18789
```

选项:
- `--host <host>`:Gateway WebSocket 主机(默认:`127.0.0.1`)
- `--port <port>`:Gateway WebSocket 端口(默认:`18789`)
- `--tls`:对 gateway 连接使用 TLS
- `--tls-fingerprint <sha256>`:预期的 TLS 证书指纹(sha256)
- `--node-id <id>`:覆盖 node id(清除配对令牌)
- `--display-name <name>`:覆盖 node 显示名称

## 服务(后台)

将无头 node 主机作为用户服务安装。

```bash
moltbot node install --host <gateway-host> --port 18789
```

选项:
- `--host <host>`:Gateway WebSocket 主机(默认:`127.0.0.1`)
- `--port <port>`:Gateway WebSocket 端口(默认:`18789`)
- `--tls`:对 gateway 连接使用 TLS
- `--tls-fingerprint <sha256>`:预期的 TLS 证书指纹(sha256)
- `--node-id <id>`:覆盖 node id(清除配对令牌)
- `--display-name <name>`:覆盖 node 显示名称
- `--runtime <runtime>`:服务运行时(`node` 或 `bun`)
- `--force`:如果已安装则重新安装/覆盖

管理服务:

```bash
moltbot node status
moltbot node stop
moltbot node restart
moltbot node uninstall
```

使用 `moltbot node run` 获取前台 node 主机(无服务)。

服务命令接受 `--json` 以获得机器可读的输出。

## 配对

第一次连接在 Gateway 上创建待处理的 node 配对请求。
通过以下方式批准:

```bash
moltbot nodes pending
moltbot nodes approve <requestId>
```

node 主机将其 node id、令牌、显示名称和 gateway 连接信息存储在
`~/.clawdbot/node.json` 中。

## 执行审批

`system.run` 由本地执行审批保护:

- `~/.clawdbot/exec-approvals.json`
- [Exec approvals](/tools/exec-approvals)
- `moltbot approvals --node <id|name|ip>`(从 Gateway 编辑)
