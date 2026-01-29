---
summary: "监控模型提供商的 OAuth 过期时间"
read_when:
  - 设置认证过期监控或告警
  - 自动化 Claude Code / Codex OAuth 刷新检查
---
# 认证监控

Moltbot 通过 `moltbot models status` 暴露 OAuth 过期健康状态。将其用于自动化和告警；脚本是为手机工作流程提供的可选额外功能。

## 推荐：CLI 检查（可移植）

```bash
moltbot models status --check
```

退出代码：
- `0`：正常
- `1`：凭据已过期或缺失
- `2`：即将过期（24 小时内）

这适用于 cron/systemd，无需额外脚本。

## 可选脚本（运维 / 手机工作流程）

这些脚本位于 `scripts/` 目录下，属于**可选**功能。它们假设可以通过 SSH 访问网关主机，并为 systemd + Termux 进行了优化。

- `scripts/claude-auth-status.sh` 现在使用 `moltbot models status --json` 作为
  真实数据源（如果 CLI 不可用，则回退到直接读取文件），
  因此请将 `moltbot` 保留在 `PATH` 中以供定时器使用。
- `scripts/auth-monitor.sh`：cron/systemd 定时器目标；发送告警（ntfy 或手机）。
- `scripts/systemd/moltbot-auth-monitor.{service,timer}`：systemd 用户定时器。
- `scripts/claude-auth-status.sh`：Claude Code + Moltbot 认证检查器（完整/json/简单）。
- `scripts/mobile-reauth.sh`：通过 SSH 进行引导式重新认证流程。
- `scripts/termux-quick-auth.sh`：一键小部件状态 + 打开认证 URL。
- `scripts/termux-auth-widget.sh`：完整的引导式小部件流程。
- `scripts/termux-sync-widget.sh`：同步 Claude Code 凭据 → Moltbot。

如果您不需要手机自动化或 systemd 定时器，可以跳过这些脚本。
