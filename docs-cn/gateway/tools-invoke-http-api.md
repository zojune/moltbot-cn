---
summary: "通过 Gateway HTTP 端点直接调用单个工具"
read_when:
  - 在不运行完整 agent 回合的情况下调用工具
  - 构建需要工具策略强制执行的自动化
---
# 工具调用（HTTP）

Moltbot 的 Gateway 公开了一个简单的 HTTP 端点，用于直接调用单个工具。它始终启用，但由 Gateway 认证和工具策略限制。

- `POST /tools/invoke`
- 与 Gateway 相同的端口（WS + HTTP 多路复用）：`http://<gateway-host>:<port>/tools/invoke`

默认最大负载大小为 2 MB。

## 认证

使用 Gateway 认证配置。发送 bearer token：

- `Authorization: Bearer <token>`

注意事项：
- 当 `gateway.auth.mode="token"` 时，使用 `gateway.auth.token`（或 `CLAWDBOT_GATEWAY_TOKEN`）。
- 当 `gateway.auth.mode="password"` 时，使用 `gateway.auth.password`（或 `CLAWDBOT_GATEWAY_PASSWORD`）。

## 请求主体

```json
{
  "tool": "sessions_list",
  "action": "json",
  "args": {},
  "sessionKey": "main",
  "dryRun": false
}
```

字段：
- `tool`（字符串，必需）：要调用的工具名称。
- `action`（字符串，可选）：如果工具架构支持 `action` 且 args 负载省略了它，则映射到 args。
- `args`（对象，可选）：工具特定的参数。
- `sessionKey`（字符串，可选）：目标会话密钥。如果省略或 `"main"`，Gateway 使用配置的主会话密钥（尊重 `session.mainKey` 和默认 agent，或全局作用域中的 `global`）。
- `dryRun`（布尔，可选）：保留供将来使用；当前被忽略。

## 策略 + 路由行为

工具可用性通过与 Gateway agents 相同的策略链进行过滤：
- `tools.profile` / `tools.byProvider.profile`
- `tools.allow` / `tools.byProvider.allow`
- `agents.<id>.tools.allow` / `agents.<id>.tools.byProvider.allow`
- 组策略（如果会话键映射到群组或通道）
- 子 agent 策略（使用子 agent 会话键调用时）

如果策略不允许工具，端点返回 **404**。

为了帮助组策略解析上下文，您可以选择设置：
- `x-moltbot-message-channel: <channel>`（示例：`slack`、`telegram`）
- `x-moltbot-account-id: <accountId>`（当存在多个帐户时）

## 响应

- `200` → `{ ok: true, result }`
- `400` → `{ ok: false, error: { type, message } }`（无效请求或工具错误）
- `401` → 未经授权
- `404` → 工具不可用（未找到或未被允许列表）
- `405` → 不允许的方法

## 示例

```bash
curl -sS http://127.0.0.1:18789/tools/invoke \
  -H 'Authorization: Bearer YOUR_TOKEN' \
  -H 'Content-Type: application/json' \
  -d '{
    "tool": "sessions_list",
    "action": "json",
    "args": {}
  }'
```
