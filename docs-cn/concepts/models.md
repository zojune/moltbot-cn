---
summary: "Models CLI：list、set、aliases、fallbacks、scan、status"
read_when:
  - 添加或修改 models CLI（models list/set/scan/aliases/fallbacks）
  - 更改模型回退行为或选择 UX
  - 更新模型扫描探测（工具/图像）
---
# Models CLI

有关身份配置文件轮换、冷却以及它如何与回退交互，请参见 [/concepts/model-failover](/concepts/model-failover)。
快速提供者概述 + 示例：[/concepts/model-providers](/concepts/model-providers)。

## 模型选择如何工作

Moltbot 按以下顺序选择模型：

1) **主**模型（`agents.defaults.model.primary` 或 `agents.defaults.model`）。
2) `agents.defaults.model.fallbacks` 中的**回退**（按顺序）。
3) **提供者身份回退**在移动到下一个模型之前在提供者内发生。

相关内容：
- `agents.defaults.models` 是 Moltbot 可以使用的模型的允许列表/目录（加上别名）。
- `agents.defaults.imageModel` **仅在**主模型无法接受图像时使用。
- 每个代理的默认值可以通过 `agents.list[].model` 加上绑定覆盖 `agents.defaults.model`（参见 [/concepts/multi-agent](/concepts/multi-agent)）。

## 快速模型选择（经验）

- **GLM**：对于编码/工具调用稍好。
- **MiniMax**：更适合写作和氛围。

## 设置向导（推荐）

如果你不想手动编辑配置，请运行入门向导：

```bash
moltbot onboard
```

它可以为常见提供者设置模型 + 身份，包括 **OpenAI Code (Codex) 订阅**（OAuth）和 **Anthropic**（推荐 API 密钥；也支持 `claude setup-token`）。

## 配置键（概述）

- `agents.defaults.model.primary` 和 `agents.defaults.model.fallbacks`
- `agents.defaults.imageModel.primary` 和 `agents.defaults.imageModel.fallbacks`
- `agents.defaults.models`（允许列表 + 别名 + 提供者参数）
- `models.providers`（写入 `models.json` 的自定义提供者）

模型引用规范化为小写。提供者别名如 `z.ai/*` 规范化为 `zai/*`。

提供者配置示例（包括 OpenCode Zen）位于 [/gateway/configuration](/gateway/configuration#opencode-zen-multi-model-proxy)。

## "Model is not allowed"（以及为什么回复停止）

如果设置了 `agents.defaults.models`，它将成为 `/model` 和会话覆盖的**允许列表**。当用户选择不在该允许列表中的模型时，Moltbot 返回：

```
Model "provider/model" is not allowed. Use /model to list available models.
```

这发生在生成正常回复**之前**，因此消息可能感觉像"没有响应"。解决方法是：
- 将模型添加到 `agents.defaults.models`，或
- 清除允许列表（删除 `agents.defaults.models`），或
- 从 `/model list` 中选择一个模型。

允许列表配置示例：

```json5
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-5": { alias: "Opus" }
    }
  }
}
```

## 在聊天中切换模型（`/model`）

你可以在不重启的情况下为当前会话切换模型：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

注意事项：
- `/model`（和 `/model list`）是一个紧凑的编号选择器（模型系列 + 可用提供者）。
- `/model <#>` 从该选择器中选择。
- `/model status` 是详细视图（身份候选者，以及配置时的提供者端点 `baseUrl` + `api` 模式）。
- 模型引用通过在**第一个** `/` 上拆分来解析。键入 `/model <ref>` 时使用 `provider/model`。
- 如果模型 ID 本身包含 `/`（OpenRouter 风格），你必须包含提供者前缀（例如：`/model openrouter/moonshotai/kimi-k2`）。
- 如果你省略提供者，Moltbot 会将输入视为别名或**默认提供者**的模型（仅在模型 ID 中没有 `/` 时有效）。

完整命令行为/配置：[斜杠命令](/tools/slash-commands)。

## CLI 命令

```bash
moltbot models list
moltbot models status
moltbot models set <provider/model>
moltbot models set-image <provider/model>

moltbot models aliases list
moltbot models aliases add <alias> <provider/model>
moltbot models aliases remove <alias>

moltbot models fallbacks list
moltbot models fallbacks add <provider/model>
moltbot models fallbacks remove <provider/model>
moltbot models fallbacks clear

moltbot models image-fallbacks list
moltbot models image-fallbacks add <provider/model>
moltbot models image-fallbacks remove <provider/model>
moltbot models image-fallbacks clear
```

`moltbot models`（无子命令）是 `models status` 的快捷方式。

### `models list`

默认显示配置的模型。有用的标志：
- `--all`：完整目录
- `--local`：仅本地提供者
- `--provider <name>`：按提供者过滤
- `--plain`：每行一个模型
- `--json`：机器可读输出

### `models status`

显示解析的主模型、回退、图像模型以及配置的提供者的身份概述。它还显示身份存储中找到的配置文件的 OAuth 到期状态（默认在 24h 内警告）。`--plain` 仅打印解析的主模型。
OAuth 状态始终显示（并包含在 `--json` 输出中）。如果配置的提供者没有凭据，`models status` 会打印**Missing auth**部分。
JSON 包括 `auth.oauth`（警告窗口 + 配置文件）和 `auth.providers`（每个提供者的有效身份）。
使用 `--check` 进行自动化（缺少/过期时退出 `1`，即将到期时退出 `2`）。

首选的 Anthropic 身份是 Claude Code CLI setup-token（在任何地方运行；如果需要，在网关主机上粘贴）：

```bash
claude setup-token
moltbot models status
```

## 扫描（OpenRouter 免费模型）

`moltbot models scan` 检查 OpenRouter 的**免费模型目录**，并可以选择探测模型的工具和图像支持。

关键标志：
- `--no-probe`：跳过实时探测（仅元数据）
- `--min-params <b>`：最小参数大小（十亿）
- `--max-age-days <days>`：跳过较旧的模型
- `--provider <name>`：提供者前缀过滤器
- `--max-candidates <n>`：回退列表大小
- `--set-default`：将 `agents.defaults.model.primary` 设置为第一个选择
- `--set-image`：将 `agents.defaults.imageModel.primary` 设置为第一个图像选择

探测需要 OpenRouter API 密钥（来自身份配置文件或 `OPENROUTER_API_KEY`）。没有密钥，使用 `--no-probe` 仅列出候选者。

扫描结果按以下方式排名：
1) 图像支持
2) 工具延迟
3) 上下文大小
4) 参数数量

输入
- OpenRouter `/models` 列表（过滤器 `:free`）
- 需要来自身份配置文件或 `OPENROUTER_API_KEY` 的 OpenRouter API 密钥（参见 [/environment](/environment)）
- 可选过滤器：`--max-age-days`、`--min-params`、`--provider`、`--max-candidates`
- 探测控制：`--timeout`、`--concurrency`

在 TTY 中运行时，你可以交互式地选择回退。在非交互模式下，传递 `--yes` 以接受默认值。

## 模型注册表（`models.json`）

`models.providers` 中的自定义提供者被写入代理目录下的 `models.json`（默认 `~/.clawdbot/agents/<agentId>/models.json`）。此文件默认合并，除非 `models.mode` 设置为 `replace`。
