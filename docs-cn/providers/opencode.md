---
summary: "将 OpenCode Zen（精选模型）与 Moltbot 一起使用"
read_when:
  - 你想要 OpenCode Zen 进行模型访问
  - 你想要一个精选的编码友好模型列表
---
# OpenCode Zen

OpenCode Zen 是 OpenCode 团队推荐的**精选模型列表**，用于编码代理。
它是一个可选的托管模型访问路径，使用 API 密钥和 `opencode` 提供程序。
Zen 目前处于 beta 阶段。

## CLI 设置

```bash
moltbot onboard --auth-choice opencode-zen
# 或非交互式
moltbot onboard --opencode-zen-api-key "$OPENCODE_API_KEY"
```

## 配置片段

```json5
{
  env: { OPENCODE_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "opencode/claude-opus-4-5" } } }
}
```

## 注意事项

- 也支持 `OPENCODE_ZEN_API_KEY`。
- 你登录到 Zen，添加账单详情，然后复制你的 API 密钥。
- OpenCode Zen 按请求计费；查看 OpenCode 仪表板了解详情。
