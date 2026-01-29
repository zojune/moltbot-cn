---
summary: "初学者指南：从零到第一条消息（向导、认证、频道、配对）"
read_when:
  - 从零开始首次设置
  - 你想要从安装 → 入门 → 第一条消息的最快路径
---

# 入门指南

目标：从**零** → **第一次工作的聊天**（使用合理的默认值）尽快。

最快的聊天：打开控制 UI（无需频道设置）。运行 `moltbot dashboard`
并在浏览器中聊天，或打开网关主机上的 `http://127.0.0.1:18789/`。
文档：[仪表板](/web/dashboard) 和 [控制 UI](/web/control-ui)。

推荐路径：使用 **CLI 入门向导**（`moltbot onboard`）。它设置：
- 模型/认证（OAuth 推荐）
- 网关设置
- 频道（WhatsApp/Telegram/Discord/Mattermost（插件）/...）
- 配对默认值（安全 DM）
- 工作区引导 + 技能
- 可选后台服务

如果你想要更深入的参考页面，请跳转到：[向导](/start/wizard)、[设置](/start/setup)、[配对](/start/pairing)、[安全](/gateway/security)。

沙箱说明：`agents.defaults.sandbox.mode: "non-main"` 使用 `session.mainKey`（默认 `"main"`），
因此群组/频道会话被沙箱化。如果你希望主代理始终在主机上运行，
请设置明确的每代理覆盖：

```json
{
  "routing": {
    "agents": {
      "main": {
        "workspace": "~/clawd",
        "sandbox": { "mode": "off" }
      }
    }
  }
}
```

## 0) 前提条件

- Node `>=22`
- `pnpm`（可选；如果从源代码构建则推荐）
- **推荐：** Brave Search API 密钥用于网络搜索。最简单的路径：
  `moltbot configure --section web`（存储 `tools.web.search.apiKey`）。
  参见 [网络工具](/tools/web)。

macOS：如果你计划构建应用，请安装 Xcode / CLT。对于仅 CLI + 网关，Node 就足够了。
Windows：使用 **WSL2**（推荐 Ubuntu）。强烈推荐 WSL2；原生 Windows 未经测试，问题更多，工具兼容性更差。首先安装 WSL2，然后在 WSL 内运行 Linux 步骤。参见 [Windows (WSL2)](/platforms/windows)。

## 1) 安装 CLI（推荐）

```bash
curl -fsSL https://molt.bot/install.sh | bash
```

安装程序选项（安装方法、非交互式、从 GitHub）：[安装](/install)。

Windows (PowerShell)：

```powershell
iwr -useb https://molt.bot/install.ps1 | iex
```

替代方案（全局安装）：

```bash
npm install -g moltbot@latest
```

```bash
pnpm add -g moltbot@latest
```

## 2) 运行入门向导（并安装服务）

```bash
moltbot onboard --install-daemon
```

你将选择：
- **本地 vs 远程** 网关
- **认证**：OpenAI Code (Codex) 订阅（OAuth）或 API 密钥。对于 Anthropic，我们推荐 API 密钥；也支持 `claude setup-token`。
- **提供商**：WhatsApp QR 登录、Telegram/Discord 机器人令牌、Mattermost 插件令牌等。
- **守护进程**：后台安装（launchd/systemd；WSL2 使用 systemd）
  - **运行时**：Node（推荐；WhatsApp/Telegram 必需）。**不推荐** Bun。
- **网关令牌**：向导默认生成一个（即使在回环上）并将其存储在 `gateway.auth.token` 中。

向导文档：[向导](/start/wizard)

### 认证：它在哪里（重要）

- **推荐的 Anthropic 路径：**设置 API 密钥（向导可以存储它供服务使用）。如果你想重用 Claude Code 凭据，也支持 `claude setup-token`。

- OAuth 凭据（旧版导入）：`~/.clawdbot/credentials/oauth.json`
- 认证配置文件（OAuth + API 密钥）：`~/.clawdbot/agents/<agentId>/agent/auth-profiles.json`

无头/服务器提示：首先在正常机器上执行 OAuth，然后将 `oauth.json` 复制到网关主机。

## 3) 启动网关

如果你在入门期间安装了服务，网关应该已经在运行：

```bash
moltbot gateway status
```

手动运行（前台）：

```bash
moltbot gateway --port 18789 --verbose
```

仪表板（本地回环）：`http://127.0.0.1:18789/`
如果配置了令牌，请将其粘贴到控制 UI 设置中（存储为 `connect.params.auth.token`）。

⚠️ **Bun 警告（WhatsApp + Telegram）：** Bun 与这些
频道有已知问题。如果你使用 WhatsApp 或 Telegram，请使用 **Node** 运行网关。

## 3.5) 快速验证（2 分钟）

```bash
moltbot status
moltbot health
moltbot security audit --deep
```

## 4) 配对 + 连接你的第一个聊天表面

### WhatsApp（QR 登录）

```bash
moltbot channels login
```

通过 WhatsApp → 设置 → 链接设备扫描。

WhatsApp 文档：[WhatsApp](/channels/whatsapp)

### Telegram / Discord / 其他

向导可以为你写入令牌/配置。如果你更喜欢手动配置，请从以下开始：
- Telegram：[Telegram](/channels/telegram)
- Discord：[Discord](/channels/discord)
- Mattermost（插件）：[Mattermost](/channels/mattermost)

**Telegram DM 提示：**你的第一条 DM 返回一个配对代码。批准它（见下一步），否则机器人不会响应。

## 5) DM 安全（配对审批）

默认姿态：未知 DM 获得一个短代码，在批准之前不会处理消息。
如果你的第一条 DM 没有回复，请批准配对：

```bash
moltbot pairing list whatsapp
moltbot pairing approve whatsapp <code>
```

配对文档：[配对](/start/pairing)

## 从源代码（开发）

如果你正在开发 Moltbot 本身，请从源代码运行：

```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖
pnpm build
moltbot onboard --install-daemon
```

如果你还没有全局安装，请通过 `pnpm moltbot ...` 从仓库运行入门步骤。
`pnpm build` 也会捆绑 A2UI 资产；如果只需要运行该步骤，请使用 `pnpm canvas:a2ui:bundle`。

网关（从此仓库）：

```bash
node moltbot.mjs gateway --port 18789 --verbose
```

## 7) 端到端验证

在新终端中，发送测试消息：

```bash
moltbot message send --target +15555550123 --message "Hello from Moltbot"
```

如果 `moltbot health` 显示"未配置认证"，请返回向导并设置 OAuth/密钥认证 — 代理将无法响应。

提示：`moltbot status --all` 是最好的可粘贴、只读调试报告。
健康探测：`moltbot health`（或 `moltbot status --deep`）询问运行的网关以获取健康快照。

## 后续步骤（可选，但很棒）

- macOS 菜单栏应用 + 语音唤醒：[macOS 应用](/platforms/macos)
- iOS/Android 节点（Canvas/摄像头/语音）：[节点](/nodes)
- 远程访问（SSH 隧道 / Tailscale Serve）：[远程访问](/gateway/remote) 和 [Tailscale](/gateway/tailscale)
- 始终在线 / VPN 设置：[远程访问](/gateway/remote)、[exe.dev](/platforms/exe-dev)、[Hetzner](/platforms/hetzner)、[macOS 远程](/platforms/mac/remote)
