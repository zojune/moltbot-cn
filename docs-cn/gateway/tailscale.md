---
summary: "Gateway 仪表板的集成 Tailscale Serve/Funnel"
read_when:
  - 在 localhost 之外暴露 Gateway Control UI
  - 自动化 tailnet 或公共仪表板访问
---
# Tailscale（Gateway 仪表板）

Moltbot 可以自动配置 Tailscale **Serve**（tailnet）或 **Funnel**（公共）用于
Gateway 仪表板和 WebSocket 端口。这使 Gateway 保持绑定到 loopback，而
Tailscale 提供 HTTPS、路由和（对于 Serve）身份标头。

## 模式

- `serve`：通过 `tailscale serve` 仅限 tailnet。gateway 保持在 `127.0.0.1`。
- `funnel`：通过 `tailscale funnel` 进行公共 HTTPS。Moltbot 需要共享密码。
- `off`：默认（无 Tailscale 自动化）。

## 认证

设置 `gateway.auth.mode` 来控制握手：

- `token`（当设置 `CLAWDBOT_GATEWAY_TOKEN` 时的默认值）
- `password`（通过 `CLAWDBOT_GATEWAY_PASSWORD` 或配置共享密钥）

当 `tailscale.mode = "serve"` 并且 `gateway.auth.allowTailscale` 为 `true` 时，
有效的 Serve 代理请求可以通过 Tailscale 身份标头
（`tailscale-user-login`）进行认证，而无需提供令牌/密码。Moltbot 通过本地 Tailscale
守护进程（`tailscale whois`）解析 `x-forwarded-for` 地址并将其与标头匹配来验证身份，
然后接受它。Moltbot 只有当请求从 loopback 到达并带有
Tailscale 的 `x-forwarded-for`、`x-forwarded-proto` 和 `x-forwarded-host`
标头时，才将其视为 Serve 请求。
要要求显式凭据，请设置 `gateway.auth.allowTailscale: false` 或
强制 `gateway.auth.mode: "password"`。

## 配置示例

### 仅限 tailnet（Serve）

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

打开：`https://<magicdns>/`（或您配置的 `gateway.controlUi.basePath`）

### 仅限 tailnet（绑定到 Tailnet IP）

当您希望 Gateway 直接监听 Tailnet IP（无 Serve/Funnel）时使用此选项。

```json5
{
  gateway: {
    bind: "tailnet",
    auth: { mode: "token", token: "your-token" }
  }
}
```

从另一个 Tailnet 设备连接：
- Control UI：`http://<tailscale-ip>:18789/`
- WebSocket：`ws://<tailscale-ip>:18789`

注意：loopback（`http://127.0.0.1:18789`）在此模式下**不会**工作。

### 公共互联网（Funnel + 共享密码）

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password", password: "replace-me" }
  }
}
```

比起将密码提交到磁盘，更喜欢使用 `CLAWDBOT_GATEWAY_PASSWORD`。

## CLI 示例

```bash
moltbot gateway --tailscale serve
moltbot gateway --tailscale funnel --auth password
```

## 注意事项

- Tailscale Serve/Funnel 需要安装并登录 `tailscale` CLI。
- `tailscale.mode: "funnel"` 除非认证模式是 `password`，否则拒绝启动以避免公共暴露。
- 如果您希望 Moltbot 在关闭时撤消 `tailscale serve` 或 `tailscale funnel` 配置，请设置 `gateway.tailscale.resetOnExit`。
- `gateway.bind: "tailnet"` 是直接的 Tailnet 绑定（无 HTTPS、无 Serve/Funnel）。
- `gateway.bind: "auto"` 优先 loopback；如果您只想使用 Tailnet，请使用 `tailnet`。
- Serve/Funnel 仅暴露 **Gateway control UI + WS**。节点通过相同的 Gateway WS 端点连接，因此 Serve 可以用于节点访问。

## 浏览器控制（远程 Gateway + 本地浏览器）

如果您在一台机器上运行 Gateway，但想在另一台机器上驱动浏览器，
请在浏览器机器上运行**节点主机**，并将两者保持在同一个 tailnet 上。
Gateway 将代理浏览器操作到节点；不需要单独的控制服务器或 Serve URL。

避免对浏览器控制使用 Funnel；像对待操作员访问一样对待节点配对。

## Tailscale 先决条件 + 限制

- Serve 需要为您的 tailnet 启用 HTTPS；如果缺少，CLI 会提示。
- Serve 注入 Tailscale 身份标头；Funnel 不注入。
- Funnel 需要 Tailscale v1.38.3+、MagicDNS、启用 HTTPS 和 funnel 节点属性。
- Funnel 仅支持通过 TLS 的端口 `443`、`8443` 和 `10000`。
- macOS 上的 Funnel 需要开源 Tailscale 应用变体。

## 了解更多

- Tailscale Serve 概述：https://tailscale.com/kb/1312/serve
- `tailscale serve` 命令：https://tailscale.com/kb/1242/tailscale-serve
- Tailscale Funnel 概述：https://tailscale.com/kb/1223/tailscale-funnel
- `tailscale funnel` 命令：https://tailscale.com/kb/1311/tailscale-funnel
