---
summary: "macOS 上的网关运行时（外部 launchd 服务）"
read_when:
  - 打包 Moltbot.app
  - 调试 macOS 网关 launchd 服务
  - 为 macOS 安装网关 CLI
---

# macOS 上的网关（外部 launchd）

Moltbot.app 不再捆绑 Node/Bun 或网关运行时。macOS 应用
需要**外部** `moltbot` CLI 安装，不会将网关生成为
子进程，并管理每用户 launchd 服务以保持网关
运行（或附加到已经运行的本地网关（如果存在））。

## 安装 CLI（本地模式所需）

您需要在 Mac 上安装 Node 22+，然后全局安装 `moltbot`：

```bash
npm install -g moltbot@<version>
```

macOS 应用的**安装 CLI** 按钮通过 npm/pnpm 运行相同的流程（不建议将 bun 用于网关运行时）。

## Launchd（网关作为 LaunchAgent）

标签：
- `bot.molt.gateway`（或 `bot.molt.<profile>`；传统 `com.clawdbot.*` 可能保留）

Plist 位置（每用户）：
- `~/Library/LaunchAgents/bot.molt.gateway.plist`
  （或 `~/Library/LaunchAgents/bot.molt.<profile>.plist`）

管理器：
- macOS 应用在本地模式下拥有 LaunchAgent 安装/更新。
- CLI 也可以安装它：`moltbot gateway install`。

行为：
- "Moltbot 活动"启用/禁用 LaunchAgent。
- 应用退出**不会**停止网关（launchd 保持其活动）。
- 如果网关已在配置的端口上运行，应用将附加到
  它而不是启动新的网关。

日志记录：
- launchd stdout/err：`/tmp/moltbot/moltbot-gateway.log`

## 版本兼容性

macOS 应用检查网关版本与其自身版本。如果它们
不兼容，请更新全局 CLI 以匹配应用版本。

## 冒烟测试

```bash
moltbot --version

CLAWDBOT_SKIP_CHANNELS=1 \
CLAWDBOT_SKIP_CANVAS_HOST=1 \
moltbot gateway --port 18999 --bind loopback
```

然后：

```bash
moltbot gateway call health --url ws://127.0.0.1:18999 --timeout 3000
```
