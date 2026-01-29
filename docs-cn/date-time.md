---
summary: "Date and time handling across envelopes, prompts, 工具, and connectors"
read_when: 
  - You are changing how timestamps are shown to the model or users
  - You are debugging time formatting in messages or system prompt output
---

# Date & Time

Moltbot defaults to **主机-local time for transport timestamps** and **用户 timezone only in the 系统 prompt**.
Provider timestamps are preserved so tools keep their native semantics (current time is available via `session_status`).

## 消息 envelopes (local 默认情况下)

Inbound 消息 are wrapped with a timestamp (minute precision):

```
[Provider ... 2026-01-05 16:26 PST] message text
```

This envelope timestamp is **主机-local 默认情况下**, regardless of the 提供商 timezone.

You can override this 行为:

```json5
{
  agents: {
    defaults: {
      envelopeTimezone: "local", // "utc" | "local" | "user" | IANA timezone
      envelopeTimestamp: "on", // "on" | "off"
      envelopeElapsed: "on" // "on" | "off"
    }
  }
}
```

- `envelopeTimezone: "utc"` uses UTC.
- `envelopeTimezone: "local"` uses the 主机 timezone.
- `envelopeTimezone: "user"` uses `agents.defaults.userTimezone` (falls back to 主机 timezone).
- Use an explicit IANA timezone (e.g., `"America/Chicago"`) for a fixed zone.
- `envelopeTimestamp: "off"` removes absolute timestamps from envelope headers.
- `envelopeElapsed: "off"` removes elapsed time suffixes (the `+2m` style).

### 示例

**Local (默认):**

```
[WhatsApp +1555 2026-01-18 00:19 PST] hello
```

**用户 timezone:**

```
[WhatsApp +1555 2026-01-18 00:19 CST] hello
```

**Elapsed time 已启用:**

```
[WhatsApp +1555 +30s 2026-01-18T05:19Z] follow-up
```

## 系统 prompt: Current Date & Time

If the 用户 timezone is known, the 系统 prompt includes a dedicated
**Current Date & Time** section with the **time zone only** (no clock/time 格式)
to keep prompt caching stable:

```
Time zone: America/Chicago
```

When the agent needs the current time, use the `session_status` 工具; the 状态
card includes a timestamp line.

## 系统 事件 lines (local 默认情况下)

Queued 系统 事件 inserted into 代理 上下文 are prefixed with a timestamp using the
same timezone selection as 消息 envelopes (默认: 主机-local).

```
System: [2026-01-12 12:19:17 PST] Model switched.
```

### Configure 用户 timezone + 格式

```json5
{
  agents: {
    defaults: {
      userTimezone: "America/Chicago",
      timeFormat: "auto" // auto | 12 | 24
    }
  }
}
```

- `userTimezone` sets the **用户-local timezone** for prompt 上下文.
- `timeFormat` controls **12h/24h display** in the prompt. `auto` follows OS prefs.

## Time 格式 detection (auto)

When `timeFormat: "auto"`, Moltbot inspects the OS preference (macOS/Windows)
and falls back to locale formatting. The detected 值 is **cached per 进程**
to avoid repeated 系统 calls.

## 工具 payloads + connectors (raw 提供商 time + normalized fields)

渠道 工具 return **提供商-native timestamps** and add normalized fields for consistency:

- `timestampMs`: epoch milliseconds (UTC)
- `timestampUtc`: ISO 8601 UTC 字符串

Raw 提供商 fields are preserved so nothing is lost.

- Slack: epoch-like strings from the API
- Discord: UTC ISO timestamps
- Telegram/WhatsApp: 提供商-specific numeric/ISO timestamps

If you need local time, convert it downstream using the known timezone.

## 相关 docs

- [系统 Prompt](/concepts/系统-prompt)
- [Timezones](/concepts/timezone)
- [消息](/concepts/消息)
