---
summary: "Moltbot 从何处加载环境变量以及优先级顺序"
read_when:
  - 您需要知道加载了哪些环境变量，以及按什么顺序
  - 您正在调试 Gateway 中缺失的 API 密钥
  - 您正在记录提供商认证或部署环境
---

# 环境变量

Moltbot 从多个来源获取环境变量。规则是**永远不覆盖现有值**。

## 优先级（从高到低）

1) **进程环境**（Gateway 进程从父 shell/daemon 已经获得的环境变量）
2) **当前工作目录中的 `.env`**（dotenv 默认值；不覆盖）
3) **全局 `.env`** 位于 `~/.clawdbot/.env`（也称为 `$CLAWDBOT_STATE_DIR/.env`；不覆盖）
4) **`~/.clawdbot/moltbot.json` 中的配置 `env` 块**（仅在缺失时应用）
5) **可选的登录 shell 导入**（`env.shellEnv.enabled` 或 `CLAWDBOT_LOAD_SHELL_ENV=1`），仅对缺失的预期键应用

如果配置文件完全缺失，则跳过步骤 4；如果启用，shell 导入仍会运行。

## 配置 `env` 块

设置内联环境变量的两种等效方式（都是非覆盖的）：

```json5
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: {
      GROQ_API_KEY: "gsk-..."
    }
  }
}
```

## Shell 环境导入

`env.shellEnv` 运行您的登录 shell 并仅导入**缺失的**预期键：

```json5
{
  env: {
    shellEnv: {
      enabled: true,
      timeoutMs: 15000
    }
  }
}
```

环境变量等效项：
- `CLAWDBOT_LOAD_SHELL_ENV=1`
- `CLAWDBOT_SHELL_ENV_TIMEOUT_MS=15000`

## 配置中的环境变量替换

您可以使用 `${VAR_NAME}` 语法直接在配置字符串值中引用环境变量：

```json5
{
  models: {
    providers: {
      "vercel-gateway": {
        apiKey: "${VERCEL_GATEWAY_API_KEY}"
      }
    }
  }
}
```

有关完整详细信息，请参阅[配置：环境变量替换](/gateway/configuration#env-var-substitution-in-config)。

## 相关内容

- [Gateway 配置](/gateway/configuration)
- [FAQ：环境变量和 .env 加载](/help/faq#env-vars-and-env-loading)
- [模型概述](/concepts/models)
