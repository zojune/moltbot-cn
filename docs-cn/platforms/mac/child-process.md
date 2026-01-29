---
summary: "macOS 上的网关生命周期（launchd）"
read_when:
  - 将 mac 应用与网关生命周期集成
---
# macOS 上的网关生命周期

macOS 应用**通过 launchd 管理网关**，默认情况下不会将
网关生成为子进程。它首先尝试附加到已
在配置端口上运行的网关；如果无法访问，它通过外部 `moltbot` CLI 启用 launchd
服务（没有嵌入式运行时）。这为您提供
可靠的登录时自动启动和崩溃时重启。

子进程模式（网关由应用直接生成）今天**未使用**。
如果您需要与 UI 更紧密的耦合，请在终端中手动运行网关。

## 默认行为（launchd）

- 应用安装每用户 LaunchAgent，标记为 `bot.molt.gateway`
  （使用 `--profile`/`CLAWDBOT_PROFILE` 时的 `bot.molt.<profile>`；支持传统 `com.clawdbot.*`）。
- 启用本地模式时，应用确保 LaunchAgent 已加载并在需要时启动网关。
- 日志写入到 launchd 网关日志路径（在调试设置中可见）。

常见命令：

```bash
launchctl kickstart -k gui/$UID/bot.molt.gateway
launchctl bootout gui/$UID/bot.molt.gateway
```

运行命名配置文件时，将标签替换为 `bot.molt.<profile>`。

## 未签名的开发构建

`scripts/restart-mac.sh --no-sign` 适用于您没有
签名密钥时的快速本地构建。为了防止 launchd 指向未签名的中继二进制文件，它会：
- 写入 `~/.clawdbot/disable-launchagent`。

`scripts/restart-mac.sh` 的签名运行会在标记存在时清除此覆盖。要手动重置：

```bash
rm ~/.clawdbot/disable-launchagent
```

## 仅附加模式

要强制 macOS 应用**永不安装或管理 launchd**，请使用
`--attach-only`（或 `--no-launchd`）启动它。这会设置 `~/.clawdbot/disable-launchagent`，
因此应用仅附加到已经运行的网关。您可以在调试设置中切换相同的行为。

## 远程模式

远程模式从不启动本地网关。应用使用 SSH 隧道到
远程主机并通过该隧道连接。

## 我们更喜欢 launchd 的原因

- 登录时自动启动。
- 内置重启/KeepAlive 语义。
- 可预测的日志和监督。

如果再次需要真正的子进程模式，应将其记录为单独的、明确的仅开发模式。
