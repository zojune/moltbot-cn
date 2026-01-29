---
summary: "macOS 应用流程，用于通过 SSH 控制远程 Moltbot 网关"
read_when:
  - 设置或调试远程 mac 控制
---
# 远程 Moltbot（macOS ⇄ 远程主机）


此流程使 macOS 应用能够充当在另一台主机（桌面/服务器）上运行的 Moltbot 网关的完整远程控制。这是应用的**通过 SSH 远程运行**功能。所有功能 — 健康检查、语音唤醒转发和 Web 聊天 — 重用来自 *设置 → 常规* 的相同远程 SSH 配置。

## 模式
- **本地（此 Mac）：** 一切都在笔记本电脑上运行。不涉及 SSH。
- **通过 SSH 远程（默认）：** Moltbot 命令在远程主机上执行。mac 应用使用 `-o BatchMode` 加上您选择的身份/密钥和本地端口转发打开 SSH 连接。
- **远程直接 (ws/wss)：** 没有 SSH 隧道。mac 应用直接连接到网关 URL（例如，通过 Tailscale Serve 或公共 HTTPS 反向代理）。

## 远程传输
远程模式支持两种传输：
- **SSH 隧道**（默认）：使用 `ssh -N -L ...` 将网关端口转发到 localhost。由于隧道是环回，网关将节点的 IP 视为 `127.0.0.1`。
- **直接 (ws/wss)：** 直接连接到网关 URL。网关看到真实的客户端 IP。

## 远程主机上的先决条件
1) 安装 Node + pnpm 并构建/安装 Moltbot CLI（`pnpm install && pnpm build && pnpm link --global`）。
2) 确保 `moltbot` 在非交互式 shell 的 PATH 上（如果需要，符号链接到 `/usr/local/bin` 或 `/opt/homebrew/bin`）。
3) 打开带有密钥认证的 SSH。我们推荐 **Tailscale** IP，以便在 LAN 外稳定可达。

## macOS 应用设置
1) 打开 *设置 → 常规*。
2) 在 **Moltbot 运行**下，选择**通过 SSH 远程**并设置：
   - **传输：** **SSH 隧道**或**直接 (ws/wss)**。
   - **SSH 目标：** `user@host`（可选 `:port`）。
     - 如果网关在同一 LAN 上并宣传 Bonjour，请从已发现的列表中选择它以自动填充此字段。
   - **网关 URL**（仅直接）：`wss://gateway.example.ts.net`（或 `ws://...` 用于本地/LAN）。
   - **身份文件**（高级）：您的密钥路径。
   - **项目根目录**（高级）：用于命令的远程签出路径。
   - **CLI 路径**（高级）：可运行的 `moltbot` 入口点/二进制文件的可选路径（在宣传时自动填充）。
3) 点击**测试远程**。成功表示远程 `moltbot status --json` 正确运行。失败通常意味着 PATH/CLI 问题；退出 127 表示远程找不到 CLI。
4) 健康检查和 Web 聊天现在将自动通过此 SSH 隧道运行。

## Web 聊天
- **SSH 隧道：** Web 聊天通过转发的 WebSocket 控制端口（默认 18789）连接到网关。
- **直接 (ws/wss)：** Web 聊天直接连接到配置的网关 URL。
- 不再有单独的 WebChat HTTP 服务器。

## 权限
- 远程主机需要与本地相同的 TCC 批准（自动化、辅助功能、屏幕录制、麦克风、语音识别、通知）。在该机器上运行入门以授予一次权限。
- 节点通过 `node.list` / `node.describe` 宣传其权限状态，以便代理知道可用的内容。

## 安全说明
- 在远程主机上优先使用环回绑定，并通过 SSH 或 Tailscale 连接。
- 如果您将网关绑定到非环回接口，则需要令牌/密码认证。
- 请参阅 [安全性](/gateway/security) 和 [Tailscale](/gateway/tailscale)。

## WhatsApp 登录流程（远程）
- **在远程主机上**运行 `moltbot channels login --verbose`。使用 WhatsApp 手机扫描 QR。
- 如果认证过期，请在该主机上重新运行登录。健康检查将显示链接问题。

## 故障排除
- **退出 127 / 未找到：** `moltbot` 不在非登录 shell 的 PATH 上。将其添加到 `/etc/paths`、您的 shell rc 或符号链接到 `/usr/local/bin`/`/opt/homebrew/bin`。
- **健康探测失败：** 检查 SSH 可达性、PATH，以及 Baileys 是否已登录（`moltbot status --json`）。
- **Web 聊天卡住：** 确认网关在远程主机上运行，并且转发的端口与网关 WS 端口匹配；UI 需要健康的 WS 连接。
- **节点 IP 显示 127.0.0.1：** SSH 隧道的预期行为。如果您希望网关看到真实的客户端 IP，请将**传输**切换到**直接 (ws/wss)**。
- **语音唤醒：** 触发短语在远程模式下自动转发；不需要单独的转发器。

## 通知声音
从带有 `moltbot` 和 `node.invoke` 的脚本中为每个通知选择声音，例如：

```bash
moltbot nodes notify --node <id> --title "Ping" --body "Remote gateway ready" --sound Glass
```

应用中不再有全局"默认声音"切换；调用者根据请求选择声音（或不选择声音）。
