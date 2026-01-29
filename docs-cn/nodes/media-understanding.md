---
summary: "入站图像/音频/视频理解（可选），带有提供商 + CLI 后备"
read_when:
  - 设计或重构媒体理解
  - 调整入站音频/视频/图像预处理
---
# 媒体理解（入站）— 2026-01-17

Moltbot 可以在回复管道运行之前**总结入站媒体**（图像/音频/视频）。它会在本地工具或提供商密钥可用时自动检测，可以禁用或自定义。如果理解关闭，模型仍会像往常一样接收原始文件/URL。

## 目标
- 可选：将入站媒体预先摘要为短文本，以便更快路由 + 更好的命令解析。
- 保留原始媒体传递到模型（始终）。
- 支持**提供商 API** 和 **CLI 后备**。
- 允许多个模型按顺序后备（错误/大小/超时）。

## 高级行为
1) 收集入站附件（`MediaPaths`、`MediaUrls`、`MediaTypes`）。
2) 对于每个启用的功能（图像/音频/视频），按策略选择附件（默认：**第一个**）。
3) 选择第一个符合条件的模型条目（大小 + 功能 + 认证）。
4) 如果模型失败或媒体太大，**回退到下一个条目**。
5) 成功时：
   - `Body` 变为 `[Image]`、`[Audio]` 或 `[Video]` 块。
   - 音频设置 `{{Transcript}}`；命令解析在存在时使用字幕文本，
     否则使用转录。
   - 字幕被保留为块内的 `User text:`。

如果理解失败或被禁用，**回复流程继续**使用原始正文 + 附件。

## 配置概述
`tools.media` 支持**共享模型**加上每个功能的覆盖：
- `tools.media.models`：共享模型列表（使用 `capabilities` 进行限制）。
- `tools.media.image` / `tools.media.audio` / `tools.media.video`：
  - 默认值（`prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`）
  - 提供商覆盖（`baseUrl`、`headers`、`providerOptions`）
  - 通过 `tools.media.audio.providerOptions.deepgram` 的 Deepgram 音频选项
  - 可选的**每个功能 `models` 列表**（在共享模型之前首选）
  - `attachments` 策略（`mode`、`maxAttachments`、`prefer`）
  - `scope`（可选的频道/chatType/会话键限制）
- `tools.media.concurrency`：最大并发功能运行（默认 **2**）。

```json5
{
  tools: {
    media: {
      models: [ /* 共享列表 */ ],
      image: { /* 可选覆盖 */ },
      audio: { /* 可选覆盖 */ },
      video: { /* 可选覆盖 */ }
    }
  }
}
```

### 模型条目
每个 `models[]` 条目可以是**提供商**或 **CLI**：

```json5
{
  type: "provider",        // 如果省略则默认
  provider: "openai",
  model: "gpt-5.2",
  prompt: "在 <= 500 字符内描述图像。",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // 可选，用于多模态条目
  profile: "vision-profile",
  preferredProfile: "vision-fallback"
}
```

```json5
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "阅读 {{MediaPath}} 处的媒体并在 <= {{MaxChars}} 字符内描述它。"
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"]
}
```

CLI 模板也可以使用：
- `{{MediaDir}}`（包含媒体文件的目录）
- `{{OutputDir}}`（为此运行创建的临时目录）
- `{{OutputBase}}`（临时文件基本路径，无扩展名）

## 默认值和限制
推荐的默认值：
- `maxChars`：图像/视频为 **500**（短，命令友好）
- `maxChars`：音频**未设置**（完整转录，除非你设置限制）
- `maxBytes`：
  - 图像：**10MB**
  - 音频：**20MB**
  - 视频：**50MB**

规则：
- 如果媒体超过 `maxBytes`，则跳过该模型并**尝试下一个模型**。
- 如果模型返回超过 `maxChars`，输出将被修剪。
- `prompt` 默认为简单的"描述 {media}。"加上 `maxChars` 指导（仅图像/视频）。
- 如果 `<capability>.enabled: true` 但未配置模型，当其提供商支持该功能时，Moltbot 会尝试**活动回复模型**。

### 自动检测媒体理解（默认）
如果 `tools.media.<capability>.enabled` **未**设置为 `false` 且你尚未配置模型，Moltbot 会按以下顺序自动检测，并在第一个可用选项处**停止**：

1) **本地 CLI**（仅音频；如果已安装）
   - `sherpa-onnx-offline`（需要 `SHERPA_ONNX_MODEL_DIR`，包含 encoder/decoder/joiner/tokens）
   - `whisper-cli`（`whisper-cpp`；使用 `WHISPER_CPP_MODEL` 或捆绑的 tiny 模型）
   - `whisper`（Python CLI；自动下载模型）
2) **Gemini CLI**（`gemini`）使用 `read_many_files`
3) **提供商密钥**
   - 音频：OpenAI → Groq → Deepgram → Google
   - 图像：OpenAI → Anthropic → Google → MiniMax
   - 视频：Google

要禁用自动检测，设置：
```json5
{
  tools: {
    media: {
      audio: {
        enabled: false
      }
    }
  }
}
```
注意：二进制检测在 macOS/Linux/Windows 上尽力而为；确保 CLI 在 `PATH` 上（我们会展开 `~`），或使用完整命令路径设置显式 CLI 模型。

## 功能（可选）
如果你设置 `capabilities`，该条目仅针对那些媒体类型运行。对于共享列表，Moltbot 可以推断默认值：
- `openai`、`anthropic`、`minimax`：**图像**
- `google`（Gemini API）：**图像 + 音频 + 视频**
- `groq`：**音频**
- `deepgram`：**音频**

对于 CLI 条目，**显式设置 `capabilities`** 以避免意外匹配。
如果省略 `capabilities`，该条目有资格出现在其列表中。

## 提供商支持矩阵（Moltbot 集成）
| 功能 | 提供商集成 | 注意 |
|------------|----------------------|-------|
| 图像 | OpenAI / Anthropic / Google / 其他通过 `pi-ai` | 注册表中任何支持图像的模型都有效。 |
| 音频 | OpenAI、Groq、Deepgram、Google | 提供商转录（Whisper/Deepgram/Gemini）。 |
| 视频 | Google（Gemini API） | 提供商视频理解。 |

## 推荐的提供商
**图像**
- 如果你的活动模型支持图像，请优先使用。
- 良好的默认值：`openai/gpt-5.2`、`anthropic/claude-opus-4-5`、`google/gemini-3-pro-preview`。

**音频**
- `openai/gpt-4o-mini-transcribe`、`groq/whisper-large-v3-turbo` 或 `deepgram/nova-3`。
- CLI 后备：`whisper-cli`（whisper-cpp）或 `whisper`。
- Deepgram 设置：[Deepgram（音频转录）](/providers/deepgram)。

**视频**
- `google/gemini-3-flash-preview`（快速）、`google/gemini-3-pro-preview`（更丰富）。
- CLI 后备：`gemini` CLI（支持视频/音频上的 `read_file`）。

## 附件策略
每个功能的 `attachments` 控制处理哪些附件：
- `mode`：`first`（默认）或 `all`
- `maxAttachments`：限制处理的数量（默认 **1**）
- `prefer`：`first`、`last`、`path`、`url`

当 `mode: "all"` 时，输出标记为 `[Image 1/2]`、`[Audio 2/2]` 等。

## 配置示例

### 1) 共享模型列表 + 覆盖
```json5
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        { provider: "google", model: "gemini-3-flash-preview", capabilities: ["image", "audio", "video"] },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "阅读 {{MediaPath}} 处的媒体并在 <= {{MaxChars}} 字符内描述它。"
          ],
          capabilities: ["image", "video"]
        }
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 }
      },
      video: {
        maxChars: 500
      }
    }
  }
}
```

### 2) 仅音频 + 视频（图像关闭）
```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"]
          }
        ]
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "阅读 {{MediaPath}} 处的媒体并在 <= {{MaxChars}} 字符内描述它。"
            ]
          }
        ]
      }
    }
  }
}
```

### 3) 可选图像理解
```json5
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-5" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "阅读 {{MediaPath}} 处的媒体并在 <= {{MaxChars}} 字符内描述它。"
            ]
          }
        ]
      }
    }
  }
}
```

### 4) 多模态单一条目（显式功能）
```json5
{
  tools: {
    media: {
      image: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      audio: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] },
      video: { models: [{ provider: "google", model: "gemini-3-pro-preview", capabilities: ["image", "video", "audio"] }] }
    }
  }
}
```

## 状态输出
当媒体理解运行时，`/status` 包括一个简短的摘要行：

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

这显示了每个功能的结果以及适用的提供商/模型。

## 注意
- 理解是**尽力而为**。错误不会阻止回复。
- 即使理解被禁用，附件仍会传递给模型。
- 使用 `scope` 限制理解运行的位置（例如仅 DM）。

## 相关文档
- [配置](/gateway/configuration)
- [图像和媒体支持](/nodes/images)
