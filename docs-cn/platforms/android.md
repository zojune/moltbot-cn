---
summary: "Android 应用（节点）：连接操作手册 + Canvas/聊天/相机"
read_when:
  - 配对或重新连接 Android 节点
  - 调试 Android 网关发现或认证
  - 验证跨客户端的聊天历史记录一致性
---

# Android 应用（节点）

## 支持快照
- 角色：配套节点应用（Android 不托管网关）。
- 需要网关：是（在 macOS、Linux 或通过 WSL2 的 Windows 上运行）。
- 安装：[入门指南](/start/getting-started) + [配对](/gateway/pairing)。
- 网关：[操作手册](/gateway) + [配置](/gateway/configuration)。
  - 协议：[网关协议](/gateway/protocol)（节点 + 控制平面）。

## 系统控制
系统控制（launchd/systemd）位于网关主机上。参见[网关](/gateway)。

## 连接操作手册

Android 节点应用 ⇄ (mDNS/NSD + WebSocket) ⇄ **网关**

Android 直接连接到网关 WebSocket（默认 `ws://<host>:18789`）并使用网关拥有的配对功能。

### 前提条件

- 您可以在"主控"机器上运行网关。
- Android 设备/模拟器可以访问网关 WebSocket：
  - 同一局域网上的 mDNS/NSD，**或**
  - 使用广域 Bonjour/单播 DNS-SD 的同一 Tailscale tailnet（见下文），**或**
  - 手动网关主机/端口（回退选项）
- 您可以在网关机器上运行 CLI（`moltbot`）（或通过 SSH）。

### 1) 启动网关

```bash
moltbot gateway --port 18789 --verbose
```

在日志中确认您看到类似内容：
- `listening on ws://0.0.0.0:18789`

对于仅 tailnet 设置（推荐用于维也纳 ⇄ 伦敦），将网关绑定到 tailnet IP：

- 在网关主机的 `~/.clawdbot/moltbot.json` 中设置 `gateway.bind: "tailnet"`。
- 重启网关 / macOS 菜单栏应用。

### 2) 验证发现（可选）

从网关机器：

```bash
dns-sd -B _moltbot-gw._tcp local.
```

更多调试说明：[Bonjour](/gateway/bonjour)。

#### 通过单播 DNS-SD 进行 Tailnet（维也纳 ⇄ 伦敦）发现

Android NSD/mDNS 发现无法跨网络工作。如果您的 Android 节点和网关位于不同网络但通过 Tailscale 连接，请改用广域 Bonjour/单播 DNS-SD：

1) 在网关主机上设置一个 DNS-SD 区域（例如 `moltbot.internal.`）并发布 `_moltbot-gw._tcp` 记录。
2) 为 `moltbot.internal` 配置指向该 DNS 服务器的 Tailscale 分离 DNS。

详细信息和 CoreDNS 配置示例：[Bonjour](/gateway/bonjour)。

### 3) 从 Android 连接

在 Android 应用中：

- 应用通过**前台服务**（持久通知）保持其网关连接活动。
- 打开**设置**。
- 在**已发现的网关**下，选择您的网关并点击**连接**。
- 如果 mDNS 被阻止，请使用**高级 → 手动网关**（主机 + 端口）并**连接（手动）**。

首次成功配对后，Android 在启动时会自动重新连接：
- 手动端点（如果已启用），否则
- 最后发现的网关（尽力而为）。

### 4) 批准配对（CLI）

在网关机器上：

```bash
moltbot nodes pending
moltbot nodes approve <requestId>
```

配对详情：[网关配对](/gateway/pairing)。

### 5) 验证节点已连接

- 通过节点状态：
  ```bash
  moltbot nodes status
  ```
- 通过网关：
  ```bash
  moltbot gateway call node.list --params "{}"
  ```

### 6) 聊天 + 历史

Android 节点的聊天表单使用网关的**主会话密钥**（`main`），因此历史记录和回复与 WebChat 及其他客户端共享：

- 历史：`chat.history`
- 发送：`chat.send`
- 推送更新（尽力而为）：`chat.subscribe` → `event:"chat"`

### 7) Canvas + 相机

#### 网关 Canvas 主机（推荐用于 Web 内容）

如果您希望节点显示代理可以在磁盘上编辑的真实 HTML/CSS/JS，请将节点指向网关 canvas 主机。

注意：节点使用 `canvasHost.port` 上的独立 canvas 主机（默认 `18793`）。

1) 在网关主机上创建 `~/clawd/canvas/index.html`。

2) 将节点导航到它（局域网）：

```bash
moltbot nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18793/__moltbot__/canvas/"}'
```

Tailnet（可选）：如果两台设备都在 Tailscale 上，请使用 MagicDNS 名称或 tailnet IP 代替 `.local`，例如 `http://<gateway-magicdns>:18793/__moltbot__/canvas/`。

此服务器将实时重新加载客户端注入到 HTML 中，并在文件更改时重新加载。
A2UI 主机位于 `http://<gateway-host>:18793/__moltbot__/a2ui/`。

Canvas 命令（仅前台）：
- `canvas.eval`、`canvas.snapshot`、`canvas.navigate`（使用 `{"url":""}` 或 `{"url":"/"}` 返回默认脚手架）。`canvas.snapshot` 返回 `{ format, base64 }`（默认 `format="jpeg"`）。
- A2UI：`canvas.a2ui.push`、`canvas.a2ui.reset`（`canvas.a2ui.pushJSONL` 遗留别名）

相机命令（仅前台；权限控制）：
- `camera.snap`（jpg）
- `camera.clip`（mp4）

有关参数和 CLI 助手，请参阅[相机节点](/nodes/camera)。
