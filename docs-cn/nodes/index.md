---
summary: "节点：配对、功能、权限以及画布/相机/屏幕/系统的 CLI 助手"
read_when:
  - 将 iOS/Android 节点配对到网关
  - 使用节点画布/相机作为代理上下文
  - 添加新的节点命令或 CLI 助手
---

# 节点

**节点**是连接到网关 **WebSocket**（与操作员相同的端口）并使用 `role: "node"` 的伴随设备（macOS/iOS/Android/无头设备），它通过 `node.invoke` 公开命令表面（例如 `canvas.*`、`camera.*`、`system.*`）。协议详情：[网关协议](/gateway/protocol)。

传统传输：[网桥协议](/gateway/bridge-protocol)（TCP JSONL；对于当前节点已弃用/移除）。

macOS 也可以运行在**节点模式**：菜单栏应用连接到网关的 WS 服务器，并将其本地画布/相机命令作为节点公开（因此 `moltbot nodes …` 适用于此 Mac）。

注意：
- 节点是**外设**，不是网关。它们不运行网关服务。
- Telegram/WhatsApp 等消息落在**网关**上，而不是节点上。

## 配对 + 状态

**WS 节点使用设备配对。** 节点在 `connect` 期间呈现设备身份；网关为 `role: node` 创建设备配对请求。通过设备 CLI（或 UI）批准。

快速 CLI：

```bash
moltbot devices list
moltbot devices approve <requestId>
moltbot devices reject <requestId>
moltbot nodes status
moltbot nodes describe --node <idOrNameOrIp>
```

注意：
- `nodes status` 在节点的设备配对角色包括 `node` 时将节点标记为**已配对**。
- `node.pair.*`（CLI：`moltbot nodes pending/approve/reject`）是一个单独的网关拥有的节点配对存储；它**不**限制 WS `connect` 握手。

## 远程节点主机（system.run）

当你的网关在一台机器上运行，而你希望在另一台机器上执行命令时，请使用**节点主机**。模型仍然与**网关**对话；当选择 `host=node` 时，网关将 `exec` 调用转发到**节点主机**。

### 什么在哪里运行
- **网关主机**：接收消息，运行模型，路由工具调用。
- **节点主机**：在节点机器上执行 `system.run`/`system.which`。
- **批准**：通过节点主机上的 `~/.clawdbot/exec-approvals.json` 强制执行。

### 启动节点主机（前台）

在节点机器上：

```bash
moltbot node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### 启动节点主机（服务）

```bash
moltbot node install --host <gateway-host> --port 18789 --display-name "Build Node"
moltbot node restart
```

### 配对 + 命名

在网关主机上：

```bash
moltbot nodes pending
moltbot nodes approve <requestId>
moltbot nodes list
```

命名选项：
- `moltbot node run` / `moltbot node install` 上的 `--display-name`（持久保存在节点上的 `~/.clawdbot/node.json` 中）。
- `moltbot nodes rename --node <id|name|ip> --name "Build Node"`（网关覆盖）。

### 允许列表命令

执行批准是**每个节点主机**的。从网关添加允许列表条目：

```bash
moltbot approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
moltbot approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

批准位于节点主机的 `~/.clawdbot/exec-approvals.json`。

### 将 exec 指向节点

配置默认值（网关配置）：

```bash
moltbot config set tools.exec.host node
moltbot config set tools.exec.security allowlist
moltbot config set tools.exec.node "<id-or-name>"
```

或每会话：

```
/exec host=node security=allowlist node=<id-or-name>
```

设置后，任何带有 `host=node` 的 `exec` 调用都在节点主机上运行（受节点允许列表/批准约束）。

相关：
- [节点主机 CLI](/cli/node)
- [Exec 工具](/tools/exec)
- [Exec 批准](/tools/exec-approvals)

## 调用命令

低级（原始 RPC）：

```bash
moltbot nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

存在用于常见的"为代理提供 MEDIA 附件"工作流的高级助手。

## 屏幕截图（画布快照）

如果节点显示画布（WebView），`canvas.snapshot` 返回 `{ format, base64 }`。

CLI 助手（写入临时文件并打印 `MEDIA:<path>`）：

```bash
moltbot nodes canvas snapshot --node <idOrNameOrIp> --format png
moltbot nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### 画布控制

```bash
moltbot nodes canvas present --node <idOrNameOrIp> --target https://example.com
moltbot nodes canvas hide --node <idOrNameOrIp>
moltbot nodes canvas navigate https://example.com --node <idOrNameOrIp>
moltbot nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

注意：
- `canvas present` 接受 URL 或本地文件路径（`--target`），以及用于定位的可选 `--x/--y/--width/--height`。
- `canvas eval` 接受内联 JS（`--js`）或位置参数。

### A2UI（画布）

```bash
moltbot nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
moltbot nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
moltbot nodes canvas a2ui reset --node <idOrNameOrIp>
```

注意：
- 仅支持 A2UI v0.8 JSONL（v0.9/createSurface 被拒绝）。

## 照片 + 视频（节点相机）

照片（`jpg`）：

```bash
moltbot nodes camera list --node <idOrNameOrIp>
moltbot nodes camera snap --node <idOrNameOrIp>            # 默认：两个朝向（2 个 MEDIA 输出）
moltbot nodes camera snap --node <idOrNameOrIp> --facing front
```

视频片段（`mp4`）：

```bash
moltbot nodes camera clip --node <idOrNameOrIp> --duration 10s
moltbot nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

注意：
- 节点必须处于**前台**才能执行 `canvas.*` 和 `camera.*`（后台调用返回 `NODE_BACKGROUND_UNAVAILABLE`）。
- 片段持续时间受到限制（目前 `<= 60s`）以避免过大的 base64 负载。
- Android 会在可能时提示 `CAMERA`/`RECORD_AUDIO` 权限；被拒绝的权限将以 `*_PERMISSION_REQUIRED` 失败。

## 屏幕录制（节点）

节点公开 `screen.record`（mp4）。示例：

```bash
moltbot nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
moltbot nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

注意：
- `screen.record` 需要节点应用处于前台。
- Android 在录制前将显示系统屏幕捕获提示。
- 屏幕录制限制为 `<= 60s`。
- `--no-audio` 禁用麦克风捕获（iOS/Android 支持；macOS 使用系统捕获音频）。
- 当有多个屏幕可用时，使用 `--screen <index>` 选择显示器。

## 位置（节点）

当在设置中启用位置时，节点公开 `location.get`。

CLI 助手：

```bash
moltbot nodes location get --node <idOrNameOrIp>
moltbot nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

注意：
- 位置**默认关闭**。
- "始终"需要系统权限；后台获取尽力而为。
- 响应包括经纬度、精度（米）和时间戳。

## SMS（Android 节点）

当用户授予 **SMS** 权限且设备支持电话功能时，Android 节点可以公开 `sms.send`。

低级调用：

```bash
moltbot nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from Moltbot"}'
```

注意：
- 在通告功能之前必须在 Android 设备上接受权限提示。
- 没有电话功能的仅 Wi-Fi 设备将不会通告 `sms.send`。

## 系统命令（节点主机 / mac 节点）

macOS 节点公开 `system.run`、`system.notify` 和 `system.execApprovals.get/set`。
无头节点主机公开 `system.run`、`system.which` 和 `system.execApprovals.get/set`。

示例：

```bash
moltbot nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
moltbot nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

注意：
- `system.run` 在负载中返回 stdout/stderr/退出代码。
- `system.notify` 遵守 macOS 应用上的通知权限状态。
- `system.run` 支持 `--cwd`、`--env KEY=VAL`、`--command-timeout` 和 `--needs-screen-recording`。
- `system.notify` 支持 `--priority <passive|active|timeSensitive>` 和 `--delivery <system|overlay|auto>`。
- macOS 节点丢弃 `PATH` 覆盖；无头节点主机仅在 `PATH` 前置节点主机 PATH 时才接受它。
- 在 macOS 节点模式下，`system.run` 受 macOS 应用中的执行批准限制（设置 → Exec 批准）。
  询问/允许列表/完整的行为与无头节点主机相同；被拒绝的提示返回 `SYSTEM_RUN_DENIED`。
- 在无头节点主机上，`system.run` 受执行批准（`~/.clawdbot/exec-approvals.json`）限制。

## Exec 节点绑定

当有多个节点可用时，你可以将 exec 绑定到特定节点。
这设置了 `exec host=node` 的默认节点（并且可以按代理覆盖）。

全局默认：

```bash
moltbot config set tools.exec.node "node-id-or-name"
```

每代理覆盖：

```bash
moltbot config get agents.list
moltbot config set agents.list[0].tools.exec.node "node-id-or-name"
```

取消设置以允许任何节点：

```bash
moltbot config unset tools.exec.node
moltbot config unset agents.list[0].tools.exec.node
```

## 权限映射

节点可能在 `node.list` / `node.describe` 中包含 `permissions` 映射，以权限名称为键（例如 `screenRecording`、`accessibility`），布尔值为值（`true` = 已授予）。

## 无头节点主机（跨平台）

Moltbot 可以运行**无头节点主机**（无 UI），连接到网关 WebSocket 并公开 `system.run` / `system.which`。这对于 Linux/Windows 或在服务器旁运行最小节点很有用。

启动它：

```bash
moltbot node run --host <gateway-host> --port 18789
```

注意：
- 仍需要配对（网关将显示节点批准提示）。
- 节点主机将其节点 ID、令牌、显示名称和网关连接信息存储在 `~/.clawdbot/node.json` 中。
- 执行批准通过 `~/.clawdbot/exec-approvals.json` 在本地强制执行
  （参见 [执行批准](/tools/exec-approvals)）。
- 在 macOS 上，无头节点主机在可达时优先使用伴侣应用 exec 主机，如果应用不可用则回退到本地执行。设置 `CLAWDBOT_NODE_EXEC_HOST=app` 以要求应用，或 `CLAWDBOT_NODE_EXEC_FALLBACK=0` 以禁用回退。
- 当网关 WS 使用 TLS 时添加 `--tls` / `--tls-fingerprint`。

## Mac 节点模式

- macOS 菜单栏应用作为节点连接到网关 WS 服务器（因此 `moltbot nodes …` 适用于此 Mac）。
- 在远程模式下，应用为网关端口打开 SSH 隧道并连接到 `localhost`。
