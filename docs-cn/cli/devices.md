---
summary: "`moltbot devices` CLI 参考(设备配对 + 令牌轮换/吊销)"
read_when:
  - 您正在批准设备配对请求
  - 您需要轮换或吊销设备令牌
---

# `moltbot devices`

管理设备配对请求和设备作用域的令牌。

## 命令

### `moltbot devices list`

列出待处理的配对请求和已配对的设备。

```
moltbot devices list
moltbot devices list --json
```

### `moltbot devices approve <requestId>`

批准待处理的设备配对请求。

```
moltbot devices approve <requestId>
```

### `moltbot devices reject <requestId>`

拒绝待处理的设备配对请求。

```
moltbot devices reject <requestId>
```

### `moltbot devices rotate --device <id> --role <role> [--scope <scope...>]`

为特定角色轮换设备令牌(可选择更新作用域)。

```
moltbot devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### `moltbot devices revoke --device <id> --role <role>`

为特定角色吊销设备令牌。

```
moltbot devices revoke --device <deviceId> --role node
```

## 常用选项

- `--url <url>`: Gateway WebSocket URL(默认为配置中的 `gateway.remote.url`)。
- `--token <token>`: Gateway 令牌(如果需要)。
- `--password <password>`: Gateway 密码(密码认证)。
- `--timeout <ms>`: RPC 超时。
- `--json`: JSON 输出(推荐用于脚本编写)。

## 注意事项

- 令牌轮换返回新令牌(敏感)。请将其视为秘密。
- 这些命令需要 `operator.pairing`(或 `operator.admin`)作用域。
