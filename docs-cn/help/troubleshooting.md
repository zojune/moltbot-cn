---
summary: "故障排除 hub: symptoms → checks → fixes"
read_when: 
  - You see an error and want the fix path
  - The installer says “success” but the CLI doesn’t work
---

# 故障排除

## First 60 seconds

Run these in order:

```bash
moltbot status
moltbot status --all
moltbot gateway probe
moltbot logs --follow
moltbot doctor
```

If the Gateway is reachable, deep probes:

```bash
moltbot status --deep
```

## Common “it broke” cases

### `moltbot: command not found`

Almost always a 节点/npm 路径 issue. Start here:

- [安装 (节点/npm 路径 sanity)](/安装#nodejs--npm-路径-sanity)

### Installer fails (or you need full 日志)

Re-run the installer in verbose mode to 参见 the full trace and npm 输出:

```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --verbose
```

For beta installs:

```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --beta --verbose
```

You can also set `CLAWDBOT_VERBOSE=1` instead of the flag.

### Gateway “unauthorized”, can’t connect, or keeps reconnecting

- [Gateway 故障排除](/Gateway/故障排除)
- [Gateway 认证](/Gateway/认证)

### Control UI fails on HTTP (设备 identity 必需)

- [Gateway 故障排除](/Gateway/故障排除)
- [Control UI](/Web/control-ui#insecure-http)

### `docs.molt.bot` shows an SSL 错误 (Comcast/Xfinity)

Some Comcast/Xfinity connections block `docs.molt.bot` via Xfinity Advanced 安全.
Disable Advanced Security or add `docs.molt.bot` to the allowlist, then retry.

- Xfinity Advanced 安全 help: https://www.xfinity.com/support/articles/using-xfinity-xfi-advanced-安全
- Quick sanity checks: 尝试 a mobile hotspot or VPN to confirm it’s ISP-level filtering

### 服务 says running, but RPC probe fails

- [Gateway 故障排除](/Gateway/故障排除)
- [Background 进程 / 服务](/Gateway/background-进程)

### 模型/认证 failures (rate limit, billing, “all 模型 failed”)

- [模型](/cli/模型)
- [OAuth / 认证 concepts](/concepts/oauth)

### `/model` says `model not allowed`

This usually means `agents.defaults.models` is configured as an allowlist. When it’s non-empty,
only those 提供商/模型 keys can be selected.

- Check the allowlist: `moltbot config get agents.defaults.models`
- Add the model you want (or clear the allowlist) and retry `/model`
- Use `/models` to browse the allowed 提供商/模型

### When filing an issue

Paste a safe report:

```bash
moltbot status --all
```

If you can, include the relevant log tail from `moltbot logs --follow`.
