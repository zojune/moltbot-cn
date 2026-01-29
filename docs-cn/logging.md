---
summary: "日志记录 概述: 文件 日志, console 输出, CLI tailing, and the Control UI"
read_when: 
  - You need a beginner-friendly overview of logging
  - You want to configure log levels or formats
  - You are troubleshooting and need to find logs quickly
---

# 日志记录

Moltbot 日志 in two places:

- **文件 日志** (JSON lines) written by the Gateway.
- **Console 输出** shown in terminals and the Control UI.

This page explains where 日志 live, 如何 read them, and 如何 configure 日志
levels and formats.

## Where 日志 live

默认情况下, the Gateway writes a rolling 日志 文件 under:

`/tmp/moltbot/moltbot-YYYY-MM-DD.log`

The date uses the Gateway 主机's local timezone.

You can override this in `~/.clawdbot/moltbot.json`:

```json
{
  "logging": {
    "file": "/path/to/moltbot.log"
  }
}
```

## 如何 read 日志

### CLI: live tail (recommended)

Use the CLI to tail the Gateway 日志 文件 via RPC:

```bash
moltbot logs --follow
```

输出 modes:

- **TTY 会话**: pretty, colorized, structured 日志 lines.
- **Non-TTY 会话**: plain text.
- `--json`: line-delimited JSON (one 日志 事件 per line).
- `--plain`: force plain text in TTY 会话.
- `--no-color`: disable ANSI colors.

In JSON mode, the CLI emits `type`-tagged objects:

- `meta`: 流 元数据 (文件, cursor, size)
- `log`: parsed 日志 entry
- `notice`: truncation / rotation hints
- `raw`: unparsed 日志 line

If the Gateway is unreachable, the CLI prints a short hint to run:

```bash
moltbot doctor
```

### Control UI (Web)

The Control UI’s **Logs** tab tails the same file using `logs.tail`.
参见 [/Web/control-ui](/Web/control-ui) for 如何 open it.

### 渠道-only 日志

To 过滤器 渠道 activity (WhatsApp/Telegram/etc), use:

```bash
moltbot channels logs --channel whatsapp
```

## 日志 formats

### 文件 日志 (JSONL)

Each line in the 日志 文件 is a JSON 对象. The CLI and Control UI parse these
entries to render structured 输出 (time, level, subsystem, 消息).

### Console 输出

Console 日志 are **TTY-aware** and formatted for readability:

- Subsystem prefixes (e.g. `gateway/channels/whatsapp`)
- Level coloring (信息/warn/错误)
- 可选 compact or JSON mode

Console formatting is controlled by `logging.consoleStyle`.

## Configuring 日志记录

All logging configuration lives under `logging` in `~/.clawdbot/moltbot.json`.

```json
{
  "logging": {
    "level": "info",
    "file": "/tmp/moltbot/moltbot-YYYY-MM-DD.log",
    "consoleLevel": "info",
    "consoleStyle": "pretty",
    "redactSensitive": "tools",
    "redactPatterns": [
      "sk-.*"
    ]
  }
}
```

### 日志 levels

- `logging.level`: **文件 日志** (JSONL) level.
- `logging.consoleLevel`: **console** verbosity level.

`--verbose` only affects console 输出; it does not change 文件 日志 levels.

### Console styles

`logging.consoleStyle`:

- `pretty`: human-friendly, colored, with timestamps.
- `compact`: tighter 输出 (best for long 会话).
- `json`: JSON per line (for 日志 processors).

### Redaction

工具 summaries can redact sensitive 令牌 before they hit the console:

- `logging.redactSensitive`: `off` | `tools` (default: `tools`)
- `logging.redactPatterns`: list of regex strings to override the 默认 set

Redaction affects **console 输出 only** and does not alter 文件 日志.

## Diagnostics + OpenTelemetry

Diagnostics are structured, machine-readable 事件 for 模型 runs **and**
消息-flow telemetry (Webhook, queueing, 会话 状态). They do **not**
replace 日志; they exist to feed metrics, traces, and other exporters.

Diagnostics 事件 are emitted in-进程, but exporters only attach when
diagnostics + the exporter 插件 are 已启用.

### OpenTelemetry vs OTLP

- **OpenTelemetry (OTel)**: the 数据 模型 + SDKs for traces, metrics, and 日志.
- **OTLP**: the wire 协议 used to export OTel 数据 to a collector/backend.
- Moltbot exports via **OTLP/HTTP (protobuf)** today.

### Signals exported

- **Metrics**: counters + histograms (令牌 用法, 消息 flow, queueing).
- **Traces**: spans for 模型 用法 + Webhook/消息 processing.
- **Logs**: exported over OTLP when `diagnostics.otel.logs` is 已启用. 日志
  volume can be high; keep `logging.level` and exporter 过滤器 in mind.

### Diagnostic 事件 catalog

模型 用法:
- `model.usage`: 令牌, cost, duration, 上下文, 提供商/模型/渠道, 会话 ids.

消息 flow:
- `webhook.received`: Webhook ingress per 渠道.
- `webhook.processed`: Webhook handled + duration.
- `webhook.error`: Webhook 处理器 错误.
- `message.queued`: 消息 enqueued for processing.
- `message.processed`: outcome + duration + 可选 错误.

Queue + 会话:
- `queue.lane.enqueue`: 命令 queue lane enqueue + depth.
- `queue.lane.dequeue`: 命令 queue lane dequeue + wait time.
- `session.state`: 会话 状态 transition + reason.
- `session.stuck`: 会话 stuck 警告 + age.
- `run.attempt`: run retry/attempt 元数据.
- `diagnostic.heartbeat`: aggregate counters (Webhook/queue/会话).

### Enable diagnostics (no exporter)

Use this if you want diagnostics 事件 available to 插件 or custom sinks:

```json
{
  "diagnostics": {
    "enabled": true
  }
}
```

### Diagnostics flags (targeted 日志)

Use flags to turn on extra, targeted debug logs without raising `logging.level`.
Flags are case-insensitive and support wildcards (e.g. `telegram.*` or `*`).

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

Env override (one-off):

```
CLAWDBOT_DIAGNOSTICS=telegram.http,telegram.payload
```

Notes:
- Flag logs go to the standard log file (same as `logging.file`).
- Output is still redacted according to `logging.redactSensitive`.
- Full 指南: [/diagnostics/flags](/diagnostics/flags).

### Export to OpenTelemetry

Diagnostics can be exported via the `diagnostics-otel` 插件 (OTLP/HTTP). This
works with any OpenTelemetry collector/backend that accepts OTLP/HTTP.

```json
{
  "plugins": {
    "allow": ["diagnostics-otel"],
    "entries": {
      "diagnostics-otel": {
        "enabled": true
      }
    }
  },
  "diagnostics": {
    "enabled": true,
    "otel": {
      "enabled": true,
      "endpoint": "http://otel-collector:4318",
      "protocol": "http/protobuf",
      "serviceName": "moltbot-gateway",
      "traces": true,
      "metrics": true,
      "logs": true,
      "sampleRate": 0.2,
      "flushIntervalMs": 60000
    }
  }
}
```

Notes:
- You can also enable the plugin with `moltbot plugins enable diagnostics-otel`.
- `protocol` currently supports `http/protobuf` only. `grpc` is ignored.
- Metrics include 令牌 用法, cost, 上下文 size, run duration, and 消息-flow
  counters/histograms (Webhook, queueing, 会话 状态, queue depth/wait).
- Traces/metrics can be toggled with `traces` / `metrics` (默认: on). Traces
  include 模型 用法 spans plus Webhook/消息 processing spans when 已启用.
- Set `headers` when your collector requires 认证.
- Environment variables supported: `OTEL_EXPORTER_OTLP_ENDPOINT`,
  `OTEL_SERVICE_NAME`, `OTEL_EXPORTER_OTLP_PROTOCOL`.

### Exported metrics (names + types)

模型 用法:
- `moltbot.tokens` (counter, attrs: `moltbot.token`, `moltbot.channel`,
  `moltbot.provider`, `moltbot.model`)
- `moltbot.cost.usd` (counter, attrs: `moltbot.channel`, `moltbot.provider`,
  `moltbot.model`)
- `moltbot.run.duration_ms` (histogram, attrs: `moltbot.channel`,
  `moltbot.provider`, `moltbot.model`)
- `moltbot.context.tokens` (histogram, attrs: `moltbot.context`,
  `moltbot.channel`, `moltbot.provider`, `moltbot.model`)

消息 flow:
- `moltbot.webhook.received` (counter, attrs: `moltbot.channel`,
  `moltbot.webhook`)
- `moltbot.webhook.error` (counter, attrs: `moltbot.channel`,
  `moltbot.webhook`)
- `moltbot.webhook.duration_ms` (histogram, attrs: `moltbot.channel`,
  `moltbot.webhook`)
- `moltbot.message.queued` (counter, attrs: `moltbot.channel`,
  `moltbot.source`)
- `moltbot.message.processed` (counter, attrs: `moltbot.channel`,
  `moltbot.outcome`)
- `moltbot.message.duration_ms` (histogram, attrs: `moltbot.channel`,
  `moltbot.outcome`)

Queues + 会话:
- `moltbot.queue.lane.enqueue` (counter, attrs: `moltbot.lane`)
- `moltbot.queue.lane.dequeue` (counter, attrs: `moltbot.lane`)
- `moltbot.queue.depth` (histogram, attrs: `moltbot.lane` or
  `moltbot.channel=heartbeat`)
- `moltbot.queue.wait_ms` (histogram, attrs: `moltbot.lane`)
- `moltbot.session.state` (counter, attrs: `moltbot.state`, `moltbot.reason`)
- `moltbot.session.stuck` (counter, attrs: `moltbot.state`)
- `moltbot.session.stuck_age_ms` (histogram, attrs: `moltbot.state`)
- `moltbot.run.attempt` (counter, attrs: `moltbot.attempt`)

### Exported spans (names + 键 attributes)

- `moltbot.model.usage`
  - `moltbot.channel`, `moltbot.provider`, `moltbot.model`
  - `moltbot.sessionKey`, `moltbot.sessionId`
  - `moltbot.tokens.*` (输入/输出/cache_read/cache_write/total)
- `moltbot.webhook.processed`
  - `moltbot.channel`, `moltbot.webhook`, `moltbot.chatId`
- `moltbot.webhook.error`
  - `moltbot.channel`, `moltbot.webhook`, `moltbot.chatId`,
    `moltbot.error`
- `moltbot.message.processed`
  - `moltbot.channel`, `moltbot.outcome`, `moltbot.chatId`,
    `moltbot.messageId`, `moltbot.sessionKey`, `moltbot.sessionId`,
    `moltbot.reason`
- `moltbot.session.stuck`
  - `moltbot.state`, `moltbot.ageMs`, `moltbot.queueDepth`,
    `moltbot.sessionKey`, `moltbot.sessionId`

### Sampling + flushing

- Trace sampling: `diagnostics.otel.sampleRate` (0.0–1.0, root spans only).
- Metric export interval: `diagnostics.otel.flushIntervalMs` (min 1000ms).

### 协议 notes

- OTLP/HTTP endpoints can be set via `diagnostics.otel.endpoint` or
  `OTEL_EXPORTER_OTLP_ENDPOINT`.
- If the endpoint already contains `/v1/traces` or `/v1/metrics`, it is used as-is.
- If the endpoint already contains `/v1/logs`, it is used as-is for 日志.
- `diagnostics.otel.logs` enables OTLP 日志 export for the main logger 输出.

### 日志 export 行为

- OTLP logs use the same structured records written to `logging.file`.
- Respect `logging.level` (文件 日志 level). Console redaction does **not** apply
  to OTLP 日志.
- High-volume installs should prefer OTLP collector sampling/filtering.

## 故障排除 tips

- **Gateway not reachable?** Run `moltbot doctor` first.
- **日志 empty?** Check that the Gateway is running and writing to the 文件 路径
  in `logging.file`.
- **Need more detail?** Set `logging.level` to `debug` or `trace` and retry.
