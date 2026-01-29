---
summary: "使用 OpenRouter 的统一 API 在 Moltbot 中访问许多模型"
read_when:
  - 你想要一个用于许多 LLM 的 API 密钥
  - 你想在 Moltbot 中通过 OpenRouter 运行模型
---
# OpenRouter

OpenRouter 提供一个**统一 API**，将请求路由到单个端点和 API 密钥后面的许多模型。它与 OpenAI 兼容，因此大多数 OpenAI SDK 只需切换基本 URL 即可工作。

## CLI 设置

```bash
moltbot onboard --auth-choice apiKey --token-provider openrouter --token "$OPENROUTER_API_KEY"
```

## 配置片段

```json5
{
  env: { OPENROUTER_API_KEY: "sk-or-..." },
  agents: {
    defaults: {
      model: { primary: "openrouter/anthropic/claude-sonnet-4-5" }
    }
  }
}
```

## 注意事项

- 模型引用是 `openrouter/<provider>/<model>`。
- 有关更多模型/提供程序选项，请参阅 [/concepts/model-providers](/concepts/model-providers)。
- OpenRouter 在底层使用带有 API 密钥的 Bearer 令牌。
