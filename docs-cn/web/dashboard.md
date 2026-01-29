---
summary: "Gateway dashboard (Control UI) access and 认证"
read_when: 
  - Changing dashboard authentication or exposure modes
---
# Dashboard (Control UI)

The Gateway dashboard is the browser Control UI served at `/` 默认情况下
(override with `gateway.controlUi.basePath`).

Quick open (local Gateway):
- http://127.0.0.1:18789/ (or http://localhost:18789/)

键 参考:
- [Control UI](/Web/control-ui) for 用法 and UI capabilities.
- [Tailscale](/Gateway/tailscale) for Serve/Funnel automation.
- [Web surfaces](/Web) for bind modes and 安全 notes.

Authentication is enforced at the WebSocket handshake via `connect.params.auth`
(token or password). See `gateway.auth` in [Gateway 配置](/Gateway/配置).

安全 注意: the Control UI is an **admin surface** (chat, 配置, exec approvals).
Do not expose it publicly. The UI stores the token in `localStorage` after first load.
Prefer localhost, Tailscale Serve, or an SSH tunnel.

## Fast 路径 (recommended)

- After onboarding, the CLI now auto-opens the dashboard with your 令牌 and prints the same tokenized link.
- Re-open anytime: `moltbot dashboard` (copies link, opens browser if possible, shows SSH hint if headless).
- The 令牌 stays local (查询 param only); the UI strips it after first load and saves it in localStorage.

## 令牌 basics (local vs remote)

- **Localhost**: open `http://127.0.0.1:18789/`. If you see “unauthorized,” run `moltbot dashboard` and use the tokenized link (`?token=...`).
- **Token source**: `gateway.auth.token` (or `CLAWDBOT_GATEWAY_TOKEN`); the UI stores it after first load.
- **Not localhost**: use Tailscale Serve (tokenless if `gateway.auth.allowTailscale: true`), tailnet bind with a 令牌, or an SSH tunnel. 参见 [Web surfaces](/Web).

## If you 参见 “unauthorized” / 1008

- Run `moltbot dashboard` to get a fresh tokenized link.
- Ensure the gateway is reachable (local: `moltbot status`; remote: SSH tunnel `ssh -N -L 18789:127.0.0.1:18789 user@host` then open `http://127.0.0.1:18789/?token=...`).
- In the dashboard settings, paste the same token you configured in `gateway.auth.token` (or `CLAWDBOT_GATEWAY_TOKEN`).
