---
summary: "在 Moltbot 中使用 Qwen OAuth（免费层）"
read_when:
  - 你想将 Qwen 与 Moltbot 一起使用
  - 你想要 Qwen Coder 的免费层 OAuth 访问
---
# Qwen

Qwen 为 Qwen Coder 和 Qwen Vision 模型提供免费层 OAuth 流程
（每天 2,000 次请求，受 Qwen 速率限制限制）。

## 启用插件

```bash
moltbot plugins enable qwen-portal-auth
```

启用后重启网关。

## 身份验证

```bash
moltbot models auth login --provider qwen-portal --set-default
```

这将运行 Qwen 设备代码 OAuth 流程并将提供程序条目写入你的
`models.json`（加上 `qwen` 别名以便快速切换）。

## 模型 ID

- `qwen-portal/coder-model`
- `qwen-portal/vision-model`

切换模型：

```bash
moltbot models set qwen-portal/coder-model
```

## 重用 Qwen Code CLI 登录

如果你已经使用 Qwen Code CLI 登录，Moltbot 将在加载身份验证存储时从 `~/.qwen/oauth_creds.json` 同步凭据。你仍然需要 `models.providers.qwen-portal` 条目（使用上面的登录命令创建一个）。

## 注意事项

- 令牌自动刷新；如果刷新失败或访问被撤销，请重新运行登录命令。
- 默认基本 URL：`https://portal.qwen.ai/v1`（如果 Qwen 提供不同的端点，请使用 `models.providers.qwen-portal.baseUrl` 覆盖）。
- 有关提供程序范围的规则，请参阅 [模型提供程序](/concepts/model-providers)。
