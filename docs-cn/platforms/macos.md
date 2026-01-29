---
summary: "Moltbot macOS 配套应用（菜单栏 + 网关代理）"
read_when:
  - 实现 macOS 应用功能
  - 更改 macOS 上的网关生命周期或节点桥接
---
# Moltbot macOS 配套（菜单栏 + 网关代理）

macOS 应用是 Moltbot 的**菜单栏配套**。它拥有权限，
在本地管理/附加到网关（launchd 或手动），并将 macOS
功能作为节点暴露给代理。

## 它的作用

- 在菜单栏中显示本机通知和状态。
- 拥有 TCC 提示（通知、辅助功能、屏幕录制、麦克风、
  语音识别、自动化/AppleScript）。
- 运行或连接到网关（本地或远程）。
- 暴露仅限 macOS 的工具（Canvas、相机、屏幕录制、`system.run`）。
- 在**远程**模式下启动本地节点主机服务（launchd），并在**本地**模式下停止它。
- 可选地托管**PeekabooBridge**用于 UI 自动化。
- 根据请求通过 npm/pnpm 安装全局 CLI（`moltbot`）（不建议将 bun 用于网关运行时）。

## 本地与远程模式

- **本地**（默认）：应用附加到正在运行的本地网关（如果存在）；
  否则它通过 `moltbot gateway install` 启用 launchd 服务。
- **远程**：应用通过 SSH/Tailscale 连接到网关，从不启动
  本地进程。
  应用启动本地**节点主机服务**，以便远程网关可以访问此 Mac。
应用不会将网关生成为子进程。

## Launchd 控制

应用管理每用户 LaunchAgent，标记为 `bot.molt.gateway`
（或使用 `--profile`/`CLAWDBOT_PROFILE` 时的 `bot.molt.<profile>`；传统 `com.clawdbot.*` 仍会卸载）。

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

运行命名配置文件时，将标签替换为 `bot.molt.<profile>`。

如果未安装 LaunchAgent，请从应用中启用它或运行
`moltbot gateway install`。

## 节点功能 (mac)

macOS 应用将自己展示为一个节点。常见命令：

- Canvas：`canvas.present`、`canvas.navigate`、`canvas.eval`、`canvas.snapshot`、`canvas.a2ui.*`
- 相机：`camera.snap`、`camera.clip`
- 屏幕：`screen.record`
- 系统：`system.run`、`system.notify`

节点报告 `permissions` 映射，以便代理可以决定允许什么。

节点服务 + 应用 IPC：
- 当无头节点主机服务运行时（远程模式），它作为节点连接到网关 WS。
- `system.run` 在 macOS 应用中执行（UI/TCC 上下文），通过本地 Unix 套接字；提示 + 输出保留在应用内。

图 (SCI)：
```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + TCC + system.run)
```

## Exec 批准 (system.run)

`system.run` 由 macOS 应用中的**Exec 批准**控制（设置 → Exec 批准）。
安全 + 询问 + 允许列表存储在 Mac 本地：

```
~/.clawdbot/exec-approvals.json
```

示例：

```json
{
  "version": 1,
  "defaults": {
    "security": "deny",
    "ask": "on-miss"
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        { "pattern": "/opt/homebrew/bin/rg" }
      ]
    }
  }
}
```

注意：
- `allowlist` 条目是已解析二进制路径的 glob 模式。
- 在提示中选择"始终允许"会将该命令添加到允许列表。
- `system.run` 环境覆盖被过滤（删除 `PATH`、`DYLD_*`、`LD_*`、`NODE_OPTIONS`、`PYTHON*`、`PERL*`、`RUBYOPT`），然后与应用的环境合并。

## 深度链接

应用注册 `moltbot://` URL 方案用于本地操作。

### `moltbot://agent`

触发网关 `agent` 请求。

```bash
open 'moltbot://agent?message=Hello%20from%20deep%20link'
```

查询参数：
- `message`（必需）
- `sessionKey`（可选）
- `thinking`（可选）
- `deliver` / `to` / `channel`（可选）
- `timeoutSeconds`（可选）
- `key`（可选无人值守模式密钥）

安全性：
- 没有 `key`，应用会提示确认。
- 使用有效的 `key`，运行是无人值守的（旨在用于个人自动化）。

## 入门流程（典型）

1) 安装并启动 **Moltbot.app**。
2) 完成权限清单（TCC 提示）。
3) 确保**本地**模式处于活动状态且网关正在运行。
4) 如果您需要终端访问，请安装 CLI。

## 构建和开发工作流（原生）

- `cd apps/macos && swift build`
- `swift run Moltbot`（或 Xcode）
- 打包应用：`scripts/package-mac-app.sh`

## 调试网关连接（macOS CLI）

使用调试 CLI 来练习相同的网关 WebSocket 握手和发现
逻辑，而无需启动应用。

```bash
cd apps/macos
swift run moltbot-mac connect --json
swift run moltbot-mac discover --timeout 3000 --json
```

连接选项：
- `--url <ws://host:port>`：覆盖配置
- `--mode <local|remote>`：从配置解析（默认：配置或本地）
- `--probe`：强制进行新的健康探测
- `--timeout <ms>`：请求超时（默认：`15000`）
- `--json`：用于比较的结构化输出

发现选项：
- `--include-local`：包括会被过滤为"本地"的网关
- `--timeout <ms>`：总体发现窗口（默认：`2000`）
- `--json`：用于比较的结构化输出

提示：与 `moltbot gateway discover --json` 进行比较，以查看
macOS 应用的发现管道（NWBrowser + tailnet DNS-SD 回退）是否与
Node CLI 的基于 `dns-sd` 的发现不同。

## 远程连接管道（SSH 隧道）

当 macOS 应用在**远程**模式下运行时，它会打开 SSH 隧道，以便本地 UI
组件可以与远程网关通信，就好像它在 localhost 上一样。

### 控制隧道（网关 WebSocket 端口）
- **目的：** 健康检查、状态、Web 聊天、配置和其他控制平面调用。
- **本地端口：** 网关端口（默认 `18789`），始终稳定。
- **远程端口：** 远程主机上的相同网关端口。
- **行为：** 没有随机本地端口；应用重用现有的健康隧道
  或在需要时重启它。
- **SSH 形状：** `ssh -N -L <local>:127.0.0.1:<remote>` 配合 BatchMode +
  ExitOnForwardFailure + keepalive 选项。
- **IP 报告：** SSH 隧道使用环回，因此网关将节点
  IP 视为 `127.0.0.1`。如果您希望出现真实的客户端
  IP，请使用**直接 (ws/wss)** 传输（请参阅 [macOS 远程访问](/platforms/mac/remote)）。

有关设置步骤，请参阅 [macOS 远程访问](/platforms/mac/remote)。有关协议
详细信息，请参阅 [网关协议](/gateway/protocol)。

## 相关文档

- [网关操作手册](/gateway)
- [网关 (macOS)](/platforms/mac/bundled-gateway)
- [macOS 权限](/platforms/mac/permissions)
- [Canvas](/platforms/mac/canvas)
