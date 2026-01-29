---
summary: "在本地 LLM 上运行 Moltbot(LM Studio、vLLM、LiteLLM、自定义 OpenAI 端点)"
read_when:
  - 您想从自己的 GPU 盒提供模型
  - 您正在连接 LM Studio 或 OpenAI 兼容的代理
  - 您需要最安全的本地模型指导
---
# 本地模型

本地是可行的,但 Moltbot 需要大上下文 + 强大的提示注入防御。小型显卡会截断上下文并泄漏安全性。目标要高:**≥2 个最大化的 Mac Studio 或等效的 GPU 设备(~$30k+)**。单个 **24 GB** GPU 仅适用于延迟较高的较轻提示。使用**您可以运行的最大/完整大小的模型变体**;激进的量化或"小"检查点会增加提示注入风险(参见 [Security](/gateway/security))。

## 推荐:LM Studio + MiniMax M2.1(Responses API,完整大小)

当前最好的本地堆栈。在 LM Studio 中加载 MiniMax M2.1,启用本地服务器(默认 `http://127.0.0.1:1234`),并使用 Responses API 将推理与最终文本分离。

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: {
        "anthropic/claude-opus-4-5": { alias: "Opus" },
        "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

**设置检查清单**
- 安装 LM Studio: https://lmstudio.ai
- 在 LM Studio 中,下载**可用的最大 MiniMax M2.1 构建**(避免"小"/重度量化变体),启动服务器,确认 `http://127.0.0.1:1234/v1/models` 列出它。
- 保持模型加载;冷加载会增加启动延迟。
- 如果您的 LM Studio 构建不同,请调整 `contextWindow`/`maxTokens`。
- 对于 WhatsApp,坚持使用 Responses API,以便仅发送最终文本。

即使在运行本地时也要配置托管模型;使用 `models.mode: "merge"` 以便后备保持可用。

### 混合配置:托管主要,本地后备

```json5
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["lmstudio/minimax-m2.1-gs32", "anthropic/claude-opus-4-5"]
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "lmstudio/minimax-m2.1-gs32": { alias: "MiniMax Local" },
        "anthropic/claude-opus-4-5": { alias: "Opus" }
      }
    }
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.1-gs32",
            name: "MiniMax M2.1 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

### 本地优先,托管安全网

交换主要和后备顺序;保持相同的 providers 块和 `models.mode: "merge"`,以便您可以在本地设备宕机时回退到 Sonnet 或 Opus。

### 区域托管/数据路由

- 托管的 MiniMax/Kimi/GLM 变体也存在于 OpenRouter 上,具有区域固定端点(例如,美国托管)。在那里选择区域变体以使流量保持在您选择的管辖区,同时仍然使用 `models.mode: "merge"` 进行 Anthropic/OpenAI 后备。
- 仅本地仍然是最强的隐私路径;托管区域路由是当您需要提供商功能但想要控制数据流时的中间立场。

## 其他 OpenAI 兼容的本地代理

vLLM、LiteLLM、OAI-proxy 或自定义 gateway 如果它们暴露 OpenAI 风格的 `/v1` 端点就可以工作。将上面的 provider 块替换为您的端点和模型 ID:

```json5
{
  models: {
    mode: "merge",
    providers: {
      local: {
        baseUrl: "http://127.0.0.1:8000/v1",
        apiKey: "sk-local",
        api: "openai-responses",
        models: [
          {
            id: "my-local-model",
            name: "Local Model",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 120000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

保持 `models.mode: "merge"` 以便托管模型作为后备保持可用。

## 故障排除
- Gateway 可以到达代理?`curl http://127.0.0.1:1234/v1/models`。
- LM Studio 模型未加载?重新加载;冷启动是常见的"挂起"原因。
- 上下文错误?降低 `contextWindow` 或提高服务器限制。
- 安全性:本地模型跳过提供商端过滤器;保持代理狭窄并启用压缩以限制提示注入爆炸半径。
