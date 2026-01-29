---
summary: "Moltbot 如何轮换身份配置文件并在模型间回退"
read_when:
  - 诊断身份配置文件轮换、冷却或模型回退行为
  - 更新身份配置文件或模型的回退规则
---

# 模型回退

Moltbot 分两个阶段处理故障：
1) 当前提供者内的**身份配置文件轮换**。
2) **模型回退**到 `agents.defaults.model.fallbacks` 中的下一个模型。

本文档解释了运行时规则和支持它们的数据。

## 身份存储（密钥 + OAuth）

Moltbot 对 API 密钥和 OAuth 令牌都使用**身份配置文件**。

- 机密存在于 `~/.clawdbot/agents/<agentId>/agent/auth-profiles.json`（遗留：`~/.clawdbot/agent/auth-profiles.json`）。
- 配置 `auth.profiles` / `auth.order` 是**仅元数据和路由**（无机密）。
- 遗留仅导入 OAuth 文件：`~/.clawdbot/credentials/oauth.json`（首次使用时导入 `auth-profiles.json`）。

更多详情：[/concepts/oauth](/concepts/oauth)

凭据类型：
- `type: "api_key"` → `{ provider, key }`
- `type: "oauth"` → `{ provider, access, refresh, expires, email? }`（某些提供者的 `+ projectId`/`enterpriseUrl`）

## 配置文件 ID

OAuth 登录创建不同的配置文件，因此多个账户可以共存。
- 默认：没有电子邮件时为 `provider:default`。
- 带电子邮件的 OAuth：`provider:<email>`（例如 `google-antigravity:user@gmail.com`）。

配置文件存在于 `profiles` 下的 `~/.clawdbot/agents/<agentId>/agent/auth-profiles.json` 中。

## 轮换顺序

当提供者有多个配置文件时，Moltbot 按以下顺序选择：

1) **显式配置**：`auth.order[provider]`（如果设置）。
2) **配置的配置文件**：按提供者过滤的 `auth.profiles`。
3) **存储的配置文件**：提供者的 `auth-profiles.json` 中的条目。

如果没有配置显式顺序，Moltbot 使用循环顺序：
- **主键**：配置文件类型（**OAuth 在 API 密钥之前**）。
- **辅助键**：`usageStats.lastUsed`（每种类型中最先的，最旧的优先）。
- **冷却/禁用的配置文件**移动到末尾，按最早到期排序。

### 会话粘性（缓存友好）

Moltbot **为每个会话固定所选的身份配置文件**以保持提供者缓存温暖。它**不**在每个请求上轮换。固定的配置文件被重用，直到：
- 会话被重置（`/new` / `/reset`）
- 压缩完成（压缩计数递增）
- 配置文件处于冷却/禁用状态

通过 `/model …@<profileId>` 手动选择为该会话设置**用户覆盖**，并且在新会话开始之前不会自动轮换。

自动固定的配置文件（由会话路由器选择）被视为**偏好**：它们首先尝试，但 Moltbot 在速率限制/超时可能会轮换到另一个配置文件。用户固定的配置文件保持锁定到该配置文件；如果失败并配置了模型回退，Moltbot 会移动到下一个模型而不是切换配置文件。

### 为什么 OAuth 可能"看起来丢失"

如果你同时拥有同一提供者的 OAuth 配置文件和 API 密钥配置文件，除非固定，否则跨消息可以在它们之间切换。要强制单个配置文件：
- 使用 `auth.order[provider] = ["provider:profileId"]` 固定，或
- 通过 `/model …` 使用每会话覆盖，并带有配置文件覆盖（当你的 UI/聊天表面支持时）。

## 冷却

当配置文件由于身份/速率限制错误（或看起来像速率限制的超时）而失败时，Moltbot 将其标记为冷却并移动到下一个配置文件。格式/无效请求错误（例如 Cloud Code Assist 工具调用 ID 验证失败）被视为可回退并使用相同的冷却。

冷却使用指数退避：
- 1 分钟
- 5 分钟
- 25 分钟
- 1 小时（上限）

状态存储在 `auth-profiles.json` 的 `usageStats` 下：

```json
{
  "usageStats": {
    "provider:profile": {
      "lastUsed": 1736160000000,
      "cooldownUntil": 1736160600000,
      "errorCount": 2
    }
  }
}
```

## 计费禁用

计费/信用失败（例如"信用不足"/"信用余额过低"）被视为可回退，但它们通常不是瞬态的。Moltbot 不是短暂冷却，而是将配置文件标记为**禁用**（具有更长的退避）并轮换到下一个配置文件/提供者。

状态存储在 `auth-profiles.json` 中：

```json
{
  "usageStats": {
    "provider:profile": {
      "disabledUntil": 1736178000000,
      "disabledReason": "billing"
    }
  }
}
```

默认值：
- 计费退避从 **5 小时**开始，每次计费失败加倍，上限为 **24 小时**。
- 如果配置文件在 **24 小时**内未失败，退避计数器重置（可配置）。

## 模型回退

如果提供者的所有配置文件都失败，Moltbot 会移动到 `agents.defaults.model.fallbacks` 中的下一个模型。这适用于身份失败、速率限制和耗尽配置文件轮换的超时（其他错误不会推进回退）。

当运行以模型覆盖开始（钩子或 CLI）时，回退仍然在尝试任何配置的回退后结束于 `agents.defaults.model.primary`。

## 相关配置

有关以下内容，请参见 [Gateway 配置](/gateway/configuration)：
- `auth.profiles` / `auth.order`
- `auth.cooldowns.billingBackoffHours` / `auth.cooldowns.billingBackoffHoursByProvider`
- `auth.cooldowns.billingMaxHours` / `auth.cooldowns.failureWindowHours`
- `agents.defaults.model.primary` / `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel` 路由

有关更广泛的模型选择和回退概述，请参见 [模型](/concepts/models)。
