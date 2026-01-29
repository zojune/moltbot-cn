---
summary: "通过 API 密钥或 setup-token 在 Moltbot 中使用 Anthropic Claude"
read_when:
  - 你想在 Moltbot 中使用 Anthropic 模型
  - 你想使用 setup-token 而不是 API 密钥
---
# Anthropic (Claude)

Anthropic 构建了 **Claude** 模型系列并通过 API 提供访问。
在 Moltbot 中，你可以使用 API 密钥或 **setup-token** 进行身份验证。

## 选项 A：Anthropic API 密钥

**最适合：** 标准 API 访问和按使用量计费。
在 Anthropic 控制台中创建你的 API 密钥。

### CLI 设置

```bash
moltbot onboard
# 选择：Anthropic API key

# 或非交互式
moltbot onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
```

### 配置片段

```json5
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

## 提示缓存 (Anthropic API)

除非你设置，否则 Moltbot **不会** 覆盖 Anthropic 的默认缓存 TTL。
这仅适用于 **API**；订阅身份验证不遵守 TTL 设置。

要为每个模型设置 TTL，请在模型 `params` 中使用 `cacheControlTtl`：

```json5
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": {
          params: { cacheControlTtl: "5m" } // 或 "1h"
        }
      }
    }
  }
}
```

Moltbot 在 Anthropic API 请求中包含 `extended-cache-ttl-2025-04-11` beta 标志；
如果你覆盖提供程序标头，请保留它（参见 [/gateway/configuration](/gateway/configuration)）。

## 选项 B：Claude setup-token

**最适合：** 使用你的 Claude 订阅。

### 在哪里获取 setup-token

Setup-token 由 **Claude Code CLI** 创建，而不是 Anthropic 控制台。你可以在**任何机器**上运行此命令：

```bash
claude setup-token
```

将 token 粘贴到 Moltbot（向导：**Anthropic token (paste setup-token)**），或在网关主机上运行：

```bash
moltbot models auth setup-token --provider anthropic
```

如果你在不同的机器上生成了 token，请粘贴它：

```bash
moltbot models auth paste-token --provider anthropic
```

### CLI 设置

```bash
# 在入职期间粘贴 setup-token
moltbot onboard --auth-choice setup-token
```

### 配置片段

```json5
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-5" } } }
}
```

## 注意事项

- 使用 `claude setup-token` 生成 setup-token 并粘贴它，或在网关主机上运行 `moltbot models auth setup-token`。
- 如果你在 Claude 订阅上看到 "OAuth token refresh failed ..."，请使用 setup-token 重新进行身份验证。参见 [/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription](/gateway/troubleshooting#oauth-token-refresh-failed-anthropic-claude-subscription)。
- 身份验证详情 + 重用规则在 [/concepts/oauth](/concepts/oauth) 中。

## 故障排除

**401 错误 / token 突然无效**
- Claude 订阅身份验证可能会过期或被撤销。在**网关主机**上重新运行 `claude setup-token` 并粘贴它。
- 如果 Claude CLI 登录位于不同的机器上，请在网关主机上使用 `moltbot models auth paste-token --provider anthropic`。

**No API key found for provider "anthropic"**
- 身份验证是**按代理**的。新代理不会继承主代理的密钥。
- 为该代理重新运行入职，或在网关主机上粘贴 setup-token / API 密钥，然后使用 `moltbot models status` 进行验证。

**No credentials found for profile `anthropic:default`**
- 运行 `moltbot models status` 查看哪个身份验证配置文件处于活动状态。
- 重新运行入职，或为该配置文件粘贴 setup-token / API 密钥。

**No available auth profile (all in cooldown/unavailable)**
- 检查 `moltbot models status --json` 中的 `auth.unusableProfiles`。
- 添加另一个 Anthropic 配置文件或等待冷却。

更多：[/gateway/troubleshooting](/gateway/troubleshooting) 和 [/help/faq](/help/faq)。
