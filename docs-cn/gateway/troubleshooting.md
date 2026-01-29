---
summary: "常见 Moltbot 故障的快速故障排除指南"
read_when:
  - 调查运行时问题或故障
---
# 故障排除🔧

当 Moltbot 行为异常时，以下是修复方法。

如果您只是想要一个快速的分诊配方，请从 FAQ 的[前 60 秒](/help/faq#first-60-seconds-if-somethings-broken)开始。此页面深入探讨运行时故障和诊断。

提供商特定的快捷方式：[/channels/troubleshooting](/channels/troubleshooting)

## 状态和诊断

快速分诊命令（按顺序）：

| 命令 | 它告诉您什么 | 何时使用它 |
|---|---|---|
| `moltbot status` | 本地摘要：OS + 更新、gateway 可达性/模式、服务、agents/会话、提供商配置状态 | 首次检查、快速概述 |
| `moltbot status --all` | 完整的本地诊断（只读、可粘贴、相当安全），包括日志尾部 | 当您需要共享调试报告时 |
| `moltbot status --deep` | 运行 gateway 健康检查（包括提供商探测；需要可访问的 gateway） | 当"配置"并不意味着"工作"时 |
| `moltbot gateway probe` | Gateway 发现 + 可达性（本地 + 远程目标） | 当您怀疑探测了错误的 gateway 时 |
| `moltbot channels status --probe` | 询问运行的 gateway 通道状态（并可选择探测） | 当 gateway 可访问但通道行为异常时 |
| `moltbot gateway status` | 监督器状态（launchd/systemd/schtasks）、运行时 PID/退出、最后的 gateway 错误 | 当服务"看起来已加载"但没有任何运行时 |
| `moltbot logs --follow` | 实时日志（运行时问题的最佳信号） | 当您需要实际的失败原因时 |

**共享输出：**更喜欢 `moltbot status --all`（它会编辑令牌）。如果您粘贴 `moltbot status`，请考虑先设置 `CLAWDBOT_SHOW_SECRETS=0`（令牌预览）。

另请参阅：[健康检查](/gateway/health)和[日志记录](/logging)。

## 常见问题

### 未找到提供商 "anthropic" 的 API 密钥

这意味着 **agent 的认证存储为空**或缺少 Anthropic 凭据。
认证是**每个 agent 的**，因此新 agent 不会继承主 agent 的密钥。

修复选项：
- 重新运行入门并为该 agent 选择 **Anthropic**。
- 或在 **gateway 主机**上粘贴 setup-token：
  ```bash
  moltbot models auth setup-token --provider anthropic
  ```
- 或将 `auth-profiles.json` 从主 agent 目录复制到新 agent 目录。

验证：
```bash
moltbot models status
```

### OAuth 令牌刷新失败（Anthropic Claude 订阅）

这意味着存储的 Anthropic OAuth 令牌已过期，刷新失败。
如果您使用的是 Claude 订阅（无 API 密钥），最可靠的修复方法是
切换到 **Claude Code setup-token** 并在 **gateway 主机**上粘贴它。

**推荐（setup-token）：**

```bash
# 在 gateway 主机上运行（粘贴 setup-token）
moltbot models auth setup-token --provider anthropic
moltbot models status
```

如果您在其他地方生成了令牌：

```bash
moltbot models auth paste-token --provider anthropic
moltbot models status
```

更多详情：[Anthropic](/providers/anthropic)和[OAuth](/concepts/oauth)。

### Control UI 在 HTTP 上失败（"需要设备身份"/"连接失败"）

如果您通过纯 HTTP（例如 `http://<lan-ip>:18789/` 或
`http://<tailscale-ip>:18789/`）打开仪表板，浏览器将在**非安全上下文**中运行
并阻止 WebCrypto，因此无法生成设备身份。

**修复：**
- 最好通过[Tailscale Serve](/gateway/tailscale)使用 HTTPS。
- 或在 gateway 主机上本地打开：`http://127.0.0.1:18789/`。
- 如果您必须保持在 HTTP 上，请启用 `gateway.controlUi.allowInsecureAuth: true` 并
  使用 gateway 令牌（仅令牌；无设备身份/配对）。请参阅
  [Control UI](/web/control-ui#insecure-http)。

### CI Secrets Scan 失败

这意味着 `detect-secrets` 发现了尚未在基线中的新候选项。
请遵循[秘密扫描](/gateway/security#secret-scanning-detect-secrets)。

### 服务已安装但没有任何运行

如果 gateway 服务已安装但进程立即退出，服务
可能看起来"已加载"，但没有任何运行。

**检查：**
```bash
moltbot gateway status
moltbot doctor
```

Doctor/服务将显示运行时状态（PID/最后退出）和日志提示。

**日志：**
- 首选：`moltbot logs --follow`
- 文件日志（始终）：`/tmp/moltbot/moltbot-YYYY-MM-DD.log`（或您配置的 `logging.file`）
- macOS LaunchAgent（如果已安装）：`$CLAWDBOT_STATE_DIR/logs/gateway.log` 和 `gateway.err.log`
- Linux systemd（如果已安装）：`journalctl --user -u moltbot-gateway[-<profile>].service -n 200 --no-pager`
- Windows：`schtasks /Query /TN "Moltbot Gateway (<profile>)" /V /FO LIST`

**启用更多日志记录：**
- 提升文件日志详细信息（持久化 JSONL）：
  ```json
  { "logging": { "level": "debug" } }
  ```
- 提升控制台详细程度（仅 TTY 输出）：
  ```json
  { "logging": { "consoleLevel": "debug", "consoleStyle": "pretty" } }
  ```
- 快速提示：`--verbose` 仅影响**控制台**输出。文件日志仍由 `logging.level` 控制。

请参阅[/logging](/logging)以获取格式、配置和访问的完整概述。

### "Gateway 启动被阻止：设置 gateway.mode=local"

这意味着配置存在但 `gateway.mode` 未设置（或不是 `local`），因此
Gateway 拒绝启动。

**修复（推荐）：**
- 运行向导并将 Gateway 运行模式设置为 **Local**：
  ```bash
  moltbot configure
  ```
- 或直接设置：
  ```bash
  moltbot config set gateway.mode local
  ```

**如果您打算运行远程 Gateway：**
- 设置远程 URL 并保持 `gateway.mode=remote`：
  ```bash
  moltbot config set gateway.mode remote
  moltbot config set gateway.remote.url "wss://gateway.example.com"
  ```

**临时/仅开发：**传递 `--allow-unconfigured` 以在没有
`gateway.mode=local` 的情况下启动 gateway。

**还没有配置文件？**运行 `moltbot setup` 创建初始配置，然后重新运行
gateway。

### 服务环境（PATH + 运行时）

gateway 服务以**最小 PATH** 运行，以避免 shell/管理器杂乱：
- macOS：`/opt/homebrew/bin`、`/usr/local/bin`、`/usr/bin`、`/bin`
- Linux：`/usr/local/bin`、`/usr/bin`、`/bin`

这有意排除了版本管理器（nvm/fnm/volta/asdf）和包
管理器（pnpm/npm），因为服务不加载您的 shell init。运行时
变量如 `DISPLAY` 应该存在于 `~/.clawdbot/.env` 中（由 gateway 提前加载）。
在 `host=gateway` 上运行 exec 会将您的登录 shell `PATH` 合并到 exec 环境中，
因此缺少的工具通常意味着您的 shell init 没有导出它们（或设置
`tools.exec.pathPrepend`）。请参阅[/tools/exec](/tools/exec)。

WhatsApp + Telegram 通道需要**Node**；不支持 Bun。如果您
的服务是使用 Bun 或版本管理的 Node 路径安装的，请运行 `moltbot doctor`
迁移到系统 Node 安装。

### 沙箱中的技能缺少 API 密钥

**症状：**技能在主机上工作，但在沙箱中由于缺少 API 密钥而失败。

**原因：**沙箱化 exec 在 Docker 内运行，并且**不**继承主机 `process.env`。

**修复：**
- 设置 `agents.defaults.sandbox.docker.env`（或每个 agent `agents.list[].sandbox.docker.env`）
- 或将密钥烘焙到您的自定义沙箱镜像中
- 然后运行 `moltbot sandbox recreate --agent <id>`（或 `--all`）

### 服务正在运行但端口未监听

如果服务报告**正在运行**但 gateway 端口上没有任何监听，
Gateway 可能拒绝绑定。

**"正在运行"在这里意味着**
- `Runtime: running` 意味着您的监督器（launchd/systemd/schtasks）认为进程是活着的。
- `RPC probe` 意味着 CLI 实际上可以连接到 gateway WebSocket 并调用 `status`。
- 始终信任 `Probe target:` + `Config (service):` 作为"我们实际尝试了什么？"行。

**检查：**
- `gateway.mode` 必须是 `local` 才能运行 `moltbot gateway` 和服务。
- 如果您设置 `gateway.mode=remote`，**CLI 默认值**为远程 URL。服务可能仍在本地运行，但您的 CLI 可能探测到错误的地方。使用 `moltbot gateway status` 查看服务的解析端口 + 探测目标（或传递 `--url`）。
- `moltbot gateway status` 和 `moltbot doctor` 在服务看起来正在运行但端口关闭时显示**最后的 gateway 错误**。
- 非 loopback 绑定（`lan`/`tailnet`/`custom`，或 loopback 不可用时的 `auto`）需要认证：
  `gateway.auth.token`（或 `CLAWDBOT_GATEWAY_TOKEN`）。
- `gateway.remote.token` 仅用于远程 CLI 调用；它**不**启用本地认证。
- `gateway.token` 被忽略；使用 `gateway.auth.token`。

**如果 `moltbot gateway status` 显示配置不匹配**
- `Config (cli): ...` 和 `Config (service): ...` 通常应该匹配。
- 如果不匹配，您几乎肯定正在编辑一个配置，而服务正在运行另一个配置。
- 修复：从您希望服务使用的相同 `--profile` / `CLAWDBOT_STATE_DIR` 重新运行 `moltbot gateway install --force`。

**如果 `moltbot gateway status` 报告服务配置问题**
- 监督器配置（launchd/systemd/schtasks）缺少当前默认值。
- 修复：运行 `moltbot doctor` 来更新它（或 `moltbot gateway install --force` 进行完全重写）。

**如果 `Last gateway error:` 提到"拒绝绑定...没有认证"**
- 您将 `gateway.bind` 设置为非 loopback 模式（`lan`/`tailnet`/`custom`，或 loopback 不可用时的 `auto`），但没有配置认证。
- 修复：设置 `gateway.auth.mode` + `gateway.auth.token`（或导出 `CLAWDBOT_GATEWAY_TOKEN`）并重新启动服务。

**如果 `moltbot gateway status` 说 `bind=tailnet` 但没有找到 tailnet 接口**
- gateway 尝试绑定到 Tailscale IP（100.64.0.0/10），但在主机上未检测到任何 IP。
- 修复：在该机器上启动 Tailscale（或将 `gateway.bind` 更改为 `loopback`/`lan`）。

**如果 `Probe note:` 说探测使用 loopback**
- 对于 `bind=lan`，这是预期的：gateway 监听 `0.0.0.0`（所有接口），loopback 应该仍然可以在本地连接。
- 对于远程客户端，使用真实的 LAN IP（不是 `0.0.0.0`）加上端口，并确保配置了认证。

### 地址已在使用中（端口 18789）

这意味着某些东西已经在 gateway 端口上监听。

**检查：**
```bash
moltbot gateway status
```

它将显示监听器和可能的原因（gateway 已经在运行，SSH 隧道）。
如果需要，停止服务或选择不同的端口。

### 检测到额外的工作区文件夹

如果您从旧安装升级，您可能仍然在磁盘上有 `~/moltbot`。
多个工作区目录可能导致令人困惑的认证或状态漂移，因为
只有一个工作区是活动的。

**修复：**保持一个活动工作区并归档/删除其余的工作区。请参阅
[Agent 工作区](/concepts/agent-workspace#extra-workspace-folders)。

### 主聊天在沙箱工作区中运行

症状：`pwd` 或文件工具显示 `~/.clawdbot/sandboxes/...`，尽管您
期望主机工作区。

**原因：** `agents.defaults.sandbox.mode: "non-main"` 键定 `session.mainKey`（默认 `"main"`）。
群组/通道会话使用自己的键，因此它们被视为非 main 并
获得沙箱工作区。

**修复选项：**
- 如果您想要 agent 的主机工作区：设置 `agents.list[].sandbox.mode: "off"`。
- 如果您想要沙箱内的主机工作区访问：为该 agent 设置 `workspaceAccess: "rw"`。

### "Agent 中止"

agent 在响应中途被中断。

**原因：**
- 用户发送了 `stop`、`abort`、`esc`、`wait` 或 `exit`
- 超时超出
- 进程崩溃

**修复：**只需发送另一条消息。会话继续。

### "Agent 在回复前失败：未知模型：anthropic/claude-haiku-3-5"

Moltbot 故意拒绝**较旧/不安全的模型**（特别是那些
更容易受到提示注入的模型）。如果您看到此错误，则不再
支持该模型名称。

**修复：**
- 为提供商选择一个**最新**模型并更新您的配置或模型别名。
- 如果您不确定哪些模型可用，请运行 `moltbot models list` 或
  `moltbot models scan` 并选择一个支持的模型。
- 检查 gateway 日志以获取详细的失败原因。

另请参阅：[模型 CLI](/cli/models)和[模型提供商](/concepts/model-providers)。

### 消息未触发

**检查 1：**发送者是否在允许列表中？
```bash
moltbot status
```
在输出中查找 `AllowFrom: ...`。

**检查 2：**对于群组聊天，是否需要提及？
```bash
# 消息必须匹配 mentionPatterns 或显式提及；默认值位于通道组/guilds 中。
# 多 agent：`agents.list[].groupChat.mentionPatterns` 覆盖全局模式。
grep -n "agents\\|groupChat\\|mentionPatterns\\|channels\\.whatsapp\\.groups\\|channels\\.telegram\\.groups\\|channels\\.imessage\\.groups\\|channels\\.discord\\.guilds" \
  "${CLAWDBOT_CONFIG_PATH:-$HOME/.clawdbot/moltbot.json}"
```

**检查 3：**检查日志
```bash
moltbot logs --follow
# 或者如果您想要快速过滤器：
tail -f "$(ls -t /tmp/moltbot/moltbot-*.log | head -1)" | grep "blocked\\|skip\\|unauthorized"
```

### 配对代码未到达

如果 `dmPolicy` 是 `pairing`，未知发送者应该收到一个代码，并且他们的消息将被忽略，直到被批准。

**检查 1：**是否已经有待处理的请求在等待？
```bash
moltbot pairing list <channel>
```

默认情况下，待处理的 DM 配对请求上限为每个通道 **3 个**。如果列表已满，新请求将不会生成代码，直到一个被批准或过期。

**检查 2：**请求是否已创建但没有发送回复？
```bash
moltbot logs --follow | grep "pairing request"
```

**检查 3：**确认 `dmPolicy` 对于该通道不是 `open`/`allowlist`。

### 图像 + 提及不起作用

已知问题：当您仅发送带有提及的图像（没有其他文本）时，WhatsApp 有时不包括提及元数据。

**解决方法：**在提及时添加一些文本：
- ❌ `@clawd` + 图像
- ✅ `@clawd 检查这个` + 图像

### 会话未恢复

**检查 1：**会话文件是否在那里？
```bash
ls -la ~/.clawdbot/agents/<agentId>/sessions/
```

**检查 2：**重置窗口是否太短？
```json
{
  "session": {
    "reset": {
      "mode": "daily",
      "atHour": 4,
      "idleMinutes": 10080  // 7 天
    }
  }
}
```

**检查 3：**是否有人发送了 `/new`、`/reset` 或重置触发器？

### Agent 超时

默认超时为 30 分钟。对于长任务：

```json
{
  "reply": {
    "timeoutSeconds": 3600  // 1 小时
  }
}
```

或使用 `process` 工具将长命令后台化。

### WhatsApp 断开连接

```bash
# 检查本地状态（凭据、会话、排队的事件）
moltbot status
# 探测运行的 gateway + 通道（WA 连接 + Telegram + Discord API）
moltbot status --deep

# 查看最近的连接事件
moltbot logs --limit 200 | grep "connection\\|disconnect\\|logout"
```

**修复：**通常一旦 Gateway 运行就会自动重新连接。如果您卡住了，请重新启动 Gateway 进程（无论如何您监督它），或使用详细输出手动运行它：

```bash
moltbot gateway --verbose
```

如果您已注销/取消链接：

```bash
moltbot channels logout
trash "${CLAWDBOT_STATE_DIR:-$HOME/.clawdbot}/credentials" # 如果注销无法完全删除所有内容
moltbot channels login --verbose       # 重新扫描 QR
```

### 媒体发送失败

**检查 1：**文件路径是否有效？
```bash
ls -la /path/to/your/image.jpg
```

**检查 2：**文件是否太大？
- 图像：最大 6MB
- 音频/视频：最大 16MB
- 文档：最大 100MB

**检查 3：**检查媒体日志
```bash
grep "media\\|fetch\\|download" "$(ls -t /tmp/moltbot/moltbot-*.log | head -1)" | tail -20
```

### 高内存使用

Moltbot 将对话历史保留在内存中。

**修复：**定期重新启动或设置会话限制：
```json
{
  "session": {
    "historyLimit": 100  // 要保留的最大消息数
  }
}
```

## 常见故障排除

### "Gateway 无法启动 — 配置无效"

Moltbot 现在拒绝在配置包含未知键、格式错误的值或无效类型时启动。
这是出于安全考虑。

使用 Doctor 修复它：
```bash
moltbot doctor
moltbot doctor --fix
```

注意事项：
- `moltbot doctor` 报告每个无效条目。
- `moltbot doctor --fix` 应用迁移/修复并重写配置。
- 诊断命令如 `moltbot logs`、`moltbot health`、`moltbot status`、`moltbot gateway status` 和 `moltbot gateway probe` 即使配置无效也会运行。

### "所有模型都失败" — 我应该先检查什么？

- 为正在尝试的提供商提供了**凭据**（认证配置文件 + 环境变量）。
- **模型路由**：确认 `agents.defaults.model.primary` 和回退是您可以访问的模型。
- **Gateway 日志**在 `/tmp/moltbot/...` 中以获取确切的提供商错误。
- **模型状态**：使用 `/model status`（聊天）或 `moltbot models status`（CLI）。

### 我在我的个人 WhatsApp 号码上运行 — 为什么自聊很奇怪？

启用自聊模式并将您自己的号码列入允许列表：

```json5
{
  channels: {
    whatsapp: {
      selfChatMode: true,
      dmPolicy: "allowlist",
      allowFrom: ["+15555550123"]
    }
  }
}
```

请参阅[WhatsApp 设置](/channels/whatsapp)。

### WhatsApp 将我注销了。如何重新认证？

再次运行登录命令并扫描 QR 码：

```bash
moltbot channels login
```

### `main` 上的构建错误 — 标准修复路径是什么？

1) `git pull origin main && pnpm install`
2) `moltbot doctor`
3) 检查 GitHub 问题或 Discord
4) 临时解决方法：检查一个较旧的提交

### npm install 失败（allow-build-scripts / 缺少 tar 或 yargs）。现在怎么办？

如果您从源代码运行，请使用仓库的包管理器：**pnpm**（首选）。
仓库声明 `packageManager: "pnpm@…"`。

典型恢复：
```bash
git status   # 确保您在仓库根目录中
pnpm install
pnpm build
moltbot doctor
moltbot gateway restart
```

原因：pnpm 是此仓库配置的包管理器。

### 如何在 git 安装和 npm 安装之间切换？

使用**网站安装程序**并使用标志选择安装方法。它
就地升级并重写 gateway 服务以指向新安装。

切换**到 git 安装**：
```bash
curl -fsSL https://molt.bot/install.sh | bash -s -- --install-method git --no-onboard
```

切换**到 npm 全局**：
```bash
curl -fsSL https://molt.bot/install.sh | bash
```

注意事项：
- git 流程仅在仓库干净时才 rebase。首先提交或隐藏更改。
- 切换后，运行：
  ```bash
  moltbot doctor
  moltbot gateway restart
  ```

### Telegram 块流式传输不在工具调用之间拆分文本。为什么？

块流式传输仅发送**已完成的文本块**。您看到单条消息的常见原因：
- `agents.defaults.blockStreamingDefault` 仍然是 `"off"`。
- `channels.telegram.blockStreaming` 设置为 `false`。
- `channels.telegram.streamMode` 是 `partial` 或 `block` **并且草稿流式传输处于活动状态**
  （私人聊天 + 主题）。在这种情况下，草稿流式传输禁用块流式传输。
- 您的 `minChars` / 合并设置太高，因此块被合并。
- 模型发出一个大的文本块（没有中途回复刷新点）。

修复清单：
1) 将块流式传输设置放在 `agents.defaults` 下，而不是根目录。
2) 如果您想要真正的多消息块回复，请设置 `channels.telegram.streamMode: "off"`。
3) 在调试时使用较小的块/合并阈值。

请参阅[流式传输](/concepts/streaming)。

### 即使 `requireMention: false`，Discord 也不在我的服务器中回复。为什么？

`requireMention` 仅在通道通过允许列表后控制提及门控。
默认情况下 `channels.discord.groupPolicy` 是**allowlist**，因此必须显式启用 guilds。
如果您设置了 `channels.discord.guilds.<guildId>.channels`，则只允许列出的通道；省略它以允许 guild 中的所有通道。

修复清单：
1) 设置 `channels.discord.groupPolicy: "open"` **或**添加 guild 允许列表条目（以及可选的通道允许列表）。
2) 在 `channels.discord.guilds.<guildId>.channels` 中使用**数字通道 ID**。
3) 将 `requireMention: false` 放在**channels.discord.guilds** 下（全局或每个通道）。
   顶级 `channels.discord.requireMention` 不是支持的键。
4) 确保机器人具有**消息内容意图**和通道权限。
5) 运行 `moltbot channels status --probe` 以获取审计提示。

文档：[Discord](/channels/discord)、[通道故障排除](/channels/troubleshooting)。

### Cloud Code Assist API 错误：无效的工具架构（400）。现在怎么办？

这几乎总是一个**工具架构兼容性**问题。Cloud Code Assist
端点接受 JSON Schema 的严格子集。Moltbot 在当前的 `main` 中清理/规范化工具
架构，但修复尚未在最后一个版本中发布（截至
2026 年 1 月 13 日）。

修复清单：
1) **更新 Moltbot**：
   - 如果您可以从源代码运行，请拉取 `main` 并重新启动 gateway。
   - 否则，请等待包含架构清理器的下一个版本。
2) 避免不支持的关键字，如 `anyOf/oneOf/allOf`、`patternProperties`、
   `additionalProperties`、`minLength`、`maxLength`、`format` 等。
3) 如果您定义自定义工具，请将顶级架构保持为 `type: "object"` 与
   `properties` 和简单的枚举。

请参阅[工具](/tools)和[TypeBox 架构](/concepts/typebox)。

## macOS 特定问题

### 授予权限时应用崩溃（语音/麦克风）

如果当您在隐私提示上单击"允许"时应用消失或显示"Abort trap 6"：

**修复 1：重置 TCC 缓存**
```bash
tccutil reset All bot.molt.mac.debug
```

**修复 2：强制新 Bundle ID**
如果重置不起作用，请更改 [`scripts/package-mac-app.sh`](https://github.com/moltbot/moltbot/blob/main/scripts/package-mac-app.sh) 中的 `BUNDLE_ID`（例如，添加 `.test` 后缀）并重新构建。这会强制 macOS 将其视为新应用。

### Gateway 卡在"正在启动..."

应用程序连接到端口 `18789` 上的本地 gateway。如果它保持卡住：

**修复 1：停止监督器（首选）**
如果 gateway 由 launchd 监督，杀死 PID 只会重新生成它。首先停止监督器：
```bash
moltbot gateway status
moltbot gateway stop
# 或：launchctl bootout gui/$UID/bot.molt.gateway（替换为 bot.molt.<profile>；遗留 com.clawdbot.* 仍然有效）
```

**修复 2：端口繁忙（查找监听器）**
```bash
lsof -nP -iTCP:18789 -sTCP:LISTEN
```

如果它是无监督的进程，请先尝试优雅停止，然后升级：
```bash
kill -TERM <PID>
sleep 1
kill -9 <PID> # 最后手段
```

**修复 3：检查 CLI 安装**
确保安装了全局 `moltbot` CLI 并且与应用程序版本匹配：
```bash
moltbot --version
npm install -g moltbot@<version>
```

## 调试模式

获取详细日志记录：

```bash
# 在配置中启用跟踪日志记录：
#   ${CLAWDBOT_CONFIG_PATH:-$HOME/.clawdbot/moltbot.json} -> { logging: { level: "trace" } }
#
# 然后运行详细命令以将调试输出镜像到 stdout：
moltbot gateway --verbose
moltbot channels login --verbose
```

## 日志位置

| 日志 | 位置 |
|-----|----------|
| Gateway 文件日志（结构化） | `/tmp/moltbot/moltbot-YYYY-MM-DD.log`（或 `logging.file`） |
| Gateway 服务日志（监督器） | macOS：`$CLAWDBOT_STATE_DIR/logs/gateway.log` + `gateway.err.log`（默认：`~/.clawdbot/logs/...`；配置文件使用 `~/.clawdbot-<profile>/logs/...`）<br />Linux：`journalctl --user -u moltbot-gateway[-<profile>].service -n 200 --no-pager`<br />Windows：`schtasks /Query /TN "Moltbot Gateway (<profile>)" /V /FO LIST` |
| 会话文件 | `$CLAWDBOT_STATE_DIR/agents/<agentId>/sessions/` |
| 媒体缓存 | `$CLAWDBOT_STATE_DIR/media/` |
| 凭据 | `$CLAWDBOT_STATE_DIR/credentials/` |

## 健康检查

```bash
# 监督器 + 探测目标 + 配置路径
moltbot gateway status
# 包括系统级扫描（遗留/额外服务、端口监听器）
moltbot gateway status --deep

# gateway 是否可达？
moltbot health --json
# 如果失败，请使用连接详细信息重新运行：
moltbot health --verbose

# 默认端口上是否有东西在监听？
lsof -nP -iTCP:18789 -sTCP:LISTEN

# 最近的活动（RPC 日志尾部）
moltbot logs --follow
# 如果 RPC 关闭时的回退
tail -20 /tmp/moltbot/moltbot-*.log
```

## 重置所有内容

核选项：

```bash
moltbot gateway stop
# 如果您安装了服务并想要干净安装：
# moltbot gateway uninstall

trash "${CLAWDBOT_STATE_DIR:-$HOME/.clawdbot}"
moltbot channels login         # 重新配对 WhatsApp
moltbot gateway restart           # 或：moltbot gateway
```

⚠️ 这会丢失所有会话并需要重新配对 WhatsApp。

## 获取帮助

1. 首先检查日志：`/tmp/moltbot/`（默认：`moltbot-YYYY-MM-DD.log`，或您配置的 `logging.file`）
2. 在 GitHub 上搜索现有问题
3. 打开一个新问题，包括：
   - Moltbot 版本
   - 相关日志片段
   - 重现步骤
   - 您的配置（编辑机密！）

---

*"您是否尝试过将其关闭并重新打开？"* — 每个 IT 人员

🦞🔧

### 浏览器未启动（Linux）

如果您看到 `"Failed to start Chrome CDP on port 18800"`：

**最可能的原因：**Ubuntu 上的 Snap 打包的 Chromium。

**快速修复：**改为安装 Google Chrome：
```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome-stable_current_amd64.deb
```

然后在配置中设置：
```json
{
  "browser": {
    "executablePath": "/usr/bin/google-chrome-stable"
  }
}
```

**完整指南：**请参阅[browser-linux-troubleshooting](/tools/browser-linux-troubleshooting)
