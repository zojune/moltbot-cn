---
summary: "Text-to-speech (TTS) for outbound replies"
read_when: 
  - Enabling text-to-speech for replies
  - Configuring TTS providers or limits
  - Using /tts commands
---

# Text-to-speech (TTS)

Moltbot can convert outbound replies into audio using ElevenLabs, OpenAI, or Edge TTS.
It works anywhere Moltbot can send audio; Telegram gets a round voice-注意 bubble.

## Supported 服务

- **ElevenLabs** (primary or fallback 提供商)
- **OpenAI** (primary or fallback 提供商; also used for summaries)
- **Edge TTS** (primary or fallback provider; uses `node-edge-tts`, 默认 when no API keys)

### Edge TTS notes

Edge TTS uses Microsoft Edge's online neural TTS service via the `node-edge-tts`
库. It's a hosted 服务 (not local), uses Microsoft’s 端点, and does
not require an API key. `node-edge-tts` exposes speech 配置 选项 and
输出 formats, but not all 选项 are supported by the Edge 服务. citeturn2search0

Because Edge TTS is a public Web 服务 without a published SLA or quota, treat it
as best-effort. If you need guaranteed limits and support, use OpenAI or ElevenLabs.
Microsoft's Speech REST API documents a 10‑minute audio limit per 请求; Edge TTS
does not publish limits, so assume similar or lower limits. citeturn0search3

## 可选 keys

If you want OpenAI or ElevenLabs:
- `ELEVENLABS_API_KEY` (or `XI_API_KEY`)
- `OPENAI_API_KEY`

Edge TTS does **not** require an API 键. If no API keys are found, Moltbot defaults
to Edge TTS (unless disabled via `messages.tts.edge.enabled=false`).

If multiple 提供商 are configured, the selected 提供商 is used first and the others are fallback 选项.
Auto-summary uses the configured `summaryModel` (or `agents.defaults.model.primary`),
so that 提供商 must also be authenticated if you enable summaries.

## 服务 links

- [OpenAI Text-to-Speech 指南](https://平台.openai.com/docs/指南/text-to-speech)
- [OpenAI Audio API 参考](https://平台.openai.com/docs/API-参考/audio)
- [ElevenLabs Text to Speech](https://elevenlabs.io/docs/API-参考/text-to-speech)
- [ElevenLabs 认证](https://elevenlabs.io/docs/API-参考/认证)
- [节点-edge-tts](https://github.com/SchneeHertz/节点-edge-tts)
- [Microsoft Speech 输出 formats](https://learn.microsoft.com/azure/ai-服务/speech-服务/REST-text-to-speech#audio-outputs)

## Is it 已启用 默认情况下?

No. Auto‑TTS is **off** 默认情况下. Enable it in 配置 with
`messages.tts.auto` or per session with `/tts always` (alias: `/tts on`).

Edge TTS **is** 已启用 默认情况下 once TTS is on, and is used automatically
when no OpenAI or ElevenLabs API keys are available.

## 配置

TTS config lives under `messages.tts` in `moltbot.json`.
Full 模式 is in [Gateway 配置](/Gateway/配置).

### Minimal 配置 (enable + 提供商)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "elevenlabs"
    }
  }
}
```

### OpenAI primary with ElevenLabs fallback

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "openai",
      summaryModel: "openai/gpt-4.1-mini",
      modelOverrides: {
        enabled: true
      },
      openai: {
        apiKey: "openai_api_key",
        model: "gpt-4o-mini-tts",
        voice: "alloy"
      },
      elevenlabs: {
        apiKey: "elevenlabs_api_key",
        baseUrl: "https://api.elevenlabs.io",
        voiceId: "voice_id",
        modelId: "eleven_multilingual_v2",
        seed: 42,
        applyTextNormalization: "auto",
        languageCode: "en",
        voiceSettings: {
          stability: 0.5,
          similarityBoost: 0.75,
          style: 0.0,
          useSpeakerBoost: true,
          speed: 1.0
        }
      }
    }
  }
}
```

### Edge TTS primary (no API 键)

```json5
{
  messages: {
    tts: {
      auto: "always",
      provider: "edge",
      edge: {
        enabled: true,
        voice: "en-US-MichelleNeural",
        lang: "en-US",
        outputFormat: "audio-24khz-48kbitrate-mono-mp3",
        rate: "+10%",
        pitch: "-5%"
      }
    }
  }
}
```

### Disable Edge TTS

```json5
{
  messages: {
    tts: {
      edge: {
        enabled: false
      }
    }
  }
}
```

### Custom limits + prefs 路径

```json5
{
  messages: {
    tts: {
      auto: "always",
      maxTextLength: 4000,
      timeoutMs: 30000,
      prefsPath: "~/.clawdbot/settings/tts.json"
    }
  }
}
```

### Only reply with audio after an inbound voice 注意

```json5
{
  messages: {
    tts: {
      auto: "inbound"
    }
  }
}
```

### Disable auto-摘要 for long replies

```json5
{
  messages: {
    tts: {
      auto: "always"
    }
  }
}
```

Then run:

```
/tts summary off
```

### Notes on fields

- `auto`: auto‑TTS mode (`off`, `always`, `inbound`, `tagged`).
  - `inbound` only sends audio after an inbound voice 注意.
  - `tagged` only sends audio when the reply includes `[[tts]]` tags.
- `enabled`: legacy toggle (doctor migrates this to `auto`).
- `mode`: `"final"` (default) or `"all"` (includes 工具/block replies).
- `provider`: `"elevenlabs"`, `"openai"`, or `"edge"` (fallback is automatic).
- If `provider` is **unset**, Moltbot prefers `openai` (if key), then `elevenlabs` (if 键),
  otherwise `edge`.
- `summaryModel`: optional cheap model for auto-summary; defaults to `agents.defaults.model.primary`.
  - Accepts `provider/model` or a configured 模型 alias.
- `modelOverrides`: allow the 模型 to emit TTS directives (on 默认情况下).
- `maxTextLength`: hard cap for TTS input (chars). `/tts audio` fails if exceeded.
- `timeoutMs`: 请求 timeout (ms).
- `prefsPath`: override the local prefs JSON 路径 (提供商/limit/摘要).
- `apiKey` values fall back to env vars (`ELEVENLABS_API_KEY`/`XI_API_KEY`, `OPENAI_API_KEY`).
- `elevenlabs.baseUrl`: override ElevenLabs API base URL.
- `elevenlabs.voiceSettings`:
  - `stability`, `similarityBoost`, `style`: `0..1`
  - `useSpeakerBoost`: `true|false`
  - `speed`: `0.5..2.0` (1.0 = normal)
- `elevenlabs.applyTextNormalization`: `auto|on|off`
- `elevenlabs.languageCode`: 2-letter ISO 639-1 (e.g. `en`, `de`)
- `elevenlabs.seed`: integer `0..4294967295` (best-effort determinism)
- `edge.enabled`: allow Edge TTS usage (default `true`; no API 键).
- `edge.voice`: Edge neural voice name (e.g. `en-US-MichelleNeural`).
- `edge.lang`: language code (e.g. `en-US`).
- `edge.outputFormat`: Edge output format (e.g. `audio-24khz-48kbitrate-mono-mp3`).
  - 参见 Microsoft Speech 输出 formats for valid values; not all formats are supported by Edge.
- `edge.rate` / `edge.pitch` / `edge.volume`: percent strings (e.g. `+10%`, `-5%`).
- `edge.saveSubtitles`: write JSON subtitles alongside the audio 文件.
- `edge.proxy`: proxy URL for Edge TTS 请求.
- `edge.timeoutMs`: 请求 timeout override (ms).

## 模型-driven overrides (默认 on)

默认情况下, the 模型 **can** emit TTS directives for a single reply.
When `messages.tts.auto` is `tagged`, these directives are 必需 to 触发器 audio.

When enabled, the model can emit `[[tts:...]]` directives to override the voice
for a single reply, plus an optional `[[tts:text]]...[[/tts:text]]` block to
provide expressive tags (laughter, singing cues, etc) that should only appear in
the audio.

示例 reply payload:

```
Here you go.

[[tts:provider=elevenlabs voiceId=pMsXgVXv3BLzUgSXRplE model=eleven_v3 speed=1.1]]
[[tts:text]](laughs) Read the song once more.[[/tts:text]]
```

Available directive keys (when 已启用):
- `provider` (`openai` | `elevenlabs` | `edge`)
- `voice` (OpenAI voice) or `voiceId` (ElevenLabs)
- `model` (OpenAI TTS 模型 or ElevenLabs 模型 id)
- `stability`, `similarityBoost`, `style`, `speed`, `useSpeakerBoost`
- `applyTextNormalization` (`auto|on|off`)
- `languageCode` (ISO 639-1)
- `seed`

Disable all 模型 overrides:

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: false
      }
    }
  }
}
```

可选 allowlist (disable specific overrides while keeping tags 已启用):

```json5
{
  messages: {
    tts: {
      modelOverrides: {
        enabled: true,
        allowProvider: false,
        allowSeed: false
      }
    }
  }
}
```

## Per-用户 preferences

Slash commands write local overrides to `prefsPath` (默认:
`~/.clawdbot/settings/tts.json`, override with `CLAWDBOT_TTS_PREFS` or
`messages.tts.prefsPath`).

Stored fields:
- `enabled`
- `provider`
- `maxLength` (摘要 threshold; 默认 1500 chars)
- `summarize` (default `true`)

These override `messages.tts.*` for that 主机.

## 输出 formats (fixed)

- **Telegram**: Opus voice note (`opus_48000_64` from ElevenLabs, `opus` from OpenAI).
  - 48kHz / 64kbps is a good voice-注意 tradeoff and 必需 for the round bubble.
- **Other channels**: MP3 (`mp3_44100_128` from ElevenLabs, `mp3` from OpenAI).
  - 44.1kHz / 128kbps is the 默认 balance for speech clarity.
- **Edge TTS**: uses `edge.outputFormat` (default `audio-24khz-48kbitrate-mono-mp3`).
  - `node-edge-tts` accepts an `outputFormat`, but not all formats are available
    from the Edge 服务. citeturn2search0
  - 输出 格式 values follow Microsoft Speech 输出 formats (including Ogg/WebM Opus). citeturn1search0
  - Telegram `sendVoice` accepts OGG/MP3/M4A; use OpenAI/ElevenLabs if you need
    guaranteed Opus voice notes. citeturn1search1
  - If the configured Edge 输出 格式 fails, Moltbot retries with MP3.

OpenAI/ElevenLabs formats are fixed; Telegram expects Opus for voice-注意 UX.

## Auto-TTS 行为

When 已启用, Moltbot:
- skips TTS if the reply already contains media or a `MEDIA:` directive.
- skips very short replies (< 10 chars).
- summarizes long replies when enabled using `agents.defaults.model.primary` (or `summaryModel`).
- attaches the generated audio to the reply.

If the reply exceeds `maxLength` and 摘要 is off (or no API 键 for the
摘要 模型), audio
is skipped and the normal text reply is sent.

## Flow diagram

```
Reply -> TTS enabled?
  no  -> send text
  yes -> has media / MEDIA: / short?
          yes -> send text
          no  -> length > limit?
                   no  -> TTS -> attach audio
                   yes -> summary enabled?
                            no  -> send text
                            yes -> summarize (summaryModel or agents.defaults.model.primary)
                                      -> TTS -> attach audio
```

## Slash 命令 用法

There is a single command: `/tts`.
参见 [Slash 命令](/工具/slash-命令) for enablement details.

Discord note: `/tts` is a built-in Discord 命令, so Moltbot registers
`/voice` as the native command there. Text `/tts ...` still works.

```
/tts off
/tts always
/tts inbound
/tts tagged
/tts status
/tts provider openai
/tts limit 2000
/tts summary off
/tts audio Hello from Moltbot
```

Notes:
- 命令 require an authorized sender (allowlist/owner 规则 still apply).
- `commands.text` or native 命令 registration must be 已启用.
- `off|always|inbound|tagged` are per‑session toggles (`/tts on` is an alias for `/tts always`).
- `limit` and `summary` are stored in local prefs, not the main 配置.
- `/tts audio` generates a one-off audio reply (does not toggle TTS on).

## 代理 工具

The `tts` tool converts text to speech and returns a `MEDIA:` 路径. When the
result is Telegram-compatible, the tool includes `[[audio_as_voice]]` so
Telegram sends a voice bubble.

## Gateway RPC

Gateway methods:
- `tts.status`
- `tts.enable`
- `tts.disable`
- `tts.convert`
- `tts.setProvider`
- `tts.providers`
