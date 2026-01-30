# 安全政策

如果你认为你在 Moltbot 中发现了安全问题，请私下报告。

## 报告

- 电子邮件：`steipete@gmail.com`
- 包括内容：重现步骤、影响评估以及（如果可能）最小的概念验证。

## 操作指南

有关威胁模型 + 加固指南（包括 `moltbot security audit --deep` 和 `--fix`），请参阅：

- `https://docs.molt.bot/gateway/security`

### Web 界面安全

Moltbot 的 web 界面仅供本地使用。**不要**将其绑定到公共互联网；它没有针对公共暴露进行加固。

## 运行时要求

### Node.js 版本

Moltbot 需要 **Node.js 22.12.0 或更高版本**（LTS）。此版本包括重要的安全补丁：

- CVE-2025-59466：async_hooks DoS 漏洞
- CVE-2026-21636：权限模型绕过漏洞

验证你的 Node.js 版本：

```bash
node --version  # 应该是 v22.12.0 或更高
```

### Docker 安全

在 Docker 中运行 Moltbot 时：

1. 官方镜像作为非 root 用户（`node`）运行，以减少攻击面
2. 尽可能使用 `--read-only` 标志进行额外的文件系统保护
3. 使用 `--cap-drop=ALL` 限制容器功能

安全 Docker 运行示例：

```bash
docker run --read-only --cap-drop=ALL \
  -v moltbot-data:/app/data \
  moltbot/moltbot:latest
```

## 安全扫描

此项目在 CI/CD 中使用 `detect-secrets` 进行自动秘密检测。
有关配置，请参阅 `.detect-secrets.cfg`；有关基线，请参阅 `.secrets.baseline`。

本地运行：

```bash
pip install detect-secrets==1.5.0
detect-secrets scan --baseline .secrets.baseline
```
