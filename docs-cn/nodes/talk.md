---
summary: "对话模式：使用 ElevenLabs TTS 的连续语音对话"
read_when:
  - 在 macOS/iOS/Android 上实现对话模式
  - 更改语音/TTS/中断行为
---
# 对话模式

对话模式是一个连续的语音对话循环：
1) 监听语音
2) 将转录发送到模型（主会话，chat.send）
3) 等待响应
4) 通过 ElevenLabs 讲出它（流式播放）

## 行为（macOS）
- 启用对话模式时的**始终在线覆盖**。
- **监听 → 思考 → 讲话**阶段转换。
- 在**短暂停顿**（静默窗口）时，发送当前转录。
- 回复被**写入 WebChat**（与键入相同）。
- **讲话时中断**（默认开启）：如果用户在助手讲话时开始说话，我们停止播放并记录中断时间戳以用于下一个提示。

## 回复中的语音指令
助手可能会在回复前加上**单行 JSON** 来控制语音：

```json
{"voice":"<voice-id>","once":true}
```

规则：
- 仅第一个非空行。
- 忽略未知键。
- `once: true` 仅适用于当前回复。
- 没有 `once`，语音成为对话模式的新默认值。
- JSON 行在 TTS 播放之前被剥离。

支持的键：
- `voice` / `voice_id` / `voiceId`
- `model` / `model_id` / `modelId`
- `speed`、`rate`（WPM）、`stability`、`similarity`、`style`、`speakerBoost`
- `seed`、`normalize`、`lang`、`output_format`、`latency_tier`
- `once`

## 配置（`~/.clawdbot/moltbot.json`）
```json5
{
  "talk": {
    "voiceId": "elevenlabs_voice_id",
    "modelId": "eleven_v3",
    "outputFormat": "mp3_44100_128",
    "apiKey": "elevenlabs_api_key",
    "interruptOnSpeech": true
  }
}
```

默认值：
- `interruptOnSpeech`：true
- `voiceId`：回退到 `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID`（或第一个 ElevenLabs 语音，当 API 密钥可用时）
- `modelId`：未设置时默认为 `eleven_v3`
- `apiKey`：回退到 `ELEVENLABS_API_KEY`（或可用的网关 shell 配置文件）
- `outputFormat`：在 macOS/iOS 上默认为 `pcm_44100`，在 Android 上默认为 `pcm_24000`（设置 `mp3_*` 以强制 MP3 流式传输）

## macOS UI
- 菜单栏切换：**对话**
- 配置选项卡：**对话模式**组（语音 ID + 中断切换）
- 覆盖：
  - **监听**：云随麦克风级别脉冲
  - **思考**：下沉动画
  - **讲话**：辐射环
  - 点击云：停止讲话
  - 点击 X：退出对话模式

## 注意
- 需要语音 + 麦克风权限。
- 使用针对会话键 `main` 的 `chat.send`。
- TTS 使用 ElevenLabs 流式 API 配合 `ELEVENLABS_API_KEY` 和 macOS/iOS/Android 上的增量播放以获得更低延迟。
- `eleven_v3` 的 `stability` 验证为 `0.0`、`0.5` 或 `1.0`；其他模型接受 `0..1`。
- 设置时 `latency_tier` 验证为 `0..4`。
- Android 支持 `pcm_16000`、`pcm_22050`、`pcm_24000` 和 `pcm_44100` 输出格式，用于低延迟 AudioTrack 流式传输。
