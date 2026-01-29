---
summary: "用于传入语音笔记的 Deepgram 转录"
read_when:
  - 你想为音频附件使用 Deepgram 语音转文字
  - 你需要一个快速的 Deepgram 配置示例
---
# Deepgram (音频转录)

Deepgram 是一个语音转文字 API。在 Moltbot 中，它用于通过 `tools.media.audio` 进行**传入音频/语音笔记转录**。

启用后，Moltbot 将音频文件上传到 Deepgram 并将转录注入到回复管道中（`{{Transcript}}` + `[Audio]` 块）。这**不是流式的**；它使用预录制的转录端点。

网站：https://deepgram.com
文档：https://developers.deepgram.com

## 快速开始

1) 设置你的 API 密钥：
```
DEEPGRAM_API_KEY=dg_...
```

2) 启用提供程序：
```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```

## 选项

- `model`：Deepgram 模型 ID（默认：`nova-3`）
- `language`：语言提示（可选）
- `tools.media.audio.providerOptions.deepgram.detect_language`：启用语言检测（可选）
- `tools.media.audio.providerOptions.deepgram.punctuate`：启用标点符号（可选）
- `tools.media.audio.providerOptions.deepgram.smart_format`：启用智能格式化（可选）

带语言的示例：
```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "deepgram", model: "nova-3", language: "en" }
        ]
      }
    }
  }
}
```

带 Deepgram 选项的示例：
```json5
{
  tools: {
    media: {
      audio: {
        enabled: true,
        providerOptions: {
          deepgram: {
            detect_language: true,
            punctuate: true,
            smart_format: true
          }
        },
        models: [{ provider: "deepgram", model: "nova-3" }]
      }
    }
  }
}
```

## 注意事项

- 身份验证遵循标准提供程序身份验证顺序；`DEEPGRAM_API_KEY` 是最简单的路径。
- 使用代理时，使用 `tools.media.audio.baseUrl` 和 `tools.media.audio.headers` 覆盖端点或标头。
- 输出遵循与其他提供程序相同的音频规则（大小限制、超时、转录注入）。
