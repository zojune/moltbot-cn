---
title: "Vercel AI Gateway"
summary: "Vercel AI Gateway 设置（身份验证 + 模型选择）"
read_when:
  - 你想将 Vercel AI Gateway 与 Moltbot 一起使用
  - 你需要 API 密钥环境变量或 CLI 身份验证选择
---
# Vercel AI Gateway


[Vercel AI Gateway](https://vercel.com/ai-gateway) 提供统一 API 以通过单个端点访问数百个模型。

- 提供程序：`vercel-ai-gateway`
- 身份验证：`AI_GATEWAY_API_KEY`
- API：Anthropic Messages 兼容

## 快速开始

1) 设置 API 密钥（推荐：为网关存储它）：

```bash
moltbot onboard --auth-choice ai-gateway-api-key
```

2) 设置默认模型：

```json5
{
  agents: {
    defaults: {
      model: { primary: "vercel-ai-gateway/anthropic/claude-opus-4.5" }
    }
  }
}
```

## 非交互式示例

```bash
moltbot onboard --non-interactive \
  --mode local \
  --auth-choice ai-gateway-api-key \
  --ai-gateway-api-key "$AI_GATEWAY_API_KEY"
```

## 环境说明

如果网关作为守护程序（launchd/systemd）运行，请确保 `AI_GATEWAY_API_KEY`
对该进程可用（例如，在 `~/.clawdbot/.env` 或通过
`env.shellEnv`）。
