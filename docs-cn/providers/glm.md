---
summary: "GLM 模型系列概述 + 如何在 Moltbot 中使用它"
read_when:
  - 你想在 Moltbot 中使用 GLM 模型
  - 你需要模型命名约定和设置
---
# GLM 模型

GLM 是一个通过 Z.AI 平台可用的**模型系列**（不是公司）。在 Moltbot 中，GLM 模型通过 `zai` 提供程序和模型 ID（如 `zai/glm-4.7`）访问。

## CLI 设置

```bash
moltbot onboard --auth-choice zai-api-key
```

## 配置片段

```json5
{
  env: { ZAI_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "zai/glm-4.7" } } }
}
```

## 注意事项

- GLM 版本和可用性可能会变化；查看 Z.AI 文档了解最新信息。
- 示例模型 ID 包括 `glm-4.7` 和 `glm-4.6`。
- 有关提供程序详细信息，请参阅 [/providers/zai](/providers/zai)。
