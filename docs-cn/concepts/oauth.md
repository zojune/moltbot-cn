---
summary: "Moltbot 中的 OAuth：令牌交换、存储和多账户模式"
read_when:
  - 你想要端到端了解 Moltbot OAuth
  - 你遇到令牌失效/注销问题
  - 你想要 setup-token 或 OAuth 身份流程
  - 你想要多个账户或配置文件路由
---
# OAuth

Moltbot 通过 OAuth 支持提供者的"订阅身份"（特别是 **OpenAI Codex (ChatGPT OAuth)**）。对于 Anthropic 订阅，请使用 **setup-token** 流程。此页面解释了：
- OAuth **令牌交换**如何工作（PKCE）
- 令牌**存储**位置（以及原因）
- 如何处理**多个账户**（配置文件 + 每会话覆盖）

Moltbot 还支持**提供者插件**，它们提供自己的 OAuth 或 API 密钥流程。通过以下方式运行它们：

```bash
moltbot models auth login --provider <id>
```

## 令牌汇（为什么存在）

OAuth 提供者通常在登录/刷新期间铸造**新的刷新令牌**。某些提供者（或 OAuth 客户端）可以在为同一用户/应用颁发新令牌时使较旧的刷新令牌失效。

实际症状：
- 你通过 Moltbot *以及*通过 Claude Code / Codex CLI 登录 → 其中一个稍后随机"注销"

为了减少这种情况，Moltbot 将 `auth-profiles.json` 视为**令牌汇**：
- 运行时从**一个地方**读取凭据
- 我们可以保持多个配置文件并确定性路由它们

## 存储（令牌所在位置）

机密按**代理**存储：

- 身份配置文件（OAuth + API 密钥）：`~/.clawdbot/agents/<agentId>/agent/auth-profiles.json`
- 运行时缓存（自动管理；不要编辑）：`~/.clawdbot/agents/<agentId>/agent/auth.json`

遗留仅导入文件（仍然支持，但不是主存储）：
- `~/.clawdbot/credentials/oauth.json`（首次使用时导入 `auth-profiles.json`）

所有上述内容也尊重 `$CLAWDBOT_STATE_DIR`（状态目录覆盖）。完整参考：[/gateway/configuration](/gateway/configuration#auth-storage-oauth--api-keys)

## Anthropic setup-token（订阅身份）

在任何机器上运行 `claude setup-token`，然后将其粘贴到 Moltbot 中：

```bash
moltbot models auth setup-token --provider anthropic
```

如果你在其他地方生成了令牌，请手动粘贴：

```bash
moltbot models auth paste-token --provider anthropic
```

验证：

```bash
moltbot models status
```

## OAuth 交换（登录如何工作）

Moltbot 的交互式登录流程在 `@mariozechner/pi-ai` 中实现并连接到向导/命令。

### Anthropic (Claude Pro/Max) setup-token

流程形状：
1) 运行 `claude setup-token`
2) 将令牌粘贴到 Moltbot 中
3) 存储为令牌身份配置文件（无刷新）

向导路径是 `moltbot onboard` → 身份选择 `setup-token`（Anthropic）。

### OpenAI Codex (ChatGPT OAuth)

流程形状（PKCE）：
1) 生成 PKCE 验证器/质询 + 随机 `state`
2) 打开 `https://auth.openai.com/oauth/authorize?...`
3) 尝试在 `http://127.0.0.1:1455/auth/callback` 上捕获回调
4) 如果回调无法绑定（或者你是远程/无头），请粘贴重定向 URL/代码
5) 在 `https://auth.openai.com/oauth/token` 处交换
6) 从访问令牌中提取 `accountId` 并存储 `{ access, refresh, expires, accountId }`

向导路径是 `moltbot onboard` → 身份选择 `openai-codex`。

## 刷新 + 到期

配置文件存储 `expires` 时间戳。

在运行时：
- 如果 `expires` 在将来 → 使用存储的访问令牌
- 如果已过期 → 刷新（在文件锁下）并覆盖存储的凭据

刷新流程是自动的；你通常不需要手动管理令牌。

## 多个账户（配置文件）+ 路由

两种模式：

### 1) 推荐：单独的代理

如果你想要"个人"和"工作"从不交互，请使用隔离代理（单独的会话 + 凭据 + 工作区）：

```bash
moltbot agents add work
moltbot agents add personal
```

然后按代理配置身份（向导）并将聊天路由到正确的代理。

### 2) 高级：一个代理中的多个配置文件

`auth-profiles.json` 支持同一提供者的多个配置文件 ID。

选择使用的配置文件：
- 通过配置排序全局（`auth.order`）
- 通过每会话 `/model ...@<profileId>`

示例（会话覆盖）：
- `/model Opus@anthropic:work`

如何查看存在哪些配置文件 ID：
- `moltbot channels list --json`（显示 `auth[]`）

相关文档：
- [/concepts/model-failover](/concepts/model-failover)（轮换 + 冷却规则）
- [/tools/slash-commands](/tools/slash-commands)（命令表面）
