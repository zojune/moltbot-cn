---
summary: "`moltbot dns` CLI 参考(广域发现助手)"
read_when:
  - 您希望通过 Tailscale + CoreDNS 进行广域发现(DNS-SD)
  - 您正在为 moltbot.internal 设置拆分 DNS
---

# `moltbot dns`

广域发现(Tailscale + CoreDNS)的 DNS 助手。目前专注于 macOS + Homebrew CoreDNS。

相关:
- Gateway 发现: [Discovery](/gateway/discovery)
- 广域发现配置: [Configuration](/gateway/configuration)

## 设置

```bash
moltbot dns setup
moltbot dns setup --apply
```
