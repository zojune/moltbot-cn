---
summary: "Gateway Web surfaces: Control UI, bind modes, and 安全"
read_when: 
  - You want to access the Gateway over Tailscale
  - You want the browser Control UI and config editing
---
# Web (Gateway)

The Gateway serves a small **browser Control UI** (Vite + Lit) from the same 端口 as the Gateway WebSocket:

- default: `http://<host>:18789/`
- optional prefix: set `gateway.controlUi.basePath` (e.g. `/moltbot`)

Capabilities live in [Control UI](/Web/control-ui).
This page focuses on bind modes, 安全, and Web-facing surfaces.

## Webhook

When `hooks.enabled=true`, the Gateway also exposes a small Webhook 端点 on the same HTTP 服务器.
See [Gateway configuration](/gateway/configuration) → `hooks` for 认证 + payloads.

## 配置 (默认-on)

The Control UI is **enabled by default** when assets are present (`dist/control-ui`).
You can control it via 配置:

```json5
{
  gateway: {
    controlUi: { enabled: true, basePath: "/moltbot" } // basePath optional
  }
}
```

## Tailscale access

### Integrated Serve (recommended)

Keep the Gateway on loopback and let Tailscale Serve proxy it:

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "serve" }
  }
}
```

Then start the Gateway:

```bash
moltbot gateway
```

Open:
- `https://<magicdns>/` (or your configured `gateway.controlUi.basePath`)

### Tailnet bind + 令牌

```json5
{
  gateway: {
    bind: "tailnet",
    controlUi: { enabled: true },
    auth: { mode: "token", token: "your-token" }
  }
}
```

Then start the Gateway (令牌 必需 for non-loopback binds):

```bash
moltbot gateway
```

Open:
- `http://<tailscale-ip>:18789/` (or your configured `gateway.controlUi.basePath`)

### Public internet (Funnel)

```json5
{
  gateway: {
    bind: "loopback",
    tailscale: { mode: "funnel" },
    auth: { mode: "password" } // or CLAWDBOT_GATEWAY_PASSWORD
  }
}
```

## 安全 notes

- Gateway 认证 is 必需 默认情况下 (令牌/password or Tailscale identity headers).
- Non-loopback binds still **require** a shared token/password (`gateway.auth` or env).
- The wizard generates a Gateway 令牌 默认情况下 (even on loopback).
- The UI sends `connect.params.auth.token` or `connect.params.auth.password`.
- With Serve, Tailscale identity headers can satisfy 认证 when
  `gateway.auth.allowTailscale` is `true` (no 令牌/password 必需). Set
  `gateway.auth.allowTailscale: false` to require explicit 凭据. 参见
  [Tailscale](/Gateway/tailscale) and [安全](/Gateway/安全).
- `gateway.tailscale.mode: "funnel"` requires `gateway.auth.mode: "password"` (shared password).

## Building the UI

The Gateway serves static files from `dist/control-ui`. Build them with:

```bash
pnpm ui:build # auto-installs UI deps on first run
```
