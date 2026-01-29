---
summary: "`moltbot voicecall` CLI 参考(语音通话插件命令界面)"
read_when:
  - 您使用语音通话插件并需要 CLI 入口点
  - 您想要 `voicecall call|continue|status|tail|expose` 的快速示例
---

# `moltbot voicecall`

`voicecall` 是插件提供的命令。它仅在安装并启用了语音通话插件时出现。

主要文档:
- 语音通话插件: [Voice Call](/plugins/voice-call)

## 常用命令

```bash
moltbot voicecall status --call-id <id>
moltbot voicecall call --to "+15555550123" --message "Hello" --mode notify
moltbot voicecall continue --call-id <id> --message "Any questions?"
moltbot voicecall end --call-id <id>
```

## 暴露 webhooks(Tailscale)

```bash
moltbot voicecall expose --mode serve
moltbot voicecall expose --mode funnel
moltbot voicecall unexpose
```

安全注意事项:仅将 webhook 端点暴露到您信任的网络。尽可能优先使用 Tailscale Serve 而不是 Funnel。
