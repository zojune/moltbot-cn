---
summary: "Moltbot 支持的模型提供程序 (LLM)"
read_when:
  - 你想选择一个模型提供程序
  - 你想要 LLM 身份验证 + 模型选择的快速设置示例
---
# 模型提供程序

Moltbot 可以使用许多 LLM 提供程序。选择一个，进行身份验证，然后将默认模型设置为 `provider/model`。

## 亮点：Venius (Venice AI)

Venius 是我们推荐的 Venice AI 设置，用于以隐私为重点的推理，并可选择使用 Opus 处理最困难的任务。

- 默认：`venice/llama-3.3-70b`
- 最佳整体：`venice/claude-opus-45`（Opus 仍然是最强的）

请参阅 [Venice AI](/providers/venice)。

## 快速开始（两步）

1) 通过提供程序进行身份验证（通常通过 `moltbot onboard`）。
2) 设置默认模型：

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

## 支持的提供程序（入门套件）

- [OpenAI (API + Codex)](/providers/openai)
- [Anthropic (API + Claude Code CLI)](/providers/anthropic)
- [OpenRouter](/providers/openrouter)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Moonshot AI (Kimi + Kimi Code)](/providers/moonshot)
- [Synthetic](/providers/synthetic)
- [OpenCode Zen](/providers/opencode)
- [Z.AI](/providers/zai)
- [GLM 模型](/providers/glm)
- [MiniMax](/providers/minimax)
- [Venius (Venice AI)](/providers/venice)
- [Amazon Bedrock](/bedrock)

有关完整的提供程序目录（xAI、Groq、Mistral 等）和高级配置，请参阅 [模型提供程序](/concepts/model-providers)。
