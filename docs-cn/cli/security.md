---
summary: "`moltbot security` CLI 参考(审计并修复常见安全问题)"
read_when:
  - 您想对配置/状态运行快速安全审计
  - 您想应用安全的"修复"建议(chmod、加强默认值)
---

# `moltbot security`

安全工具(审计 + 可选修复)。

相关:
- 安全指南: [Security](/gateway/security)

## 审计

```bash
moltbot security audit
moltbot security audit --deep
moltbot security audit --fix
```

当多个 DM 发件人共享主会话时,审计会发出警告,并建议 `session.dmScope="per-channel-peer"`(对于多账户频道的共享收件箱,建议使用 `per-account-channel-peer`)。
它还警告当使用小型模型(`<=300B`)时没有沙箱并且启用了 web/浏览器工具。
