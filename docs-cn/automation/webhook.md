---
summary: "用于唤醒和隔离代理运行的 Webhook 接入"
read_when:
  - 添加或更改 webhook 端点
  - 将外部系统集成到 Moltbot
---

# Webhooks

网关可以暴露一个小型 HTTP webhook 端点，用于外部触发器。

## 启用

```json5
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks"
  }
}
```

注意：
- 当 `hooks.enabled=true` 时，`hooks.token` 是必需的。
- `hooks.path` 默认为 `/hooks`。

## 认证

每个请求必须包含 hook 令牌。首选使用请求头：
- `Authorization: Bearer <token>`（推荐）
- `x-moltbot-token: <token>`
- `?token=<token>`（已弃用；记录警告并将在未来的主要版本中删除）

## 端点

### `POST /hooks/wake`

有效负载：
```json
{ "text": "系统行", "mode": "now" }
```

- `text` **必需**（字符串）：事件的描述（例如"收到新邮件"）。
- `mode` 可选（`now` | `next-heartbeat`）：是立即触发心跳（默认 `now`）还是等待下一次定期检查。

效果：
- 为**主**会话将系统事件加入队列
- 如果 `mode=now`，立即触发心跳

### `POST /hooks/agent`

有效负载：
```json
{
  "message": "运行这个",
  "name": "邮件",
  "sessionKey": "hook:email:msg-123",
  "wakeMode": "now",
  "deliver": true,
  "channel": "last",
  "to": "+15551234567",
  "model": "openai/gpt-5.2-mini",
  "thinking": "low",
  "timeoutSeconds": 120
}
```

- `message` **必需**（字符串）：代理要处理的提示或消息。
- `name` 可选（字符串）：hook 的人类可读名称（例如"GitHub"），用作会话摘要中的前缀。
- `sessionKey` 可选（字符串）：用于标识代理会话的密钥。默认为随机的 `hook:<uuid>`。使用一致的密钥允许在 hook 上下文中进行多轮对话。
- `wakeMode` 可选（`now` | `next-heartbeat`）：是立即触发心跳（默认 `now`）还是等待下一次定期检查。
- `deliver` 可选（布尔值）：如果为 `true`，代理的响应将被发送到消息频道。默认为 `true`。仅为心跳确认的响应会被自动跳过。
- `channel` 可选（字符串）：传递的消息频道。以下之一：`last`、`whatsapp`、`telegram`、`discord`、`slack`、`mattermost`（插件）、`signal`、`imessage`、`msteams`。默认为 `last`。
- `to` 可选（字符串）：频道的接收者标识符（例如 WhatsApp/Signal 的电话号码、Telegram 的聊天 ID、Discord/Slack/Mattermost（插件）/MS Teams 的频道 ID）。默认为主会话中的最后一个接收者。
- `model` 可选（字符串）：模型覆盖（例如 `anthropic/claude-3-5-sonnet` 或别名）。如果受限，必须在允许的模型列表中。
- `thinking` 可选（字符串）：思考级别覆盖（例如 `low`、`medium`、`high`）。
- `timeoutSeconds` 可选（数字）：代理运行的最大持续时间（秒）。

效果：
- 运行**隔离**的代理轮次（自己的会话密钥）
- 始终将摘要发布到**主**会话
- 如果 `wakeMode=now`，立即触发心跳

### `POST /hooks/<name>`（映射）

自定义 hook 名称通过 `hooks.mappings` 解析（请参阅配置）。映射可以将任意有效负载转换为 `wake` 或 `agent` 操作，并带有可选的模板或代码转换。

映射选项（摘要）：
- `hooks.presets: ["gmail"]` 启用内置的 Gmail 映射。
- `hooks.mappings` 允许您在配置中定义 `match`、`action` 和模板。
- `hooks.transformsDir` + `transform.module` 加载用于自定义逻辑的 JS/TS 模块。
- 使用 `match.source` 保持通用接入端点（有效负载驱动的路由）。
- TS 转换需要 TS 加载器（例如 `bun` 或 `tsx`）或运行时预编译的 `.js`。
- 在映射上设置 `deliver: true` + `channel`/`to` 以将回复路由到聊天界面
  （`channel` 默认为 `last` 并回退到 WhatsApp）。
- `allowUnsafeExternalContent: true` 为该 hook 禁用外部内容安全包装器
  （危险；仅适用于受信任的内部来源）。
- `moltbot webhooks gmail setup` 为 `moltbot webhooks gmail run` 编写 `hooks.gmail` 配置。
请参阅 [Gmail Pub/Sub](/automation/gmail-pubsub) 了解完整的 Gmail watch 流程。

## 响应

- `/hooks/wake` 返回 `200`
- `/hooks/agent` 返回 `202`（异步运行已启动）
- 认证失败返回 `401`
- 无效有效负载返回 `400`
- 有效负载过大返回 `413`

## 示例

```bash
curl -X POST http://127.0.0.1:18789/hooks/wake \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"text":"收到新邮件","mode":"now"}'
```

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-moltbot-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"总结收件箱","name":"邮件","wakeMode":"next-heartbeat"}'
```

### 使用不同的模型

将 `model` 添加到代理有效负载（或映射）以覆盖该运行的模型：

```bash
curl -X POST http://127.0.0.1:18789/hooks/agent \
  -H 'x-moltbot-token: SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"message":"总结收件箱","name":"邮件","model":"openai/gpt-5.2-mini"}'
```

如果您强制执行 `agents.defaults.models`，请确保覆盖模型包含在其中。

```bash
curl -X POST http://127.0.0.1:18789/hooks/gmail \
  -H 'Authorization: Bearer SECRET' \
  -H 'Content-Type: application/json' \
  -d '{"source":"gmail","messages":[{"from":"Ada","subject":"你好","snippet":"嗨"}]}'
```

## 安全

- 将 hook 端点保持在环回、tailnet 或受信任的反向代理后面。
- 使用专用的 hook 令牌；不要重用网关认证令牌。
- 避免在 webhook 日志中包含敏感的原始有效负载。
- Hook 有效负载默认被视为不受信任，并使用安全边界包装。
  如果您必须为特定 hook 禁用此功能，请在该 hook 的映射中设置 `allowUnsafeExternalContent: true`
  （危险）。
