---
summary: "在 Moltbot 中使用 MiniMax M2.1"
read_when:
  - 你想在 Moltbot 中使用 MiniMax 模型
  - 你需要 MiniMax 设置指导
---
# MiniMax

MiniMax 是一家构建 **M2/M2.1** 模型系列的 AI 公司。当前专注于编码的版本是 **MiniMax M2.1**（2025 年 12 月 23 日），专为现实世界的复杂任务而构建。

来源：[MiniMax M2.1 发布说明](https://www.minimax.io/news/minimax-m21)

## 模型概述 (M2.1)

MiniMax 在 M2.1 中强调了这些改进：

- 更强的**多语言编码**（Rust、Java、Go、C++、Kotlin、Objective-C、TS/JS）。
- 更好的**网络/应用开发**和美观的输出质量（包括原生移动）。
- 改进的**复合指令**处理，用于办公风格的工作流程，建立在交错思维和集成约束执行的基础上。
- **更简洁的响应**，具有更低的令牌使用和更快的迭代循环。
- 更强的**工具/代理框架**兼容性和上下文管理（Claude Code、Droid/Factory AI、Cline、Kilo Code、Roo Code、BlackBox）。
- 更高质量的**对话和技术写作**输出。

## MiniMax M2.1 vs MiniMax M2.1 Lightning

- **速度：** Lightning 是 MiniMax 定价文档中的"快速"变体。
- **成本：** 定价显示相同的输入成本，但 Lightning 具有更高的输出成本。
- **编码计划路由：** Lightning 后端在 MiniMax 编码计划上不直接可用。MiniMax 会自动将大多数请求路由到 Lightning，但在流量高峰期间回退到常规 M2.1 后端。

## 选择设置

### MiniMax M2.1 — 推荐

**最适合：** 通过 Anthropic 兼容 API 托管 MiniMax。

通过 CLI 配置：
- 运行 `moltbot configure`
- 选择 **Model/auth**
- 选择 **MiniMax M2.1**

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.1" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.1",
            name: "MiniMax M2.1",
            reasoning: false,
            input: ["text"],
            cost: { input: 15, output: 60, cacheRead: 2, cacheWrite: 10 },
            contextWindow: 200000,
            maxTokens: 8192
          }
        ]
      }
    }
  }
}
```

### MiniMax M2.1 作为后备（Opus 主用）

**最适合：** 保持 Opus 4.5 为主用，回退到 MiniMax M2.1。

```json5
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-5": { alias: "opus" },
        "minimax/MiniMax-M2.1": { alias: "minimax" }
      },
      model: {
        primary: "anthropic/claude-opus-4-5",
        fallbacks: ["minimax/MiniMax-M2.1"]
      }
    }
  }
}
```

### 可选：通过 LM Studio 本地（手动）

**最适合：** 使用 LM Studio 进行本地推理。
我们在强大的硬件（例如桌面/服务器）上使用 LM Studio 的本地服务器，通过 MiniMax M2.1 看到了强大的结果。

通过 `moltbot.json` 手动配置：

```json5
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.1-gs32" },
      models: { "lmstudio/minimax-m2.1-gs32": { alias: "Minimax" } }
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

## 通过 `moltbot configure` 配置

使用交互式配置向导设置 MiniMax 而无需编辑 JSON：

1) 运行 `moltbot configure`。
2) 选择 **Model/auth**。
3) 选择 **MiniMax M2.1**。
4) 提示时选择你的默认模型。

## 配置选项

- `models.providers.minimax.baseUrl`：首选 `https://api.minimax.io/anthropic`（Anthropic 兼容）；`https://api.minimax.io/v1` 是 OpenAI 兼容负载的可选选项。
- `models.providers.minimax.api`：首选 `anthropic-messages`；`openai-completions` 是 OpenAI 兼容负载的可选选项。
- `models.providers.minimax.apiKey`：MiniMax API 密钥（`MINIMAX_API_KEY`）。
- `models.providers.minimax.models`：定义 `id`、`name`、`reasoning`、`contextWindow`、`maxTokens`、`cost`。
- `agents.defaults.models`：为你想要在允许列表中的模型设置别名。
- `models.mode`：如果你想将 MiniMax 与内置模型一起添加，请保持 `merge`。

## 注意事项

- 模型引用是 `minimax/<model>`。
- 编码计划使用 API：`https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains`（需要编码计划密钥）。
- 如果需要准确的成本跟踪，请更新 `models.json` 中的定价值。
- MiniMax 编码计划的推荐链接（10% 折扣）：https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link
- 有关提供程序规则，请参阅 [/concepts/model-providers](/concepts/model-providers)。
- 使用 `moltbot models list` 和 `moltbot models set minimax/MiniMax-M2.1` 进行切换。

## 故障排除

### "Unknown model: minimax/MiniMax-M2.1"

这通常意味着**未配置 MiniMax 提供程序**（没有提供程序条目，也未找到 MiniMax 身份验证配置文件/环境密钥）。针对此检测的修复在 **2026.1.12** 中（撰写时未发布）。修复方法：
- 升级到 **2026.1.12**（或从源代码 `main` 运行），然后重启网关。
- 运行 `moltbot configure` 并选择 **MiniMax M2.1**，或
- 手动添加 `models.providers.minimax` 块，或
- 设置 `MINIMAX_API_KEY`（或 MiniMax 身份验证配置文件）以便注入提供程序。

确保模型 ID **区分大小写**：
- `minimax/MiniMax-M2.1`
- `minimax/MiniMax-M2.1-lightning`

然后重新检查：
```bash
moltbot models list
```
