---
summary: "用于定向调试日志的诊断标志"
read_when:
  - 您需要定向调试日志而不提高全局日志记录级别
  - 您需要捕获特定子系统的日志以获得支持
---

# 诊断标志

诊断标志允许您启用定向调试日志，而无需到处启用详细日志记录。标志是可选的，除非子系统检查它们，否则它们没有效果。

## 工作原理

- 标志是字符串（不区分大小写）。
- 您可以在配置中或通过 env 覆盖来启用标志。
- 支持通配符：
  - `telegram.*` 匹配 `telegram.http`
  - `*` 启用所有标志

## 通过配置启用

```json
{
  "diagnostics": {
    "flags": ["telegram.http"]
  }
}
```

多个标志：

```json
{
  "diagnostics": {
    "flags": ["telegram.http", "gateway.*"]
  }
}
```

更改标志后重启 gateway。

## 环境变量覆盖（一次性）

```bash
CLAWDBOT_DIAGNOSTICS=telegram.http,telegram.payload
```

禁用所有标志：

```bash
CLAWDBOT_DIAGNOSTICS=0
```

## 日志去向

标志将日志发送到标准诊断日志文件。默认情况下：

```
/tmp/moltbot/moltbot-YYYY-MM-DD.log
```

如果您设置了 `logging.file`，则使用该路径。日志是 JSONL（每行一个 JSON 对象）。基于 `logging.redactSensitive` 仍然应用编辑。

## 提取日志

选择最新的日志文件：

```bash
ls -t /tmp/moltbot/moltbot-*.log | head -n 1
```

过滤 Telegram HTTP 诊断：

```bash
rg "telegram http error" /tmp/moltbot/moltbot-*.log
```

或在重现时使用 tail：

```bash
tail -f /tmp/moltbot/moltbot-$(date +%F).log | rg "telegram http error"
```

对于远程 gateway，您也可以使用 `moltbot logs --follow`（参见 [/cli/logs](/cli/logs)）。

## 注意事项

- 如果 `logging.level` 设置高于 `warn`，这些日志可能会被抑制。默认 `info` 就可以。
- 标志可以安全地保持启用；它们仅影响特定子系统的日志量。
- 使用 [/logging](/logging) 更改日志目标、级别和编辑。
