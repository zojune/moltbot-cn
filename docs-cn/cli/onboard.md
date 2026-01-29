---
summary: "`moltbot onboard` CLI 参考(交互式入门向导)"
read_when:
  - 您需要引导式设置 gateway、工作区、认证、频道和技能
---

# `moltbot onboard`

交互式入门向导(本地或远程 Gateway 设置)。

相关:
- 向导指南: [Onboarding](/start/onboarding)

## 示例

```bash
moltbot onboard
moltbot onboard --flow quickstart
moltbot onboard --flow manual
moltbot onboard --mode remote --remote-url ws://gateway-host:18789
```

流程注意事项:
- `quickstart`:最少的提示,自动生成 gateway 令牌。
- `manual`:端口/绑定/认证的完整提示(advanced 的别名)。
- 最快的第一条聊天:`moltbot dashboard`(Control UI,无需频道设置)。
