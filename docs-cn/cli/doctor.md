---
summary: "`moltbot doctor` CLI 参考(健康检查 + 引导式修复)"
read_when:
  - 您有连接/认证问题并希望得到引导式修复
  - 您已更新并希望进行健全性检查
---

# `moltbot doctor`

gateway 和频道的健康检查 + 快速修复。

相关:
- 故障排除: [Troubleshooting](/gateway/troubleshooting)
- 安全审计: [Security](/gateway/security)

## 示例

```bash
moltbot doctor
moltbot doctor --repair
moltbot doctor --deep
```

注意事项:
- 交互式提示(如钥匙串/OAuth 修复)仅在 stdin 是 TTY 且**未**设置 `--non-interactive` 时运行。无头运行(cron、Telegram、无终端)将跳过提示。
- `--fix`(`--repair` 的别名)将备份写入 `~/.clawdbot/moltbot.json.bak` 并删除未知的配置键,列出每个删除项。

## macOS:`launchctl` 环境覆盖

如果您之前运行了 `launchctl setenv CLAWDBOT_GATEWAY_TOKEN ...`(或 `...PASSWORD`),该值将覆盖您的配置文件,并可能导致持续的"未授权"错误。

```bash
launchctl getenv CLAWDBOT_GATEWAY_TOKEN
launchctl getenv CLAWDBOT_GATEWAY_PASSWORD

launchctl unsetenv CLAWDBOT_GATEWAY_TOKEN
launchctl unsetenv CLAWDBOT_GATEWAY_PASSWORD
```
