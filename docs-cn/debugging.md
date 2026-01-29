---
summary: "调试 工具: watch mode, raw 模型 streams, and tracing reasoning leakage"
read_when: 
  - You need to inspect raw model output for reasoning leakage
  - You want to run the Gateway in watch mode while iterating
  - You need a repeatable debugging workflow
---

# 调试

This page covers 调试 helpers for 流式 输出, especially when a
提供商 mixes reasoning into normal text.

## Runtime 调试 overrides

Use `/debug` in chat to set **runtime-only** 配置 overrides (memory, not disk).
`/debug` is disabled by default; enable with `commands.debug: true`.
This is handy when you need to toggle obscure settings without editing `moltbot.json`.

示例:

```
/debug show
/debug set messages.responsePrefix="[moltbot]"
/debug unset messages.responsePrefix
/debug reset
```

`/debug reset` clears all overrides and returns to the on-disk 配置.

## Gateway watch mode

For fast iteration, run the Gateway under the 文件 watcher:

```bash
pnpm gateway:watch --force
```

This maps to:

```bash
tsx watch src/entry.ts gateway --force
```

Add any gateway CLI flags after `gateway:watch` and they will be passed through
on each restart.

## Dev 配置文件 + dev Gateway (--dev)

Use the dev 配置文件 to isolate 状态 and spin up a safe, disposable 设置 for
debugging. There are **two** `--dev` flags:

- **Global `--dev` (profile):** isolates state under `~/.clawdbot-dev` and
  defaults the gateway port to `19001` (derived 端口 shift with it).
- **`gateway --dev`: tells the Gateway to auto-create a 默认 配置 +
  工作空间** when missing (and skip BOOTSTRAP.md).

Recommended flow (dev 配置文件 + dev bootstrap):

```bash
pnpm gateway:dev
CLAWDBOT_PROFILE=dev moltbot tui
```

If you don’t have a global install yet, run the CLI via `pnpm moltbot ...`.

What this does:

1) **Profile isolation** (global `--dev`)
   - `CLAWDBOT_PROFILE=dev`
   - `CLAWDBOT_STATE_DIR=~/.clawdbot-dev`
   - `CLAWDBOT_CONFIG_PATH=~/.clawdbot-dev/moltbot.json`
   - `CLAWDBOT_GATEWAY_PORT=19001` (browser/canvas shift accordingly)

2) **Dev bootstrap** (`gateway --dev`)
   - Writes a minimal config if missing (`gateway.mode=local`, bind loopback).
   - Sets `agent.workspace` to the dev 工作空间.
   - Sets `agent.skipBootstrap=true` (no BOOTSTRAP.md).
   - Seeds the 工作空间 文件 if missing:
     `AGENTS.md`, `SOUL.md`, `TOOLS.md`, `IDENTITY.md`, `USER.md`, `HEARTBEAT.md`.
   - 默认 identity: **C3‑PO** (协议 droid).
   - Skips channel providers in dev mode (`CLAWDBOT_SKIP_CHANNELS=1`).

Reset flow (fresh start):

```bash
pnpm gateway:dev:reset
```

Note: `--dev` is a **global** 配置文件 flag and gets eaten by some runners.
If you need to spell it out, use the env var form:

```bash
CLAWDBOT_PROFILE=dev moltbot gateway --dev --reset
```

`--reset` wipes 配置, 凭据, 会话, and the dev 工作空间 (using
`trash`, not `rm`), then recreates the 默认 dev 设置.

提示: if a non‑dev Gateway is already running (launchd/systemd), stop it first:

```bash
moltbot gateway stop
```

## Raw 流 日志记录 (Moltbot)

Moltbot can 日志 the **raw assistant 流** before any filtering/formatting.
This is the best way to 参见 whether reasoning is arriving as plain text deltas
(or as separate thinking blocks).

Enable it via CLI:

```bash
pnpm gateway:watch --force --raw-stream
```

可选 路径 override:

```bash
pnpm gateway:watch --force --raw-stream --raw-stream-path ~/.clawdbot/logs/raw-stream.jsonl
```

Equivalent env vars:

```bash
CLAWDBOT_RAW_STREAM=1
CLAWDBOT_RAW_STREAM_PATH=~/.clawdbot/logs/raw-stream.jsonl
```

默认 文件:

`~/.clawdbot/logs/raw-stream.jsonl`

## Raw chunk 日志记录 (pi-mono)

To capture **raw OpenAI-compat chunks** before they are parsed into blocks,
pi-mono exposes a separate logger:

```bash
PI_RAW_STREAM=1
```

可选 路径:

```bash
PI_RAW_STREAM_PATH=~/.pi-mono/logs/raw-openai-completions.jsonl
```

默认 文件:

`~/.pi-mono/logs/raw-openai-completions.jsonl`

> 注意: this is only emitted by 进程 using pi-mono’s
> `openai-completions` 提供商.

## Safety notes

- Raw 流 日志 can include full prompts, 工具 输出, and 用户 数据.
- Keep 日志 local and delete them after 调试.
- If you share 日志, scrub secrets and PII first.
