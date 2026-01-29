---
summary: "agents、信封和提示的时区处理"
read_when:
  - 您需要了解时间戳如何为模型规范化
  - 为系统提示配置用户时区
---

# 时区

Moltbot 标准化时间戳，以便模型看到**单一参考时间**。

## 消息信封（默认为本地）

入站消息包装在信封中，如：

```
[Provider ... 2026-01-05 16:26 PST] message text
```

信封中的时间戳默认为**主机本地**，精确到分钟。

您可以使用以下内容覆盖：

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

- `envelopeTimezone: "utc"` 使用 UTC。
- `envelopeTimezone: "user"` 使用 `agents.defaults.userTimezone`（回退到主机时区）。
- 使用显式 IANA 时区（例如 `"Europe/Vienna"`）进行固定偏移。
- `envelopeTimestamp: "off"` 从信封标头中删除绝对时间戳。
- `envelopeElapsed: "off"` 删除经过时间后缀（`+2m` 样式）。

### 示例

**本地（默认）：**

```
[Signal Alice +1555 2026-01-18 00:19 PST] hello
```

**固定时区：**

```
[Signal Alice +1555 2026-01-18 06:19 GMT+1] hello
```

**经过时间：**

```
[Signal Alice +1555 +2m 2026-01-18T05:19Z] follow-up
```

## 工具负载（原始提供程序数据 + 规范化字段）

工具调用（`channels.discord.readMessages`、`channels.slack.readMessages` 等）返回**原始提供程序时间戳**。
我们还附加规范化字段以保持一致性：

- `timestampMs`（UTC 纪元毫秒）
- `timestampUtc`（ISO 8601 UTC 字符串）

保留原始提供程序字段。

## 系统提示的用户时区

设置 `agents.defaults.userTimezone` 以告诉模型用户的本地时区。如果未设置，Moltbot 在运行时解析**主机时区**（不写入配置）。

```json5
{
  agents: { defaults: { userTimezone: "America/Chicago" } }
}
```

系统提示包括：
- `Current Date & Time` 部分，带有本地时间和时区
- `Time format: 12-hour` 或 `24-hour`

您可以使用 `agents.defaults.timeFormat`（`auto` | `12` | `24`）控制提示格式。

请参阅 [Date & Time](/date-time) 了解完整的行为和示例。
