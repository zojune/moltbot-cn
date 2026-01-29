---
summary: "Moltbot Gateway CLI(`moltbot gateway`) — 运行、查询和发现 gateway"
read_when:
  - 从 CLI 运行 Gateway(开发或服务器)
  - 调试 Gateway 认证、绑定模式和连接性
  - 通过 Bonjour(LAN + tailnet)发现 gateway
---

# Gateway CLI

Gateway 是 Moltbot 的 WebSocket 服务器(频道、节点、会话、hooks)。

此页面中的子命令位于 `moltbot gateway …` 下。

相关文档:
- [/gateway/bonjour](/gateway/bonjour)
- [/gateway/discovery](/gateway/discovery)
- [/gateway/configuration](/gateway/configuration)

## 运行 Gateway

运行本地 Gateway 进程:

```bash
moltbot gateway
```

前台别名:

```bash
moltbot gateway run
```

注意事项:
- 默认情况下,除非在 `~/.clawdbot/moltbot.json` 中设置 `gateway.mode=local`,否则 Gateway 拒绝启动。使用 `--allow-unconfigured` 进行临时/开发运行。
- 在没有认证的情况下绑定到环回之外将被阻止(安全保护)。
- 当获得授权时,`SIGUSR1` 触发进程内重启(启用 `commands.restart` 或使用 gateway 工具/配置/应用/更新)。
- `SIGINT`/`SIGTERM` 处理程序停止 gateway 进程,但它们不会恢复任何自定义终端状态。如果您使用 TUI 或原始模式输入包装 CLI,请在退出之前恢复终端。

### 选项

- `--port <port>`: WebSocket 端口(默认来自配置/env;通常是 `18789`)。
- `--bind <loopback|lan|tailnet|auto|custom>`: 监听器绑定模式。
- `--auth <token|password>`: 认证模式覆盖。
- `--token <token>`: 令牌覆盖(也为进程设置 `CLAWDBOT_GATEWAY_TOKEN`)。
- `--password <password>`: 密码覆盖(也为进程设置 `CLAWDBOT_GATEWAY_PASSWORD`)。
- `--tailscale <off|serve|funnel>`: 通过 Tailscale 暴露 Gateway。
- `--tailscale-reset-on-exit`: 关闭时重置 Tailscale serve/funnel 配置。
- `--allow-unconfigured`: 允许在没有配置中的 `gateway.mode=local` 的情况下启动 gateway。
- `--dev`: 如果缺失,创建开发配置 + 工作区(跳过 BOOTSTRAP.md)。
- `--reset`: 重置开发配置 + 凭据 + 会话 + 工作区(需要 `--dev`)。
- `--force`: 在启动之前杀死选定端口上的任何现有监听器。
- `--verbose`: 详细日志。
- `--claude-cli-logs`: 仅在控制台中显示 claude-cli 日志(并启用其 stdout/stderr)。
- `--ws-log <auto|full|compact>`: websocket 日志样式(默认 `auto`)。
- `--compact`: `--ws-log compact` 的别名。
- `--raw-stream`: 将原始模型流事件记录到 jsonl。
- `--raw-stream-path <path>`: 原始流 jsonl 路径。

## 查询运行的 Gateway

所有查询命令都使用 WebSocket RPC。

输出模式:
- 默认:人类可读(TTY 中彩色)。
- `--json`: 机器可读的 JSON(无样式/微调器)。
- `--no-color`(或 `NO_COLOR=1`):禁用 ANSI,同时保持人类布局。

共享选项(在支持的地方):
- `--url <url>`:Gateway WebSocket URL。
- `--token <token>`:Gateway 令牌。
- `--password <password>`:Gateway 密码。
- `--timeout <ms>`:超时/预算(每个命令而异)。
- `--expect-final`:等待"最终"响应(agent 调用)。

### `gateway health`

```bash
moltbot gateway health --url ws://127.0.0.1:18789
```

### `gateway status`

`gateway status` 显示 Gateway 服务(launchd/systemd/schtasks)加上可选的 RPC 探测。

```bash
moltbot gateway status
moltbot gateway status --json
```

选项:
- `--url <url>`:覆盖探测 URL。
- `--token <token>`:探测的令牌认证。
- `--password <password>`:探测的密码认证。
- `--timeout <ms>`:探测超时(默认 `10000`)。
- `--no-probe`:跳过 RPC 探测(仅服务视图)。
- `--deep`:也扫描系统级服务。

### `gateway probe`

`gateway probe` 是"调试所有"命令。它始终探测:
- 您配置的远程 gateway(如果设置),以及
- localhost(环回)**即使配置了远程**。

如果可以到达多个 gateway,它将打印所有 gateway。当您使用隔离的配置文件/端口时(例如,救援 bot),支持多个 gateway,但大多数安装仍然运行单个 gateway。

```bash
moltbot gateway probe
moltbot gateway probe --json
```

#### 通过 SSH 远程(Mac 应用对等)

macOS 应用的"通过 SSH 远程"模式使用本地端口转发,因此远程 gateway(可能仅绑定到环回)可以在 `ws://127.0.0.1:<port>` 处访问。

CLI 等效项:

```bash
moltbot gateway probe --ssh user@gateway-host
```

选项:
- `--ssh <target>`:`user@host` 或 `user@host:port`(端口默认为 `22`)。
- `--ssh-identity <path>`:身份文件。
- `--ssh-auto`:选择第一个发现的 gateway 主机作为 SSH 目标(仅 LAN/WAB)。

配置(可选,用作默认值):
- `gateway.remote.sshTarget`
- `gateway.remote.sshIdentity`

### `gateway call <method>`

低级 RPC 助手。

```bash
moltbot gateway call status
moltbot gateway call logs.tail --params '{"sinceMs": 60000}'
```

## 管理 Gateway 服务

```bash
moltbot gateway install
moltbot gateway start
moltbot gateway stop
moltbot gateway restart
moltbot gateway uninstall
```

注意事项:
- `gateway install` 支持 `--port`、`--runtime`、`--token`、`--force`、`--json`。
- 生命周期命令接受 `--json` 用于脚本编写。

## 发现 gateway(Bonjour)

`gateway discover` 扫描 Gateway 信标(`_moltbot-gw._tcp`)。

- 多播 DNS-SD: `local.`
- 单播 DNS-SD(广域 Bonjour):`moltbot.internal.`(需要拆分 DNS + DNS 服务器;参见 [/gateway/bonjour](/gateway/bonjour))

只有启用 Bonjour 发现的 gateway(默认)才广播信标。

广域发现记录包括(TXT):
- `role`(gateway 角色提示)
- `transport`(传输提示,例如 `gateway`)
- `gatewayPort`(WebSocket 端口,通常是 `18789`)
- `sshPort`(SSH 端口;如果不存在则默认为 `22`)
- `tailnetDns`(MagicDNS 主机名,如果可用)
- `gatewayTls` / `gatewayTlsSha256`(启用 TLS + 证书指纹)
- `cliPath`(远程安装的可选提示)

### `gateway discover`

```bash
moltbot gateway discover
```

选项:
- `--timeout <ms>`:每个命令的超时(浏览/解析);默认 `2000`。
- `--json`:机器可读的输出(也禁用样式/微调器)。

示例:

```bash
moltbot gateway discover --timeout 4000
moltbot gateway discover --json | jq '.beacons[].wsUrl'
```
