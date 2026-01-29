---
summary: "Terminal UI (TUI): connect to the Gateway from any machine"
read_when: 
  - You want a beginner-friendly walkthrough of the TUI
  - You need the complete list of TUI features, commands, and shortcuts
---
# TUI (Terminal UI)

## 快速开始
1) Start the Gateway.
```bash
moltbot gateway
```
2) Open the TUI.
```bash
moltbot tui
```
3) 类型 a 消息 and press Enter.

Remote Gateway:
```bash
moltbot tui --url ws://<host>:<port> --token <gateway-token>
```
Use `--password` if your Gateway uses password 认证.

## What you 参见
- Header: 连接 URL, current 代理, current 会话.
- Chat 日志: 用户 消息, assistant replies, 系统 notices, 工具 cards.
- 状态 line: 连接/run 状态 (connecting, running, 流式, idle, 错误).
- Footer: 连接 状态 + 代理 + 会话 + 模型 + think/verbose/reasoning + 令牌 counts + deliver.
- 输入: text editor with autocomplete.

## Mental 模型: 代理 + 会话
- Agents are unique slugs (e.g. `main`, `research`). The Gateway exposes the list.
- 会话 belong to the current 代理.
- Session keys are stored as `agent:<agentId>:<sessionKey>`.
  - If you type `/session main`, the TUI expands it to `agent:<currentAgent>:main`.
  - If you type `/session agent:other:main`, you switch to that 代理 会话 explicitly.
- 会话 scope:
  - `per-sender` (默认): each 代理 has many 会话.
  - `global`: the TUI always uses the `global` 会话 (the picker may be empty).
- The current 代理 + 会话 are always visible in the footer.

## Sending + delivery
- 消息 are sent to the Gateway; delivery to 提供商 is off 默认情况下.
- Turn delivery on:
  - `/deliver on`
  - or the 设置 panel
  - or start with `moltbot tui --deliver`

## Pickers + overlays
- 模型 picker: list available 模型 and set the 会话 override.
- 代理 picker: choose a different 代理.
- 会话 picker: shows only 会话 for the current 代理.
- 设置: toggle deliver, 工具 输出 expansion, and thinking visibility.

## Keyboard shortcuts
- Enter: send 消息
- Esc: abort active run
- Ctrl+C: clear 输入 (press twice to exit)
- Ctrl+D: exit
- Ctrl+L: 模型 picker
- Ctrl+G: 代理 picker
- Ctrl+P: 会话 picker
- Ctrl+O: toggle 工具 输出 expansion
- Ctrl+T: toggle thinking visibility (reloads history)

## Slash 命令
Core:
- `/help`
- `/status`
- `/agent <id>` (or `/agents`)
- `/session <key>` (or `/sessions`)
- `/model <provider/model>` (or `/models`)

会话 controls:
- `/think <off|minimal|low|medium|high>`
- `/verbose <on|full|off>`
- `/reasoning <on|off|stream>`
- `/usage <off|tokens|full>`
- `/elevated <on|off|ask|full>` (alias: `/elev`)
- `/activation <mention|always>`
- `/deliver <on|off>`

会话 lifecycle:
- `/new` or `/reset` (reset the 会话)
- `/abort` (abort the active run)
- `/settings`
- `/exit`

Other Gateway slash commands (for example, `/context`) are forwarded to the Gateway and shown as 系统 输出. 参见 [Slash 命令](/工具/slash-命令).

## Local shell 命令
- Prefix a line with `!` to run a local shell 命令 on the TUI 主机.
- The TUI prompts once per session to allow local execution; declining keeps `!` 已禁用 for the 会话.
- Commands run in a fresh, non-interactive shell in the TUI working directory (no persistent `cd`/env).
- A lone `!` is sent as a normal 消息; leading spaces do not 触发器 local exec.

## 工具 输出
- 工具 calls show as cards with args + 结果.
- Ctrl+O toggles between collapsed/expanded views.
- While 工具 run, partial 更新 流 into the same card.

## History + 流式
- On connect, the TUI loads the latest history (默认 200 消息).
- 流式 响应 更新 in place until finalized.
- The TUI also listens to 代理 工具 事件 for richer 工具 cards.

## 连接 details
- The TUI registers with the Gateway as `mode: "tui"`.
- Reconnects show a 系统 消息; 事件 gaps are surfaced in the 日志.

## 选项
- `--url <url>`: Gateway WebSocket URL (defaults to config or `ws://127.0.0.1:<port>`)
- `--token <token>`: Gateway 令牌 (if 必需)
- `--password <password>`: Gateway password (if 必需)
- `--session <key>`: Session key (default: `main`, or `global` when scope is global)
- `--deliver`: Deliver assistant replies to the 提供商 (默认 off)
- `--thinking <level>`: Override thinking level for sends
- `--timeout-ms <ms>`: Agent timeout in ms (defaults to `agents.defaults.timeoutSeconds`)

## 故障排除

No 输出 after sending a 消息:
- Run `/status` in the TUI to confirm the Gateway is connected and idle/busy.
- Check the Gateway logs: `moltbot logs --follow`.
- Confirm the agent can run: `moltbot status` and `moltbot models status`.
- If you expect messages in a chat channel, enable delivery (`/deliver on` or `--deliver`).
- `--history-limit <n>`: History entries to load (默认 200)

## 故障排除
- `disconnected`: ensure the Gateway is running and your `--url/--token/--password` are correct.
- No agents in picker: check `moltbot agents list` and your routing 配置.
- Empty 会话 picker: you might be in global scope or have no 会话 yet.
