---
summary: "模型认证:OAuth、API 密钥和 setup-token"
read_when:
  - 调试模型认证或 OAuth 过期
  - 文档化认证或凭据存储
---
# 认证

Moltbot 支持模型提供商的 OAuth 和 API 密钥。对于 Anthropic
账户,我们建议使用 **API 密钥**。对于 Claude 订阅访问,
使用由 `claude setup-token` 创建的长期令牌。

参见 [/concepts/oauth](/concepts/oauth) 了解完整的 OAuth 流程和存储
布局。

## 推荐的 Anthropic 设置(API 密钥)

如果您直接使用 Anthropic,请使用 API 密钥。

1) 在 Anthropic Console 中创建 API 密钥。
2) 将其放在 **gateway host** 上(运行 `moltbot gateway` 的机器)。

```bash
export ANTHROPIC_API_KEY="..."
moltbot models status
```

3) 如果 Gateway 在 systemd/launchd 下运行,最好将密钥放在
`~/.clawdbot/.env` 中,以便守护进程可以读取它:

```bash
cat >> ~/.clawdbot/.env <<'EOF'
ANTHROPIC_API_KEY=...
EOF
```

然后重启守护进程(或重启您的 Gateway 进程)并重新检查:

```bash
moltbot models status
moltbot doctor
```

如果您不想自己管理环境变量,入职向导可以存储
API 密钥供守护进程使用: `moltbot onboard`。

参见 [Help](/help) 了解环境继承的详细信息(`env.shellEnv`,
`~/.clawdbot/.env`、systemd/launchd)。

## Anthropic: setup-token(订阅认证)

对于 Anthropic,推荐路径是 **API 密钥**。如果您使用 Claude
订阅,也支持 setup-token 流程。在 **gateway host** 上运行:

```bash
claude setup-token
```

然后将其粘贴到 Moltbot 中:

```bash
moltbot models auth setup-token --provider anthropic
```

如果令牌是在另一台机器上创建的,请手动粘贴:

```bash
moltbot models auth paste-token --provider anthropic
```

如果您看到类似以下的 Anthropic 错误:

```
This credential is only authorized for use with Claude Code and cannot be used for other API requests.
```

...请改用 Anthropic API 密钥。

手动令牌输入(任何提供商;写入 `auth-profiles.json` + 更新配置):

```bash
moltbot models auth paste-token --provider anthropic
moltbot models auth paste-token --provider openrouter
```

自动化友好的检查(过期/缺失时退出 `1`,即将过期时退出 `2`):

```bash
moltbot models status --check
```

可选的运维脚本(systemd/Termux)记录在此:
[/automation/auth-monitoring](/automation/auth-monitoring)

> `claude setup-token` 需要交互式 TTY。

## 检查模型认证状态

```bash
moltbot models status
moltbot doctor
```

## 控制使用哪个凭据

### 每次会话(聊天命令)

使用 `/model <alias-or-id>@<profileId>` 为当前会话固定特定的提供商凭据(示例配置文件 id: `anthropic:default`、`anthropic:work`)。

使用 `/model`(或 `/model list`)获取紧凑的选择器;使用 `/model status` 获取完整视图(候选者 + 下一个认证配置文件,以及配置时的提供商端点详细信息)。

### 每个代理(CLI 覆盖)

为代理设置显式的认证配置文件顺序覆盖(存储在该代理的 `auth-profiles.json` 中):

```bash
moltbot models auth order get --provider anthropic
moltbot models auth order set --provider anthropic anthropic:default
moltbot models auth order clear --provider anthropic
```

使用 `--agent <id>` 定位特定代理;省略它以使用配置的默认代理。

## 故障排除

### "未找到凭据"

如果 Anthropic 令牌配置文件缺失,请在 **gateway host** 上运行 `claude setup-token`,
然后重新检查:

```bash
moltbot models status
```

### 令牌即将过期/已过期

运行 `moltbot models status` 以确认哪个配置文件即将过期。如果配置文件
缺失,请重新运行 `claude setup-token` 并再次粘贴令牌。

## 要求

- Claude Max 或 Pro 订阅(用于 `claude setup-token`)
- 已安装 Claude Code CLI(可使用 `claude` 命令)
