---
summary: "web_fetch 的 Firecrawl 回退（反机器人 + 缓存提取）"
read_when:
  - 您想要基于 Firecrawl 的 web 提取
  - 您需要 Firecrawl API 密钥
  - 您想要 web_fetch 的反机器人提取
---

# Firecrawl

Moltbot 可以使用 **Firecrawl** 作为 `web_fetch` 的回退提取器。它是一个托管的内容提取服务，支持机器人规避和缓存，这有助于处理重度 JS 的站点或阻止纯 HTTP 获取的页面。

## 获取 API 密钥

1) 创建 Firecrawl 帐户并生成 API 密钥。
2) 将其存储在配置中或在网关环境中设置 `FIRECRAWL_API_KEY`。

## 配置 Firecrawl

```json5
{
  tools: {
    web: {
      fetch: {
        firecrawl: {
          apiKey: "FIRECRAWL_API_KEY_HERE",
          baseUrl: "https://api.firecrawl.dev",
          onlyMainContent: true,
          maxAgeMs: 172800000,
          timeoutSeconds: 60
        }
      }
    }
  }
}
```

说明：
- 当存在 API 密钥时，`firecrawl.enabled` 默认为 true。
- `maxAgeMs` 控制缓存结果的最旧时间（毫秒）。默认为 2 天。

## 隐身 / 机器人规避

Firecrawl 为机器人规避公开一个**代理模式**参数（`basic`、`stealth` 或 `auto`）。Moltbot 始终对 Firecrawl 请求使用 `proxy: "auto"` 加上 `storeInCache: true`。如果省略代理，Firecrawl 默认为 `auto`。`auto` 在基本尝试失败时使用隐身代理重试，这可能比仅基本抓取使用更多积分。

## `web_fetch` 如何使用 Firecrawl

`web_fetch` 提取顺序：
1) Readability（本地）
2) Firecrawl（如果配置）
3) 基本 HTML 清理（最后回退）

有关完整的 web 工具设置，请参阅 [Web 工具](/tools/web)。
