---
summary: "iOS 节点应用：连接到网关、配对、canvas 和故障排除"
read_when:
  - 配对或重新连接 iOS 节点
  - 从源代码运行 iOS 应用
  - 调试网关发现或 canvas 命令
---
# iOS 应用（节点）

可用性：内部预览。iOS 应用尚未公开发布。

## 它的作用

- 通过 WebSocket（LAN 或 tailnet）连接到网关。
- 暴露节点功能：Canvas、屏幕快照、相机捕获、位置、对话模式、语音唤醒。
- 接收 `node.invoke` 命令并报告节点状态事件。

## 要求

- 在另一台设备（macOS、Linux 或通过 WSL2 的 Windows）上运行的网关。
- 网络路径：
  - 同一 LAN 上的 Bonjour，**或**
  - 通过单播 DNS-SD 的 Tailnet（`moltbot.internal.`），**或**
  - 手动主机/端口（回退）。

## 快速入门（配对 + 连接）

1) 启动网关：

```bash
moltbot gateway --port 18789
```

2) 在 iOS 应用中，打开设置并选择已发现的网关（或启用手动主机并输入主机/端口）。

3) 在网关主机上批准配对请求：

```bash
moltbot nodes pending
moltbot nodes approve <requestId>
```

4) 验证连接：

```bash
moltbot nodes status
moltbot gateway call node.list --params "{}"
```

## 发现路径

### Bonjour (LAN)

网关在 `local.` 上宣传 `_moltbot._tcp`。iOS 应用自动列出这些。

### Tailnet（跨网络）

如果 mDNS 被阻止，请使用单播 DNS-SD 区域（推荐域名：`moltbot.internal.`）和 Tailscale 分离 DNS。
有关 CoreDNS 示例，请参阅 [Bonjour](/gateway/bonjour)。

### 手动主机/端口

在设置中，启用**手动主机**并输入网关主机 + 端口（默认 `18789`）。

## Canvas + A2UI

iOS 节点渲染 WKWebView canvas。使用 `node.invoke` 来驱动它：

```bash
moltbot nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18793/__moltbot__/canvas/"}'
```

注意：
- 网关 canvas 主机提供 `/__moltbot__/canvas/` 和 `/__moltbot__/a2ui/`。
- 当宣传 canvas 主机 URL 时，iOS 节点在首次连接时自动导航到 A2UI。
- 使用 `canvas.navigate` 和 `{"url":""}` 返回到内置脚手架。

### Canvas eval / 快照

```bash
moltbot nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__moltbot; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
moltbot nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## 语音唤醒 + 对话模式

- 语音唤醒和对话模式在设置中可用。
- iOS 可能会暂停后台音频；当应用未处于活动状态时，将语音功能视为尽力而为。

## 常见错误

- `NODE_BACKGROUND_UNAVAILABLE`：将 iOS 应用置于前台（canvas/相机/屏幕命令需要它）。
- `A2UI_HOST_NOT_CONFIGURED`：网关没有宣传 canvas 主机 URL；检查 [网关配置](/gateway/configuration) 中的 `canvasHost`。
- 配对提示从未出现：运行 `moltbot nodes pending` 并手动批准。
- 重新安装后重新连接失败：钥匙串配对令牌已清除；重新配对节点。

## 相关文档

- [配对](/gateway/pairing)
- [发现](/gateway/discovery)
- [Bonjour](/gateway/bonjour)
