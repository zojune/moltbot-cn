---
summary: "将 Z.AI (GLM 模型) 与 Moltbot 一起使用"
read_when:
  - 你想在 Moltbot 中使用 Z.AI / GLM 模型
  - 你需要简单的 ZAI_API_KEY 设置
---
# Z.AI

Z.AI 是 **GLM** 模型的 API 平台。它为 GLM 提供 REST API 并使用 API 密钥
进行身份验证。在 Z.AI 控制台中创建你的 API 密钥。Moltbot 使用带有 Z.AI API 密钥的 `zai` 提供程序。

## CLI 设置

```bash
moltbot onboard --auth-choice zai-api-key
# 或非交互式
moltbot onboard --zai-api-key "$ZAI_API_KEY"
```

## 配置片段

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

## 注意事项

- GLM 模型作为 `zai/<model>` 可用（例如：`zai/glm-4.7`）。
- 有关模型系列概述，请参阅 [/providers/glm](/providers/glm)。
- Z.AI 使用带有 API 密钥的 Bearer 身份验证。
