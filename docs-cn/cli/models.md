---
summary: "`moltbot models` CLI 参考(状态/列表/设置/扫描、别名、回退、认证)"
read_when:
  - 您想更改默认模型或查看提供商认证状态
  - 您想扫描可用的模型/提供商并调试认证配置文件
---

# `moltbot models`

模型发现、扫描和配置(默认模型、回退、认证配置文件)。

相关:
- 提供商 + 模型: [Models](/providers/models)
- 提供商认证设置: [Getting started](/start/getting-started)

## 常用命令

```bash
moltbot models status
moltbot models list
moltbot models set <model-or-alias>
moltbot models scan
```

`moltbot models status` 显示解析的默认值/回退以及认证概述。
当提供商使用快照可用时,OAuth/令牌状态部分包括提供商使用头。
添加 `--probe` 对每个配置的提供商配置文件运行实时认证探测。
探测是真正的请求(可能会消耗令牌并触发速率限制)。

注意事项:
- `models set <model-or-alias>` 接受 `provider/model` 或别名。
- 模型引用通过在**第一个** `/` 处拆分来解析。如果模型 ID 包含 `/`(OpenRouter 风格),请包含提供商前缀(例如:`openrouter/moonshotai/kimi-k2`)。
- 如果省略提供商,Moltbot 将输入视为别名或**默认提供商**的模型(仅在模型 ID 中没有 `/` 时有效)。

### `models status`
选项:
- `--json`
- `--plain`
- `--check`(退出 1=过期/缺失,2=即将过期)
- `--probe`(配置的认证配置文件的实时探测)
- `--probe-provider <name>`(探测一个提供商)
- `--probe-profile <id>`(可重复或逗号分隔的配置文件 id)
- `--probe-timeout <ms>`
- `--probe-concurrency <n>`
- `--probe-max-tokens <n>`

## 别名 + 回退

```bash
moltbot models aliases list
moltbot models fallbacks list
```

## 认证配置文件

```bash
moltbot models auth add
moltbot models auth login --provider <id>
moltbot models auth setup-token
moltbot models auth paste-token
```
`models auth login` 运行提供商插件的认证流程(OAuth/API 密钥)。使用
`moltbot plugins list` 查看安装了哪些提供商。

注意事项:
- `setup-token` 提示输入 setup-token 值(在任何机器上使用 `claude setup-token` 生成)。
- `paste-token` 接受在其他地方或从自动化生成的令牌字符串。
