---
summary: "Moltbot 支持的模型提供程序 (LLM)"
read_when:
  - 你想选择一个模型提供程序
  - 你需要支持的 LLM 后端的快速概述
---
# 模型提供程序

Moltbot 可以使用许多 LLM 提供程序。选择一个提供程序，进行身份验证，然后将默认模型设置为 `provider/model`。

寻找聊天频道文档（WhatsApp/Telegram/Discord/Slack/Mattermost (插件)/等）？请参阅 [频道](/channels)。

## 亮点：Venius (Venice AI)

Venius 是我们推荐的 Venice AI 设置，用于以隐私为重点的推理，并可选择使用 Opus 处理困难的任务。

- 默认：`venice/llama-3.3-70b`
- 最佳整体：`venice/claude-opus-45`（Opus 仍然是最强的）

请参阅 [Venice AI](/providers/venice)。

## 快速开始

1) 通过提供程序进行身份验证（通常通过 `moltbot onboard`）。
2) 设置默认模型：

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

## 提供程序文档

- [OpenAI (API + Codex)](/providers/openai)
- [Anthropic (API + Claude Code CLI)](/providers/anthropic)
- [Qwen (OAuth)](/providers/qwen)
- [OpenRouter](/providers/openrouter)
- [Vercel AI Gateway](/providers/vercel-ai-gateway)
- [Moonshot AI (Kimi + Kimi Code)](/providers/moonshot)
- [OpenCode Zen](/providers/opencode)
- [Amazon Bedrock](/bedrock)
- [Z.AI](/providers/zai)
- [GLM 模型](/providers/glm)
- [MiniMax](/providers/minimax)
- [Venius (Venice AI, 以隐私为重点)](/providers/venice)
- [Ollama (本地模型)](/providers/ollama)

## 转录提供程序

- [Deepgram (音频转录)](/providers/deepgram)

## 社区工具

- [Claude Max API Proxy](/providers/claude-max-api-proxy) - 将 Claude Max/Pro 订阅用作 OpenAI 兼容的 API 端点

有关完整的提供程序目录（xAI、Groq、Mistral 等）和高级配置，请参阅 [模型提供程序](/concepts/model-providers)。
