---
summary: "Gateway 服务运行手册、生命周期和操作"
read_when:
  - 运行或调试 gateway 进程
---
# Gateway 服务运行手册

最后更新:2025-12-09

## 它是什么
- 拥有单个 Baileys/Telegram 连接和控制/事件平面的始终在线进程。
- 替代旧的 `gateway` 命令。CLI 入口点:`moltbot gateway`。
- 运行直到停止;在致命错误时以非零退出,因此主管会重新启动它。

## 如何运行(本地)
```bash
moltbot gateway --port 18789
# 用于 stdio 中的完整调试/跟踪日志:
moltbot gateway --port 18789 --verbose
# 如果端口繁忙,终止监听器然后启动:
moltbot gateway --force
# 开发循环(TS 更改时自动重新加载):
pnpm gateway:watch
```
- 配置热重新加载监视 `~/.clawdbot/moltbot.json`(或 `CLAWDBOT_CONFIG_PATH`)。
  - 默认模式:`gateway.reload.mode="hybrid"`(热应用安全更改,在关键更改时重新启动)。
  - 热重新加载在需要时通过 **SIGUSR1** 使用进程内重新启动。
  - 使用 `gateway.reload.mode="off"` 禁用。
- 将 WebSocket 控制平面绑定到 `127.0.0.1:<port>`(默认 18789)。
- 同一个端口还提供 HTTP(控制 UI、hooks、A2UI)。单端口多路复用。
  - OpenAI Chat Completions (HTTP):[`/v1/chat/completions`](/gateway/openai-http-api)。
  - OpenResponses (HTTP):[`/v1/responses`](/gateway/openresponses-http-api)。
  - Tools Invoke (HTTP):[`/tools/invoke`](/gateway/tools-invoke-http-api)。
- 默认情况下,在 `canvasHost.port`(默认 `18793`)上启动 Canvas 文件服务器,从 `~/clawd/canvas` 提供 `http://<gateway-host>:18793/__moltbot__/canvas/`。使用 `canvasHost.enabled=false` 或 `CLAWDBOT_SKIP_CANVAS_HOST=1` 禁用。
- 记录到 stdout;使用 launchd/systemd 保持其活动并轮换日志。
- 传递 `--verbose` 以在故障排除时将调试日志(握手、req/res、事件)从日志文件镜像到 stdio。
- `--force` 使用 `lsof` 查找所选端口上的监听器,发送 SIGTERM,记录它终止的内容,然后启动 gateway(如果缺少 `lsof`,则快速失败)。
- 如果您在主管(launchd/systemd/mac 应用子进程模式)下运行,停止/重新启动通常会发送 **SIGTERM**;旧构建可能会将此显示为 `pnpm` `ELIFECYCLE` 退出代码 **143** (SIGTERM),这是正常关闭,而不是崩溃。
- **SIGUSR1** 在授权时触发进程内重新启动(gateway tool/config apply/update,或为手动重新启动启用 `commands.restart`)。
- 默认情况下需要 Gateway 认证:设置 `gateway.auth.token`(或 `CLAWDBOT_GATEWAY_TOKEN`)或 `gateway.auth.password`。除非使用 Tailscale Serve 身份,否则客户端必须发送 `connect.params.auth.token/password`。
- 向导现在默认生成令牌,即使在环回上也是如此。
- 端口优先级:`--port` > `CLAWDBOT_GATEWAY_PORT` > `gateway.port` > 默认 `18789`。

## 远程访问
- 首选 Tailscale/VPN;否则 SSH 隧道:
  ```bash
  ssh -N -L 18789:127.0.0.1:18789 user@host
  ```
- 然后客户端通过隧道连接到 `ws://127.0.0.1:18789`。
- 如果配置了令牌,客户端即使通过隧道也必须在 `connect.params.auth.token` 中包含它。

## 多个 gateway(同一主机)

通常没有必要:一个 Gateway 可以服务多个消息频道和代理。仅为了冗余或严格隔离使用多个 Gateway(例如:救援机器人)。

如果您隔离状态 + 配置并使用唯一端口,则受支持。完整指南:[Multiple gateways](/gateway/multiple-gateways)。

服务名称具有配置文件感知能力:
- macOS:`bot.molt.<profile>`(旧的 `com.clawdbot.*` 可能仍然存在)
- Linux:`moltbot-gateway-<profile>.service`
- Windows:`Moltbot Gateway (<profile>)`

安装元数据嵌入在服务配置中:
- `CLAWDBOT_SERVICE_MARKER=moltbot`
- `CLAWDBOT_SERVICE_KIND=gateway`
- `CLAWDBOT_SERVICE_VERSION=<version>`

救援机器人模式:保持第二个 Gateway 隔离,具有自己的配置文件、状态目录、工作区和基本端口间隔。完整指南:[Rescue-bot guide](/gateway/multiple-gateways#rescue-bot-guide)。

### 开发配置文件(`--dev`)

快速路径:运行完全隔离的开发实例(配置/状态/工作区),而无需触及您的主要设置。

```bash
moltbot --dev setup
moltbot --dev gateway --allow-unconfigured
# 然后定位开发实例:
moltbot --dev status
moltbot --dev health
```

默认值(可以通过 env/flags/config 覆盖):
- `CLAWDBOT_STATE_DIR=~/.clawdbot-dev`
- `CLAWDBOT_CONFIG_PATH=~/.clawdbot-dev/moltbot.json`
- `CLAWDBOT_GATEWAY_PORT=19001`(Gateway WS + HTTP)
- 浏览器控制服务端口 = `19003`(派生:`gateway.port+2`,仅环回)
- `canvasHost.port=19005`(派生:`gateway.port+4`)
- 当您在 `--dev` 下运行 `setup`/`onboard` 时,`agents.defaults.workspace` 默认变为 `~/clawd-dev`。

派生端口(经验法则):
- 基本端口 = `gateway.port`(或 `CLAWDBOT_GATEWAY_PORT` / `--port`)
- 浏览器控制服务端口 = 基本 + 2(仅环回)
- `canvasHost.port = 基本 + 4`(或 `CLAWDBOT_CANVAS_HOST_PORT` / 配置覆盖)
- 浏览器配置文件 CDP 端口从 `browser.controlPort + 9 .. + 108` 自动分配(每个配置文件持久化)。

每个实例的检查清单:
- 唯一的 `gateway.port`
- 唯一的 `CLAWDBOT_CONFIG_PATH`
- 唯一的 `CLAWDBOT_STATE_DIR`
- 唯一的 `agents.defaults.workspace`
- 分开的 WhatsApp 号码(如果使用 WA)

每个配置文件的服务安装:
```bash
moltbot --profile main gateway install
moltbot --profile rescue gateway install
```

示例:
```bash
CLAWDBOT_CONFIG_PATH=~/.clawdbot/a.json CLAWDBOT_STATE_DIR=~/.clawdbot-a moltbot gateway --port 19001
CLAWDBOT_CONFIG_PATH=~/.clawdbot/b.json CLAWDBOT_STATE_DIR=~/.clawdbot-b moltbot gateway --port 19002
```

## 协议(操作员视图)
- 完整文档:[Gateway protocol](/gateway/protocol) 和 [Bridge protocol (legacy)](/gateway/bridge-protocol)。
- 客户端必需的第一帧:`req {type:"req", id, method:"connect", params:{minProtocol,maxProtocol,client:{id,displayName?,version,platform,deviceFamily?,modelIdentifier?,mode,instanceId?}, caps, auth?, locale?, userAgent? } }`。
- Gateway 回复 `res {type:"res", id, ok:true, payload:hello-ok }`(或带有错误的 `ok:false`,然后关闭)。
- 握手后:
  - 请求:`{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
  - 事件:`{type:"event", event, payload, seq?, stateVersion?}`
- 结构化在线条目:`{host, ip, version, platform?, deviceFamily?, modelIdentifier?, mode, lastInputSeconds?, ts, reason?, tags?[], instanceId? }`(对于 WS 客户端,`instanceId` 来自 `connect.client.instanceId`)。
- `agent` 响应是两个阶段的:第一个 `res` ack `{runId,status:"accepted"}`,然后在运行完成后的最终 `res` `{runId,status:"ok"|"error",summary}`;流式输出作为 `event:"agent"` 到达。

## 方法(初始集合)
- `health` — 完整的健康快照(与 `moltbot health --json` 形状相同)。
- `status` — 简短摘要。
- `system-presence` — 当前在线列表。
- `system-event` — 发布在线/系统注释(结构化)。
- `send` — 通过活动频道发送消息。
- `agent` — 运行代理轮次(在同一连接上回流事件)。
- `node.list` — 列出已配对 + 当前连接的节点(包括 `caps`、`deviceFamily`、`modelIdentifier`、`paired`、`connected` 和通告的 `commands`)。
- `node.describe` — 描述节点(功能 + 支持的 `node.invoke` 命令;适用于已配对的节点和当前连接的未配对节点)。
- `node.invoke` — 在节点上调用命令(例如 `canvas.*`、`camera.*`)。
- `node.pair.*` — 配对生命周期(`request`、`list`、`approve`、`reject`、`verify`)。

另请参阅:[Presence](/concepts/presence) 了解在线如何生成/去重以及稳定的 `client.instanceId` 为何重要。

## 事件
- `agent` — 来自代理运行的流式工具/输出事件(seq 标记)。
- `presence` — 在线更新(带有 stateVersion 的增量)推送到所有连接的客户端。
- `tick` — 定期保活/无操作以确认活跃度。
- `shutdown` — Gateway 正在退出;负载包括 `reason` 和可选的 `restartExpectedMs`。客户端应该重新连接。

## WebChat 集成
- WebChat 是一个原生 SwiftUI UI,直接与 Gateway WebSocket 通信以获取历史记录、发送、中止和事件。
- 远程使用通过相同的 SSH/Tailscale 隧道;如果配置了 gateway 令牌,客户端在 `connect` 期间包含它。
- macOS 应用通过单个 WS 连接(共享连接);它从初始快照中水合在线,并监听 `presence` 事件以更新 UI。

## 类型和验证
- 服务器使用 AJV 根据从协议定义发出的 JSON Schema 验证每个入站帧。
- 客户端(TS/Swift)消费生成的类型(TS 直接;Swift 通过仓库的生成器)。
- 协议定义是真实的来源;使用以下命令重新生成架构/模型:
  - `pnpm protocol:gen`
  - `pnpm protocol:gen:swift`

## 连接快照
- `hello-ok` 包括一个带有 `presence`、`health`、`stateVersion` 和 `uptimeMs` 加上 `policy {maxPayload,maxBufferedBytes,tickIntervalMs}` 的 `snapshot`,以便客户端可以立即呈现而无需额外请求。
- `health`/`system-presence` 仍然可用于手动刷新,但在连接时不是必需的。

## 错误代码(res.error 形状)
- 错误使用 `{ code, message, details?, retryable?, retryAfterMs? }`。
- 标准代码:
  - `NOT_LINKED` — WhatsApp 未通过身份验证。
  - `AGENT_TIMEOUT` — 代理在配置的截止时间内未响应。
  - `INVALID_REQUEST` — 架构/参数验证失败。
  - `UNAVAILABLE` — Gateway 正在关闭或依赖项不可用。

## 保活行为
- `tick` 事件(或 WS ping/pong)定期发出,以便即使没有流量发生,客户端也知道 Gateway 是活着的。
- 发送/代理确认仍然是单独的响应;不要使发送的 tick 过载。

## 重放/间隙
- 事件不会重放。客户端检测 seq 间隙,应该在继续之前刷新(`health` + `system-presence`)。WebChat 和 macOS 客户端现在在间隙时自动刷新。

## 监督(macOS 示例)
- 使用 launchd 保持服务活动:
  - 程序:`moltbot` 的路径
  - 参数:`gateway`
  - KeepAlive:true
  - StandardOut/Err:文件路径或 `syslog`
- 失败时,launchd 重新启动;致命的错误配置应该保持退出,以便操作员注意到。
- LaunchAgents 是每用户的,需要登录的会话;对于无头设置,使用自定义 LaunchDaemon(未附带)。
  - `moltbot gateway install` 写入 `~/Library/LaunchAgents/bot.molt.gateway.plist`
    (或 `bot.molt.<profile>.plist`;旧的 `com.clawdbot.*` 被清理)。
  - `moltbot doctor` 审计 LaunchAgent 配置,可以将其更新到当前默认值。

## Gateway 服务管理(CLI)

使用 Gateway CLI 进行安装/启动/停止/重新启动/状态:

```bash
moltbot gateway status
moltbot gateway install
moltbot gateway stop
moltbot gateway restart
moltbot logs --follow
```

注意事项:
- `gateway status` 默认通过服务解析的端口/配置探测 Gateway RPC(使用 `--url` 覆盖)。
- `gateway status --deep` 添加系统级扫描(LaunchDaemons/system 单元)。
- `gateway status --no-probe` 跳过 RPC 探测(当网络关闭时很有用)。
- `gateway status --json` 对脚本稳定。
- `gateway status` 分别报告**主管运行时**(launchd/systemd 运行)和 **RPC 可达性**(WS 连接 + 状态 RPC)。
- `gateway status` 打印配置路径 + 探测目标以避免"localhost 与 LAN 绑定"混淆和配置文件不匹配。
- `gateway status` 包括最后的 gateway 错误行,当服务看起来正在运行但端口关闭时。
- `logs` 通过 RPC 尾随 Gateway 文件日志(无需手动 `tail`/`grep`)。
- 如果检测到其他类似 gateway 的服务,CLI 会警告,除非它们是 Moltbot 配置文件服务。
  我们仍然建议大多数设置**每台机器一个 gateway**;使用隔离的配置文件/端口进行冗余或救援机器人。参见 [Multiple gateways](/gateway/multiple-gateways)。
  - 清理:`moltbot gateway uninstall`(当前服务)和 `moltbot doctor`(旧迁移)。
- `gateway install` 在已安装时是无操作;使用 `moltbot gateway install --force` 重新安装(配置文件/env/路径更改)。

捆绑的 mac 应用:
- Moltbot.app 可以捆绑基于 Node 的 gateway 中继,并安装标记为
  `bot.molt.gateway`(或 `bot.molt.<profile>`;旧的 `com.clawdbot.*` 标签仍然干净地卸载)的每用户 LaunchAgent。
- 要干净地停止它,请使用 `moltbot gateway stop`(或 `launchctl bootout gui/$UID/bot.molt.gateway`)。
- 要重新启动,请使用 `moltbot gateway restart`(或 `launchctl kickstart -k gui/$UID/bot.molt.gateway`)。
  - `launchctl` 仅在安装了 LaunchAgent 时有效;否则首先使用 `moltbot gateway install`。
  - 在运行命名配置文件时,将标签替换为 `bot.molt.<profile>`。

## 监督(systemd 用户单元)
Moltbot 在 Linux/WSL2 上默认安装 **systemd 用户服务**。我们
为单用户机器推荐用户服务(更简单的 env、每用户配置)。
为多用户或始终在线的服务器使用 **system 服务**(不需要 lingering,
共享监督)。

`moltbot gateway install` 写入用户单元。`moltbot doctor` 审计
单元,可以将其更新以匹配当前推荐的默认值。

创建 `~/.config/systemd/user/moltbot-gateway[-<profile>].service`:
```
[Unit]
Description=Moltbot Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/moltbot gateway --port 18789
Restart=always
RestartSec=5
Environment=CLAWDBOT_GATEWAY_TOKEN=
WorkingDirectory=/home/youruser

[Install]
WantedBy=default.target
```
启用 lingering(必需,以便用户服务在注销/空闲后继续运行):
```
sudo loginctl enable-linger youruser
```
入职在 Linux/WSL2 上运行此操作(可能提示 sudo;写入 `/var/lib/systemd/linger`)。
然后启用服务:
```
systemctl --user enable --now moltbot-gateway[-<profile>].service
```

**替代方案(system 服务)** - 对于始终在线或多用户服务器,您可以
安装 systemd **system** 单元而不是用户单元(不需要 lingering)。
创建 `/etc/systemd/system/moltbot-gateway[-<profile>].service`(复制上面的单元,
切换 `WantedBy=multi-user.target`,设置 `User=` + `WorkingDirectory=`),然后:
```
sudo systemctl daemon-reload
sudo systemctl enable --now moltbot-gateway[-<profile>].service
```

## Windows (WSL2)

Windows 安装应使用 **WSL2** 并遵循上面的 Linux systemd 部分。

## 操作检查
- 活跃度:打开 WS 并发送 `req:connect` → 期望 `res` 带有 `payload.type="hello-ok"`(带有快照)。
- 就绪:调用 `health` → 期望 `ok: true` 和 `linkChannel` 中的已链接频道(如适用)。
- 调试:订阅 `tick` 和 `presence` 事件;确保 `status` 显示已链接/认证时间;在线条目显示 Gateway 主机和连接的客户端。

## 安全保证
- 默认情况下假设每个主机一个 Gateway;如果您运行多个配置文件,请隔离端口/状态并定位正确的实例。
- 不会回退到直接 Baileys 连接;如果 Gateway 宕机,发送将快速失败。
- 非连接的第一帧或格式错误的 JSON 被拒绝,套接字被关闭。
- 优雅关闭:在关闭之前发出 `shutdown` 事件;客户端必须处理关闭 + 重新连接。

## CLI 助手
- `moltbot gateway health|status` — 通过 Gateway WS 请求健康/状态。
- `moltbot message send --target <num> --message "hi" [--media ...]` — 通过 Gateway 发送(对于 WhatsApp 幂等)。
- `moltbot agent --message "hi" --to <num>` — 运行代理轮次(默认等待最终结果)。
- `moltbot gateway call <method> --params '{"k":"v"}'` — 用于调试的原始方法调用器。
- `moltbot gateway stop|restart` — 停止/重新启动受监督的 gateway 服务(launchd/systemd)。
- Gateway 助手子命令假设 `--url` 上运行的 gateway;它们不再自动生成一个。

## 迁移指南
- 停止使用 `moltbot gateway` 和旧的 TCP 控制端口。
- 更新客户端以使用 WS 协议进行强制性连接和结构化在线。
