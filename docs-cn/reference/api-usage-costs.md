---
summary: "审核哪些功能可以花钱、使用哪些密钥以及如何查看使用情况"
read_when:
  - 您想了解哪些功能可能调用付费 API
  - 您需要审核密钥、成本和使用可见性
  - 您正在解释 /status 或 /usage 成本报告
---
# API 使用和成本

本文档列出了**可以调用 API 密钥的功能**及其成本显示位置。它侧重于
可以生成提供商使用或付费 API 调用的 Moltbot 功能。

## 成本显示位置（聊天 + CLI）

**每会话成本快照**
- `/status` 显示当前会话模型、上下文使用情况和上次响应的令牌。
- 如果模型使用 **API 密钥认证**，`/status` 还会显示上次回复的**估算成本**。

**每条消息成本页脚**
- `/usage full` 在每条回复后附加使用情况页脚，包括**估算成本**（仅限 API 密钥）。
- `/usage tokens` 仅显示令牌；OAuth 流程隐藏美元成本。

**CLI 使用情况窗口（提供商配额）**
- `moltbot status --usage` 和 `moltbot channels list` 显示提供商**使用情况窗口**
  （配额快照，而不是每条消息的成本）。

有关详细信息和示例，请参阅 [令牌使用和成本](/token-use)。

## 密钥的发现方式

Moltbot 可以从以下位置获取凭据：
- **认证配置文件**（每个代理，存储在 `auth-profiles.json` 中）。
- **环境变量**（例如 `OPENAI_API_KEY`、`BRAVE_API_KEY`、`FIRECRAWL_API_KEY`）。
- **配置**（`models.providers.*.apiKey`、`tools.web.search.*`、`tools.web.fetch.firecrawl.*`、
  `memorySearch.*`、`talk.apiKey`）。
- **技能**（`skills.entries.<name>.apiKey`），可能会将密钥导出到技能进程环境。

## 可以使用密钥的功能

### 1) 核心模型响应（聊天 + 工具）
每个回复或工具调用都使用**当前模型提供商**（OpenAI、Anthropic 等）。这是
使用和成本的主要来源。

有关定价配置，请参阅 [模型](/providers/models)，有关显示，请参阅 [令牌使用和成本](/token-use)。

### 2) 媒体理解（音频/图像/视频）
入站媒体可以在回复运行之前进行摘要/转录。这会使用模型/提供商 API。

- 音频：OpenAI / Groq / Deepgram（当密钥存在时现在**自动启用**）。
- 图像：OpenAI / Anthropic / Google。
- 视频：Google。

请参阅 [媒体理解](/nodes/media-understanding)。

### 3) 内存嵌入 + 语义搜索
当为远程提供商配置时，语义内存搜索使用**嵌入 API**：
- `memorySearch.provider = "openai"` → OpenAI 嵌入
- `memorySearch.provider = "gemini"` → Gemini 嵌入
- 如果本地嵌入失败，可选回退到 OpenAI

您可以使用 `memorySearch.provider = "local"` 使其保持本地（无 API 使用）。

请参阅 [内存](/concepts/memory)。

### 4) Web 搜索工具（Brave / Perplexity 通过 OpenRouter）
`web_search` 使用 API 密钥并可能产生使用费用：

- **Brave Search API**：`BRAVE_API_KEY` 或 `tools.web.search.apiKey`
- **Perplexity**（通过 OpenRouter）：`PERPLEXITY_API_KEY` 或 `OPENROUTER_API_KEY`

**Brave 免费层（慷慨）：**
- **每月 2,000 次请求**
- **每秒 1 次请求**
- **需要信用卡**进行验证（除非您升级，否则不收费）

请参阅 [Web 工具](/tools/web)。

### 5) Web 获取工具（Firecrawl）
当存在 API 密钥时，`web_fetch` 可以调用 **Firecrawl**：
- `FIRECRAWL_API_KEY` 或 `tools.web.fetch.firecrawl.apiKey`

如果未配置 Firecrawl，该工具将回退到直接获取 + 可读性（无付费 API）。

请参阅 [Web 工具](/tools/web)。

### 6) 提供商使用快照（状态/健康）
某些状态命令调用**提供商使用端点**以显示配额窗口或认证运行状况。
这些通常是低量调用，但仍会访问提供商 API：
- `moltbot status --usage`
- `moltbot models status --json`

请参阅 [模型 CLI](/cli/models)。

### 7) 压缩保护摘要
压缩保护可以使用**当前模型**汇总会话历史，这会在
它运行时调用提供商 API。

请参阅 [会话管理 + 压缩](/reference/session-management-compaction)。

### 8) 模型扫描/探测
`moltbot models scan` 可以探测 OpenRouter 模型，并在启用探测时使用 `OPENROUTER_API_KEY`。

请参阅 [模型 CLI](/cli/models)。

### 9) 说话（语音）
说话模式在配置时可以调用 **ElevenLabs**：
- `ELEVENLABS_API_KEY` 或 `talk.apiKey`

请参阅 [说话模式](/nodes/talk)。

### 10) 技能（第三方 API）
技能可以在 `skills.entries.<name>.apiKey` 中存储 `apiKey`。如果技能将该密钥用于外部
API，则可能会根据技能的提供商产生费用。

请参阅 [技能](/tools/skills)。
