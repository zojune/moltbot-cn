---
summary: "Browser-based control UI for the Gateway (chat, 节点, 配置)"
read_when: 
  - You want to operate the Gateway from a browser
  - You want Tailnet access without SSH tunnels
---
# Control UI (browser)

The Control UI is a small **Vite + Lit** single-page 应用 served by the Gateway:

- default: `http://<host>:18789/`
- optional prefix: set `gateway.controlUi.basePath` (e.g. `/moltbot`)

It speaks **directly to the Gateway WebSocket** on the same 端口.

## Quick open (local)

If the Gateway is running on the same computer, open:

- http://127.0.0.1:18789/ (or http://localhost:18789/)

If the page fails to load, start the Gateway first: `moltbot gateway`.

认证 is supplied during the WebSocket handshake via:
- `connect.params.auth.token`
- `connect.params.auth.password`
The dashboard 设置 panel lets you store a 令牌; passwords are not persisted.
The onboarding wizard generates a Gateway 令牌 默认情况下, so paste it here on first connect.

## What it can do (today)
- Chat with the model via Gateway WS (`chat.history`, `chat.send`, `chat.abort`, `chat.inject`)
- 流 工具 calls + live 工具 输出 cards in Chat (代理 事件)
- Channels: WhatsApp/Telegram/Discord/Slack + plugin channels (Mattermost, etc.) status + QR login + per-channel config (`channels.status`, `web.login.*`, `config.patch`)
- Instances: presence list + refresh (`system-presence`)
- Sessions: list + per-session thinking/verbose overrides (`sessions.list`, `sessions.patch`)
- Cron jobs: list/add/run/enable/disable + run history (`cron.*`)
- Skills: status, enable/disable, install, API key updates (`skills.*`)
- Nodes: list + caps (`node.list`)
- Exec approvals: edit gateway or node allowlists + ask policy for `exec host=gateway/node` (`exec.approvals.*`)
- Config: view/edit `~/.clawdbot/moltbot.json` (`config.get`, `config.set`)
- Config: apply + restart with validation (`config.apply`) and wake the last active 会话
- 配置 writes include a base-hash guard to prevent clobbering concurrent edits
- Config schema + form rendering (`config.schema`, including 插件 + 渠道 schemas); Raw JSON editor remains available
- Debug: status/health/models snapshots + event log + manual RPC calls (`status`, `health`, `models.list`)
- Logs: live tail of gateway file logs with filter/export (`logs.tail`)
- Update: run a package/git update + restart (`update.run`) with a restart report

## Chat 行为

- `chat.send` is **non-blocking**: it acks immediately with `{ runId, status: "started" }` and the response streams via `chat` 事件.
- Re-sending with the same `idempotencyKey` returns `{ status: "in_flight" }` while running, and `{ status: "ok" }` after completion.
- `chat.inject` appends an assistant note to the session transcript and broadcasts a `chat` 事件 for UI-only 更新 (no 代理 run, no 渠道 delivery).
- Stop:
  - Click **Stop** (calls `chat.abort`)
  - Type `/stop` (or `stop|esc|abort|wait|exit|interrupt`) to abort out-of-band
  - `chat.abort` supports `{ sessionKey }` (no `runId`) to abort all active runs for that 会话

## Tailnet access (recommended)

### Integrated Tailscale Serve (preferred)

Keep the Gateway on loopback and let Tailscale Serve proxy it with HTTPS:

```bash
moltbot gateway --tailscale serve
```

Open:
- `https://<magicdns>/` (or your configured `gateway.controlUi.basePath`)

默认情况下, Serve 请求 can authenticate via Tailscale identity headers
(`tailscale-user-login`) when `gateway.auth.allowTailscale` is `true`. Moltbot
verifies the identity by resolving the `x-forwarded-for` address with
`tailscale whois` and matching it to the header, and only accepts these when the
request hits loopback with Tailscale’s `x-forwarded-*` headers. Set
`gateway.auth.allowTailscale: false` (or force `gateway.auth.mode: "password"`)
if you want to require a 令牌/password even for Serve traffic.

### Bind to tailnet + 令牌

```bash
moltbot gateway --bind tailnet --token "$(openssl rand -hex 32)"
```

Then open:
- `http://<tailscale-ip>:18789/` (or your configured `gateway.controlUi.basePath`)

Paste the token into the UI settings (sent as `connect.params.auth.token`).

## Insecure HTTP

If you open the dashboard over plain HTTP (`http://<lan-ip>` or `http://<tailscale-ip>`),
the browser runs in a **non-secure 上下文** and blocks WebCrypto. 默认情况下,
Moltbot **blocks** Control UI 连接 without 设备 identity.

**Recommended fix:** use HTTPS (Tailscale Serve) or open the UI locally:
- `https://<magicdns>/` (Serve)
- `http://127.0.0.1:18789/` (on the Gateway 主机)

**Downgrade 示例 (令牌-only over HTTP):**

```json5
{
  gateway: {
    controlUi: { allowInsecureAuth: true },
    bind: "tailnet",
    auth: { mode: "token", token: "replace-me" }
  }
}
```

This disables 设备 identity + pairing for the Control UI (even on HTTPS). Use
only if you trust the 网络.

参见 [Tailscale](/Gateway/tailscale) for HTTPS 设置 guidance.

## Building the UI

The Gateway serves static files from `dist/control-ui`. Build them with:

```bash
pnpm ui:build # auto-installs UI deps on first run
```

可选 absolute base (when you want fixed asset URLs):

```bash
CLAWDBOT_CONTROL_UI_BASE_PATH=/moltbot/ pnpm ui:build
```

For local 开发 (separate dev 服务器):

```bash
pnpm ui:dev # auto-installs UI deps on first run
```

Then point the UI at your Gateway WS URL (e.g. `ws://127.0.0.1:18789`).

## 调试/测试: dev 服务器 + remote Gateway

The Control UI is static 文件; the WebSocket target is configurable and can be
different from the HTTP origin. This is handy when you want the Vite dev 服务器
locally but the Gateway runs elsewhere.

1) Start the UI dev server: `pnpm ui:dev`
2) Open a URL like:

```text
http://localhost:5173/?gatewayUrl=ws://<gateway-host>:18789
```

可选 one-time 认证 (if needed):

```text
http://localhost:5173/?gatewayUrl=wss://<gateway-host>:18789&token=<gateway-token>
```

Notes:
- `gatewayUrl` is stored in localStorage after load and removed from the URL.
- `token` is stored in localStorage; `password` is kept in memory only.
- Use `wss://` when the Gateway is behind TLS (Tailscale Serve, HTTPS proxy, etc.).

Remote access 设置 details: [Remote access](/Gateway/remote).
