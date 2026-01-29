---
summary: "Bonjour/mDNS 发现 + 调试(Gateway 信标、客户端和常见故障模式)"
read_when:
  - 在 macOS/iOS 上调试 Bonjour 发现问题
  - 更改 mDNS 服务类型、TXT 记录或发现 UX
---
# Bonjour / mDNS 发现

Moltbot 使用 Bonjour(mDNS / DNS-SD)作为**仅限 LAN 的便利**来发现
活动的 Gateway(WebSocket 端点)。它是尽力而为的,并**不**替代 SSH 或
基于 Tailnet 的连接。

## 通过 Tailscale 的广域 Bonjour(单播 DNS-SD)

如果节点和 gateway 在不同的网络上,多播 mDNS 将不会跨越
边界。您可以通过切换到通过 Tailscale 的**单播 DNS-SD**
("广域 Bonjour")来保持相同的发现 UX。

高级步骤:

1) 在 gateway host 上运行 DNS 服务器(可通过 Tailnet 访问)。
2) 在专用区域下为 `_moltbot-gw._tcp` 发布 DNS-SD 记录
   (示例: `moltbot.internal.`)。
3) 配置 Tailscale **分离 DNS**,以便 `moltbot.internal` 通过该
   DNS 服务器为客户端(包括 iOS)解析。

Moltbot 在此模式下标准化为 `moltbot.internal.`。iOS/Android 节点
自动浏览 `local.` 和 `moltbot.internal.`。

### Gateway 配置(推荐)

```json5
{
  gateway: { bind: "tailnet" }, // 仅 tailnet(推荐)
  discovery: { wideArea: { enabled: true } } // 启用 moltbot.internal DNS-SD 发布
}
```

### 一次性 DNS 服务器设置(gateway host)

```bash
moltbot dns setup --apply
```

这将安装 CoreDNS 并将其配置为:
- 仅在 gateway 的 Tailscale 接口上监听端口 53
- 从 `~/.clawdbot/dns/moltbot.internal.db` 提供 `moltbot.internal.`

从连接到 tailnet 的机器验证:

```bash
dns-sd -B _moltbot-gw._tcp moltbot.internal.
dig @<TAILNET_IPV4> -p 53 _moltbot-gw._tcp.clawdbot.internal PTR +short
```

### Tailscale DNS 设置

在 Tailscale 管理控制台中:

- 添加指向 gateway tailnet IP 的名称服务器(UDP/TCP 53)。
- 添加分离 DNS,以便域 `moltbot.internal` 使用该名称服务器。

一旦客户端接受 tailnet DNS,iOS 节点就可以浏览
`moltbot.internal.` 中的 `_moltbot-gw._tcp`,
无需多播。

### Gateway 监听器安全(推荐)

Gateway WS 端口(默认 `18789`)默认绑定到环回。对于 LAN/tailnet
访问,显式绑定并保持启用认证。

对于仅 tailnet 设置:
- 在 `~/.clawdbot/moltbot.json` 中设置 `gateway.bind: "tailnet"`。
- 重启 Gateway(或重启 macOS 菜单栏应用)。

## 通告内容

只有 Gateway 通告 `_moltbot-gw._tcp`。

## 服务类型

- `_moltbot-gw._tcp` — gateway 传输信标(由 macOS/iOS/Android 节点使用)。

## TXT 键(非机密提示)

Gateway 通告小的非机密提示以使 UI 流程方便:

- `role=gateway`
- `displayName=<友好名称>`
- `lanHost=<hostname>.local`
- `gatewayPort=<port>`(Gateway WS + HTTP)
- `gatewayTls=1`(仅当启用 TLS 时)
- `gatewayTlsSha256=<sha256>`(仅当启用 TLS 且指纹可用时)
- `canvasPort=<port>`(仅当启用 canvas host 时;默认 `18793`)
- `sshPort=<port>`(未覆盖时默认为 22)
- `transport=gateway`
- `cliPath=<path>`(可选;可运行 `moltbot` 入口点的绝对路径)
- `tailnetDns=<magicdns>`(Tailnet 可用时的可选提示)

## 在 macOS 上调试

有用的内置工具:

- 浏览实例:
  ```bash
  dns-sd -B _moltbot-gw._tcp local.
  ```
- 解析一个实例(替换 `<instance>`):
  ```bash
  dns-sd -L "<instance>" _moltbot-gw._tcp local.
  ```

如果浏览有效但解析失败,您通常会遇到 LAN 策略或
mDNS 解析器问题。

## 在 Gateway 日志中调试

Gateway 写入滚动日志文件(在启动时打印为
`gateway log file: ...`)。查找 `bonjour:` 行,特别是:

- `bonjour: advertise failed ...`
- `bonjour: ... name conflict resolved` / `hostname conflict resolved`
- `bonjour: watchdog detected non-announced service ...`

## 在 iOS 节点上调试

iOS 节点使用 `NWBrowser` 发现 `_moltbot-gw._tcp`。

要捕获日志:
- 设置 → Gateway → 高级 → **发现调试日志**
- 设置 → Gateway → 高级 → **发现日志** → 重现 → **复制**

日志包括浏览器状态转换和结果集更改。

## 常见故障模式

- **Bonjour 不跨网络**:使用 Tailnet 或 SSH。
- **多播被阻止**:某些 Wi-Fi 网络禁用 mDNS。
- **睡眠/接口混乱**:macOS 可能会暂时丢失 mDNS 结果;请重试。
- **浏览有效但解析失败**:保持机器名称简单(避免表情符号或
  标点符号),然后重启 Gateway。服务实例名称来自
  主机名,因此过于复杂的名称可能会使某些解析器感到困惑。

## 转义的实例名称(`\032`)

Bonjour/DNS-SD 通常将服务实例名称中的字节转义为十进制 `\DDD`
序列(例如,空格变为 `\032`)。

- 这在协议级别是正常的。
- UI 应该解码以显示(iOS 使用 `BonjourEscapes.decode`)。

## 禁用/配置

- `CLAWDBOT_DISABLE_BONJOUR=1` 禁用通告。
- `~/.clawdbot/moltbot.json` 中的 `gateway.bind` 控制 Gateway 绑定模式。
- `CLAWDBOT_SSH_PORT` 覆盖 TXT 中通告的 SSH 端口。
- `CLAWDBOT_TAILNET_DNS` 在 TXT 中发布 MagicDNS 提示。
- `CLAWDBOT_CLI_PATH` 覆盖通告的 CLI 路径。

## 相关文档

- 发现策略和传输选择: [Discovery](/gateway/discovery)
- 节点配对 + 批准: [Gateway pairing](/gateway/pairing)
