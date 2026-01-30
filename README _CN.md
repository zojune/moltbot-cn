# 🦞 Moltbot — 个人 AI 助手

<p align="center">
  <img src="https://raw.githubusercontent.com/moltbot/moltbot/main/docs/whatsapp-clawd.jpg" alt="Clawdbot" width="400">
</p>

<p align="center">
  <strong>蜕皮吧！蜕皮吧！</strong>
</p>

<p align="center">
  <a href="https://github.com/moltbot/moltbot/actions/workflows/ci.yml?branch=main"><img src="https://img.shields.io/github/actions/workflow/status/moltbot/moltbot/ci.yml?branch=main&style=for-the-badge" alt="CI 状态"></a>
  <a href="https://github.com/moltbot/moltbot/releases"><img src="https://img.shields.io/github/v/release/moltbot/moltbot?include_prereleases&style=for-the-badge" alt="GitHub 发布"></a>
  <a href="https://deepwiki.com/moltbot/moltbot"><img src="https://img.shields.io/badge/DeepWiki-moltbot-111111?style=for-the-badge" alt="DeepWiki"></a>
  <a href="https://discord.gg/clawd"><img src="https://img.shields.io/discord/1456350064065904867?label=Discord&logo=discord&logoColor=white&color=5865F2&style=for-the-badge" alt="Discord"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue.svg?style=for-the-badge" alt="MIT 许可证"></a>
</p>

**Moltbot** 是一个运行在你自己设备上的*个人 AI 助手*。
它在你已经使用的频道上回答你的问题（WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage、Microsoft Teams、WebChat），以及扩展频道如 BlueBubbles、Matrix、Zalo 和 Zalo Personal。它可以在 macOS/iOS/Android 上说话和倾听，并可以渲染一个你控制的实时画布。网关只是控制平面——产品才是助手。

如果你想要一个个人的、单用户的助手，感觉本地、快速且始终在线，这就是它。

[网站](https://molt.bot) · [文档](https://docs.molt.bot) · [快速入门](https://docs.molt.bot/start/getting-started) · [更新](https://docs.molt.bot/install/updating) · [展示](https://docs.molt.bot/start/showcase) · [FAQ](https://docs.molt.bot/start/faq) · [向导](https://docs.molt.bot/start/wizard) · [Nix](https://github.com/moltbot/nix-clawdbot) · [Docker](https://docs.molt.bot/install/docker) · [Discord](https://discord.gg/clawd)

推荐设置：运行入门向导（`moltbot onboard`）。它会引导你完成网关、工作区、频道和技能的设置。CLI 向导是推荐的路径，适用于 **macOS、Linux 和 Windows（通过 WSL2；强烈推荐）**。
支持 npm、pnpm 或 bun。
新安装？从这里开始：[快速入门](https://docs.molt.bot/start/getting-started)

**订阅（OAuth）：**
- **[Anthropic](https://www.anthropic.com/)** (Claude Pro/Max)
- **[OpenAI](https://openai.com/)** (ChatGPT/Codex)

模型说明：虽然支持任何模型，但我强烈推荐 **Anthropic Pro/Max (100/200) + Opus 4.5** 以获得长上下文强度和更好的提示注入抵抗能力。参见[入门指南](https://docs.molt.bot/start/onboarding)。

## 模型（选择 + 认证）

- 模型配置 + CLI：[模型](https://docs.molt.bot/concepts/models)
- 认证配置轮换（OAuth vs API 密钥）+ 故障转移：[模型故障转移](https://docs.molt.bot/concepts/model-failover)

## 安装（推荐）

运行时要求：**Node ≥22**。

```bash
npm install -g moltbot@latest
# 或：pnpm add -g moltbot@latest

moltbot onboard --install-daemon
```

向导会安装网关守护进程（launchd/systemd 用户服务）使其保持运行。
遗留说明：`clawdbot` 作为兼容性垫片仍然可用。

## 快速入门（简述）

运行时要求：**Node ≥22**。

完整的新手指南（认证、配对、频道）：[快速入门](https://docs.molt.bot/start/getting-started)

```bash
moltbot onboard --install-daemon

moltbot gateway --port 18789 --verbose

# 发送消息
moltbot message send --to +1234567890 --message "来自 Moltbot 的问候"

# 与助手对话（可选择传回到任何已连接的频道：WhatsApp/Telegram/Slack/Discord/Google Chat/Signal/iMessage/BlueBubbles/Microsoft Teams/Matrix/Zalo/Zalo Personal/WebChat）
moltbot agent --message "发货清单" --thinking high
```

正在升级？[更新指南](https://docs.molt.bot/install/updating)（并运行 `moltbot doctor`）。

## 开发频道

- **stable**：标记的发布版本（`vYYYY.M.D` 或 `vYYYY.M.D-<patch>`），npm dist-tag `latest`。
- **beta**：预发布标签（`vYYYY.M.D-beta.N`），npm dist-tag `beta`（可能缺少 macOS 应用）。
- **dev**：`main` 分支的最新状态，npm dist-tag `dev`（发布时）。

切换频道（git + npm）：`moltbot update --channel stable|beta|dev`。
详情：[开发频道](https://docs.molt.bot/install/development-channels)。

## 从源码构建（开发）

推荐使用 `pnpm` 进行源码构建。Bun 是可选的，用于直接运行 TypeScript。

```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot

pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖
pnpm build

pnpm moltbot onboard --install-daemon

# 开发循环（TS 更改时自动重新加载）
pnpm gateway:watch
```

注意：`pnpm moltbot ...` 直接运行 TypeScript（通过 `tsx`）。`pnpm build` 生成 `dist/` 用于通过 Node / 打包的 `moltbot` 二进制文件运行。

## 安全默认值（私信访问）

Moltbot 连接到真实的消息界面。将入站私信视为**不受信任的输入**。

完整安全指南：[安全](https://docs.molt.bot/gateway/security)

Telegram/WhatsApp/Signal/iMessage/Microsoft Teams/Discord/Google Chat/Slack 上的默认行为：
- **私信配对**（`dmPolicy="pairing"` / `channels.discord.dm.policy="pairing"` / `channels.slack.dm.policy="pairing"`）：未知发送者会收到一个简短的配对代码，机器人不会处理他们的消息。
- 使用以下命令批准：`moltbot pairing approve <channel> <code>`（然后将发送者添加到本地允许列表存储）。
- 公开入站私信需要明确选择加入：设置 `dmPolicy="open"` 并在频道允许列表中包含 `"*"`（`allowFrom` / `channels.discord.dm.allowFrom` / `channels.slack.dm.allowFrom`）。

运行 `moltbot doctor` 来暴露有风险/配置错误的私信策略。

## 亮点

- **[本地优先网关](https://docs.molt.bot/gateway)** — 会话、频道、工具和事件的单一控制平面。
- **[多频道收件箱](https://docs.molt.bot/channels)** — WhatsApp、Telegram、Slack、Discord、Google Chat、Signal、iMessage、BlueBubbles、Microsoft Teams、Matrix、Zalo、Zalo Personal、WebChat、macOS、iOS/Android。
- **[多代理路由](https://docs.molt.bot/gateway/configuration)** — 将入站频道/账户/对等点路由到隔离的代理（工作区 + 每代理会话）。
- **[语音唤醒](https://docs.molt.bot/nodes/voicewake) + [对话模式](https://docs.molt.bot/nodes/talk)** — 适用于 macOS/iOS/Android 的始终在线语音，配合 ElevenLabs。
- **[实时画布](https://docs.molt.bot/platforms/mac/canvas)** — 由代理驱动的视觉工作空间，具有 [A2UI](https://docs.molt.bot/platforms/mac/canvas#canvas-a2ui)。
- **[一流工具](https://docs.molt.bot/tools)** — 浏览器、画布、节点、cron、会话以及 Discord/Slack 操作。
- **[配套应用](https://docs.molt.bot/platforms/macos)** — macOS 菜单栏应用 + iOS/Android [节点](https://docs.molt.bot/nodes)。
- **[入门](https://docs.molt.bot/start/wizard) + [技能](https://docs.molt.bot/tools/skills)** — 向导驱动的设置，包含捆绑/管理/工作区技能。

## Star 历史

[![Star 历史图表](https://api.star-history.com/svg?repos=moltbot/moltbot&type=date&legend=top-left)](https://www.star-history.com/#moltbot/moltbot&type=date&legend=top-left)

## 我们到目前为止构建的所有内容

### 核心平台
- [网关 WS 控制平面](https://docs.molt.bot/gateway)，包含会话、在线状态、配置、cron、webhook、[控制 UI](https://docs.molt.bot/web)和[画布主机](https://docs.molt.bot/platforms/mac/canvas#canvas-a2ui)。
- [CLI 界面](https://docs.molt.bot/tools/agent-send)：网关、代理、发送、[向导](https://docs.molt.bot/start/wizard)和[医生](https://docs.molt.bot/gateway/doctor)。
- [Pi 代理运行时](https://docs.molt.bot/concepts/agent)（RPC 模式），支持工具流和块流。
- [会话模型](https://docs.molt.bot/concepts/session)：`main` 用于直接聊天、群组隔离、激活模式、队列模式、回复。群组规则：[群组](https://docs.molt.bot/concepts/groups)。
- [媒体管道](https://docs.molt.bot/nodes/images)：图片/音频/视频、转录钩子、大小限制、临时文件生命周期。音频详情：[音频](https://docs.molt.bot/nodes/audio)。

### 频道
- [频道](https://docs.molt.bot/channels)：[WhatsApp](https://docs.molt.bot/channels/whatsapp)（Baileys）、[Telegram](https://docs.molt.bot/channels/telegram)（grammY）、[Slack](https://docs.molt.bot/channels/slack)（Bolt）、[Discord](https://docs.molt.bot/channels/discord)（discord.js）、[Google Chat](https://docs.molt.bot/channels/googlechat)（Chat API）、[Signal](https://docs.molt.bot/channels/signal)（signal-cli）、[iMessage](https://docs.molt.bot/channels/imessage)（imsg）、[BlueBubbles](https://docs.molt.bot/channels/bluebubbles)（扩展）、[Microsoft Teams](https://docs.molt.bot/channels/msteams)（扩展）、[Matrix](https://docs.molt.bot/channels/matrix)（扩展）、[Zalo](https://docs.molt.bot/channels/zalo)（扩展）、[Zalo Personal](https://docs.molt.bot/channels/zalouser)（扩展）、[WebChat](https://docs.molt.bot/web/webchat)。
- [群组路由](https://docs.molt.bot/concepts/group-messages)：提及过滤、回复标签、每频道分块和路由。频道规则：[频道](https://docs.molt.bot/channels)。

### 应用 + 节点
- [macOS 应用](https://docs.molt.bot/platforms/macos)：菜单栏控制平面、[语音唤醒](https://docs.molt.bot/nodes/voicewake)/PTT、[对话模式](https://docs.molt.bot/nodes/talk)覆盖层、[WebChat](https://docs.molt.bot/web/webchat)、调试工具、[远程网关](https://docs.molt.bot/gateway/remote)控制。
- [iOS 节点](https://docs.molt.bot/platforms/ios)：[画布](https://docs.molt.bot/platforms/mac/canvas)、[语音唤醒](https://docs.molt.bot/nodes/voicewake)、[对话模式](https://docs.molt.bot/nodes/talk)、相机、屏幕录制、Bonjour 配对。
- [Android 节点](https://docs.molt.bot/platforms/android)：[画布](https://docs.molt.bot/platforms/mac/canvas)、[对话模式](https://docs.molt.bot/nodes/talk)、相机、屏幕录制、可选的 SMS。
- [macOS 节点模式](https://docs.molt.bot/nodes)：system.run/notify + 画布/相机暴露。

### 工具 + 自动化
- [浏览器控制](https://docs.molt.bot/tools/browser)：专用的 moltbot Chrome/Chromium、快照、操作、上传、配置文件。
- [画布](https://docs.molt.bot/platforms/mac/canvas)：[A2UI](https://docs.molt.bot/platforms/mac/canvas#canvas-a2ui)推送/重置、eval、快照。
- [节点](https://docs.molt.bot/nodes)：相机抓拍/剪辑、屏幕录制、[location.get](https://docs.molt.bot/nodes/location-command)、通知。
- [Cron + 唤醒](https://docs.molt.bot/automation/cron-jobs)；[webhook](https://docs.molt.bot/automation/webhook)；[Gmail Pub/Sub](https://docs.molt.bot/automation/gmail-pubsub)。
- [技能平台](https://docs.molt.bot/tools/skills)：捆绑、管理和工作区技能，具有安装门控 + UI。

### 运行时 + 安全
- [频道路由](https://docs.molt.bot/concepts/channel-routing)、[重试策略](https://docs.molt.bot/concepts/retry)和[流式传输/分块](https://docs.molt.bot/concepts/streaming)。
- [在线状态](https://docs.molt.bot/concepts/presence)、[输入指示器](https://docs.molt.bot/concepts/typing-indicators)和[使用跟踪](https://docs.molt.bot/concepts/usage-tracking)。
- [模型](https://docs.molt.bot/concepts/models)、[模型故障转移](https://docs.molt.bot/concepts/model-failover)和[会话修剪](https://docs.molt.bot/concepts/session-pruning)。
- [安全](https://docs.molt.bot/gateway/security)和[故障排除](https://docs.molt.bot/channels/troubleshooting)。

### 运维 + 打包
- 直接从网关提供[控制 UI](https://docs.molt.bot/web) + [WebChat](https://docs.molt.bot/web/webchat)。
- [Tailscale Serve/Funnel](https://docs.molt.bot/gateway/tailscale)或[SSH 隧道](https://docs.molt.bot/gateway/remote)，带有令牌/密码认证。
- [Nix 模式](https://docs.molt.bot/install/nix)用于声明性配置；基于[Docker](https://docs.molt.bot/install/docker)的安装。
- [医生](https://docs.molt.bot/gateway/doctor)迁移、[日志记录](https://docs.molt.bot/logging)。

## 工作原理（简述）

```
WhatsApp / Telegram / Slack / Discord / Google Chat / Signal / iMessage / BlueBubbles / Microsoft Teams / Matrix / Zalo / Zalo Personal / WebChat
               │
               ▼
┌───────────────────────────────┐
│            网关                │
│       （控制平面）             │
│     ws://127.0.0.1:18789      │
└──────────────┬────────────────┘
               │
               ├─ Pi 代理（RPC）
               ├─ CLI（moltbot …）
               ├─ WebChat UI
               ├─ macOS 应用
               └─ iOS / Android 节点
```

## 关键子系统

- **[网关 WebSocket 网络](https://docs.molt.bot/concepts/architecture)** — 客户端、工具和事件的单一 WS 控制平面（加上运维：[网关运行手册](https://docs.molt.bot/gateway)）。
- **[Tailscale 暴露](https://docs.molt.bot/gateway/tailscale)** — 用于网关仪表板 + WS 的 Serve/Funnel（远程访问：[远程](https://docs.molt.bot/gateway/remote)）。
- **[浏览器控制](https://docs.molt.bot/tools/browser)** — 由 moltbot 管理的 Chrome/Chromium，具有 CDP 控制。
- **[画布 + A2UI](https://docs.molt.bot/platforms/mac/canvas)** — 由代理驱动的视觉工作空间（A2UI 主机：[画布/A2UI](https://docs.molt.bot/platforms/mac/canvas#canvas-a2ui)）。
- **[语音唤醒](https://docs.molt.bot/nodes/voicewake) + [对话模式](https://docs.molt.bot/nodes/talk)** — 始终在线的语音和连续对话。
- **[节点](https://docs.molt.bot/nodes)** — 画布、相机抓拍/剪辑、屏幕录制、`location.get`、通知，加上仅限 macOS 的 `system.run`/`system.notify`。

## Tailscale 访问（网关仪表板）

Moltbot 可以自动配置 Tailscale **Serve**（仅限 tailnet）或 **Funnel**（公开），同时网关保持绑定到回环地址。配置 `gateway.tailscale.mode`：

- `off`：没有 Tailscale 自动化（默认）。
- `serve`：通过 `tailscale serve` 进行仅限 tailnet 的 HTTPS（默认使用 Tailscale 身份标头）。
- `funnel`：通过 `tailscale funnel` 进行公开 HTTPS（需要共享密码 auth）。

说明：
- 启用 Serve/Funnel 时，`gateway.bind` 必须保持 `loopback`（Moltbot 强制执行此操作）。
- 可以通过设置 `gateway.auth.mode: "password"` 或 `gateway.auth.allowTailscale: false` 来强制 Serve 需要密码。
- 除非设置了 `gateway.auth.mode: "password"`，否则 Funnel 拒绝启动。
- 可选：`gateway.tailscale.resetOnExit` 在关闭时撤消 Serve/Funnel。

详情：[Tailscale 指南](https://docs.molt.bot/gateway/tailscale) · [Web 表面](https://docs.molt.bot/web)

## 远程网关（Linux 很棒）

在小型 Linux 实例上运行网关是完全没问题的。客户端（macOS 应用、CLI、WebChat）可以通过 **Tailscale Serve/Funnel** 或 **SSH 隧道**连接，你仍然可以配对设备节点（macOS/iOS/Android）以在需要时执行设备本地操作。

- **网关主机**默认运行 exec 工具和频道连接。
- **设备节点**通过 `node.invoke` 运行设备本地操作（`system.run`、相机、屏幕录制、通知）。
简而言之：exec 在网关所在的地方运行；设备操作在设备所在的地方运行。

详情：[远程访问](https://docs.molt.bot/gateway/remote) · [节点](https://docs.molt.bot/nodes) · [安全](https://docs.molt.bot/gateway/security)

## 通过 Gateway 协议进行 macOS 权限管理

macOS 应用可以在**节点模式**下运行，并通过 Gateway WebSocket（`node.list` / `node.describe`）通告其功能 + 权限映射。然后客户端可以通过 `node.invoke` 执行本地操作：

- `system.run` 运行本地命令并返回 stdout/stderr/退出代码；设置 `needsScreenRecording: true` 以需要屏幕录制权限（否则你会得到 `PERMISSION_MISSING`）。
- `system.notify` 发布用户通知并在拒绝通知时失败。
- `canvas.*`、`camera.*`、`screen.record` 和 `location.get` 也通过 `node.invoke` 路由，并遵循 TCC 权限状态。

提升的 bash（主机权限）与 macOS TCC 分开：

- 使用 `/elevated on|off` 在启用 + 允许列表时切换每会话提升访问。
- Gateway 通过 `sessions.patch`（WS 方法）持久化每会话切换，与 `thinkingLevel`、`verboseLevel`、`model`、`sendPolicy` 和 `groupActivation` 一起。

详情：[节点](https://docs.molt.bot/nodes) · [macOS 应用](https://docs.molt.bot/platforms/macos) · [Gateway 协议](https://docs.molt.bot/concepts/architecture)

## Agent 到 Agent（sessions_* 工具）

- 使用这些工具在没有跳过聊天界面的情况下协调跨会话的工作。
- `sessions_list` — 发现活动会话（agents）及其元数据。
- `sessions_history` — 获取会话的脚本日志。
- `sessions_send` — 向另一个会话发送消息；可选的回复回乒乓 + 公告步骤（`REPLY_SKIP`、`ANNOUNCE_SKIP`）。

详情：[会话工具](https://docs.molt.bot/concepts/session-tool)

## 技能注册表（ClawdHub）

ClawdHub 是一个最小的技能注册表。启用 ClawdHub 后，agent 可以自动搜索技能并根据需要拉入新技能。

[ClawdHub](https://ClawdHub.com)

## 聊天命令

在 WhatsApp/Telegram/Slack/Google Chat/Microsoft Teams/WebChat 中发送这些命令（组命令仅所有者可用）：

- `/status` — 紧凑的会话状态（模型 + 令牌，可用时包括成本）
- `/new` 或 `/reset` — 重置会话
- `/compact` — 压缩会话上下文（摘要）
- `/think <level>` — off|minimal|low|medium|high|xhigh（仅 GPT-5.2 + Codex 模型）
- `/verbose on|off`
- `/usage off|tokens|full` — 每回复使用页脚
- `/restart` — 重启网关（组中仅所有者）
- `/activation mention|always` — 组激活切换（仅组）

## 应用（可选）

仅网关就能提供出色的体验。所有应用都是可选的，并添加额外功能。

如果你计划构建/运行配套应用，请遵循以下平台运行手册。

### macOS（Moltbot.app）（可选）

- 网关和健康状况的菜单栏控制。
- 语音唤醒 + 按下通话覆盖层。
- WebChat + 调试工具。
- 通过 SSH 进行远程网关控制。

注意：签名的构建需要 macOS 权限才能在重建后保持有效（参见 `docs/mac/permissions.md`）。

### iOS 节点（可选）

- 通过网桥作为节点配对。
- 语音触发转发 + 画布表面。
- 通过 `moltbot nodes …` 控制。

运行手册：[iOS 连接](https://docs.molt.bot/platforms/ios)。

### Android 节点（可选）

- 通过与 iOS 相同的网桥 + 配对流程配对。
- 暴露画布、相机和屏幕捕获命令。
- 运行手册：[Android 连接](https://docs.molt.bot/platforms/android)。

## Agent 工作区 + 技能

- 工作区根目录：`~/clawd`（可通过 `agents.defaults.workspace` 配置）。
- 注入的提示文件：`AGENTS.md`、`SOUL.md`、`TOOLS.md`。
- 技能：`~/clawd/skills/<skill>/SKILL.md`。

## 配置

最小的 `~/.clawdbot/moltbot.json`（模型 + 默认值）：

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  }
}
```

[完整配置参考（所有键 + 示例）。](https://docs.molt.bot/gateway/configuration)

## 安全模型（重要）

- **默认：**工具在**主**会话的主机上运行，因此当只有你时，agent 具有完全访问权限。
- **组/频道安全：**设置 `agents.defaults.sandbox.mode: "non-main"`以在每会话 Docker 沙箱中运行**非主会话**（组/频道）；然后 bash 在这些会话的 Docker 中运行。
- **沙箱默认值：**允许列表 `bash`、`process`、`read`、`write`、`edit`、`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`；拒绝列表 `browser`、`canvas`、`nodes`、`cron`、`discord`、`gateway`。

详情：[安全指南](https://docs.molt.bot/gateway/security) · [Docker + 沙箱](https://docs.molt.bot/install/docker) · [沙箱配置](https://docs.molt.bot/gateway/configuration)

### [WhatsApp](https://docs.molt.bot/channels/whatsapp)

- 链接设备：`pnpm moltbot channels login`（将凭据存储在 `~/.clawdbot/credentials` 中）。
- 通过 `channels.whatsapp.allowFrom` 允许谁可以与助手交谈。
- 如果设置了 `channels.whatsapp.groups`，它会成为组允许列表；包含 `"*"` 以允许所有。

### [Telegram](https://docs.molt.bot/channels/telegram)

- 设置 `TELEGRAM_BOT_TOKEN` 或 `channels.telegram.botToken`（环境获胜）。
- 可选：设置 `channels.telegram.groups`（带有 `channels.telegram.groups."*".requireMention`）；当设置时，它是组允许列表（包含 `"*"` 以允许所有）。还需要 `channels.telegram.allowFrom` 或 `channels.telegram.webhookUrl`。

```json5
{
  channels: {
    telegram: {
      botToken: "123456:ABCDEF"
    }
  }
}
```

### [Slack](https://docs.molt.bot/channels/slack)

- 设置 `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN`（或 `channels.slack.botToken` + `channels.slack.appToken`）。

### [Discord](https://docs.molt.bot/channels/discord)

- 设置 `DISCORD_BOT_TOKEN` 或 `channels.discord.token`（环境获胜）。
- 可选：设置 `commands.native`、`commands.text` 或 `commands.useAccessGroups`，以及根据需要设置 `channels.discord.dm.allowFrom`、`channels.discord.guilds` 或 `channels.discord.mediaMaxMb`。

```json5
{
  channels: {
    discord: {
      token: "1234abcd"
    }
  }
}
```

### [Signal](https://docs.molt.bot/channels/signal)

- 需要 `signal-cli` 和 `channels.signal` 配置部分。

### [iMessage](https://docs.molt.bot/channels/imessage)

- 仅 macOS；Messages 必须已登录。
- 如果设置了 `channels.imessage.groups`，它会成为组允许列表；包含 `"*"` 以允许所有。

### [Microsoft Teams](https://docs.molt.bot/channels/msteams)

- 配置 Teams 应用 + Bot 框架，然后添加 `msteams` 配置部分。
- 通过 `msteams.allowFrom` 允许谁可以交谈；通过 `msteams.groupAllowFrom` 或 `msteams.groupPolicy: "open"` 进行组访问。

### [WebChat](https://docs.molt.bot/web/webchat)

- 使用 Gateway WebSocket；没有单独的 WebChat 端口/配置。

浏览器控制（可选）：

```json5
{
  browser: {
    enabled: true,
    color: "#FF4500"
  }
}
```

## 文档

当你通过入门流程并想要更深入的参考时，请使用这些内容。
- [从文档索引开始，了解导航和"什么在哪里"。](https://docs.molt.bot)
- [阅读架构概述，了解网关 + 协议模型。](https://docs.molt.bot/concepts/architecture)
- [当你需要每个键和示例时，使用完整的配置参考。](https://docs.molt.bot/gateway/configuration)
- [按照运行手册运行网关。](https://docs.molt.bot/gateway)
- [了解控制 UI/Web 表面的工作原理以及如何安全地暴露它们。](https://docs.molt.bot/web)
- [通过 SSH 隧道或 tailnet 了解远程访问。](https://docs.molt.bot/gateway/remote)
- [遵循入门向导流程进行引导式设置。](https://docs.molt.bot/start/wizard)
- [通过 webhook 表面连接外部触发器。](https://docs.molt.bot/automation/webhook)
- [设置 Gmail Pub/Sub 触发器。](https://docs.molt.bot/automation/gmail-pubsub)
- [了解 macOS 菜单栏配套详情。](https://docs.molt.bot/platforms/mac/menu-bar)
- [平台指南：Windows (WSL2)](https://docs.molt.bot/platforms/windows)、[Linux](https://docs.molt.bot/platforms/linux)、[macOS](https://docs.molt.bot/platforms/macos)、[iOS](https://docs.molt.bot/platforms/ios)、[Android](https://docs.molt.bot/platforms/android)
- [使用故障排除指南调试常见故障。](https://docs.molt.bot/channels/troubleshooting)
- [在暴露任何内容之前查看安全指南。](https://docs.molt.bot/gateway/security)

## 高级文档（发现 + 控制）

- [发现 + 传输](https://docs.molt.bot/gateway/discovery)
- [Bonjour/mDNS](https://docs.molt.bot/gateway/bonjour)
- [网关配对](https://docs.molt.bot/gateway/pairing)
- [远程网关 README](https://docs.molt.bot/gateway/remote-gateway-readme)
- [控制 UI](https://docs.molt.bot/web/control-ui)
- [仪表板](https://docs.molt.bot/web/dashboard)

## 运维和故障排除

- [健康检查](https://docs.molt.bot/gateway/health)
- [网关锁](https://docs.molt.bot/gateway/gateway-lock)
- [后台进程](https://docs.molt.bot/gateway/background-process)
- [浏览器故障排除（Linux）](https://docs.molt.bot/tools/browser-linux-troubleshooting)
- [日志记录](https://docs.molt.bot/logging)

## 深入探讨

- [Agent 循环](https://docs.molt.bot/concepts/agent-loop)
- [在线状态](https://docs.molt.bot/concepts/presence)
- [TypeBox 架构](https://docs.molt.bot/concepts/typebox)
- [RPC 适配器](https://docs.molt.bot/reference/rpc)
- [队列](https://docs.molt.bot/concepts/queue)

## 工作区和技能

- [技能配置](https://docs.molt.bot/tools/skills-config)
- [默认 AGENTS](https://docs.molt.bot/reference/AGENTS.default)
- [模板：AGENTS](https://docs.molt.bot/reference/templates/AGENTS)
- [模板：BOOTSTRAP](https://docs.molt.bot/reference/templates/BOOTSTRAP)
- [模板：IDENTITY](https://docs.molt.bot/reference/templates/IDENTITY)
- [模板：SOUL](https://docs.molt.bot/reference/templates/SOUL)
- [模板：TOOLS](https://docs.molt.bot/reference/templates/TOOLS)
- [模板：USER](https://docs.molt.bot/reference/templates/USER)

## 平台内部

- [macOS 开发设置](https://docs.molt.bot/platforms/mac/dev-setup)
- [macOS 菜单栏](https://docs.molt.bot/platforms/mac/menu-bar)
- [macOS 语音唤醒](https://docs.molt.bot/platforms/mac/voicewake)
- [iOS 节点](https://docs.molt.bot/platforms/ios)
- [Android 节点](https://docs.molt.bot/platforms/android)
- [Windows (WSL2)](https://docs.molt.bot/platforms/windows)
- [Linux 应用](https://docs.molt.bot/platforms/linux)

## Email 钩子（Gmail）

- [docs.molt.bot/gmail-pubsub](https://docs.molt.bot/automation/gmail-pubsub)

## Molty

Moltbot 是为 **Molty** 构建的，一个太空龙虾 AI 助手。🦞
由 Peter Steinberger 和社区制作。

- [clawd.me](https://clawd.me)
- [soul.md](https://soul.md)
- [steipete.me](https://steipete.me)
- [@moltbot](https://x.com/moltbot)

## 社区

参见 [CONTRIBUTING.md](CONTRIBUTING.md) 了解指南、维护者和如何提交 PR。
欢迎 AI/vibe 编码的 PR！🤖

特别感谢 [Mario Zechner](https://mariozechner.at/) 的支持和
[pi-mono](https://github.com/badlogic/pi-mono)。

感谢所有 clawtributors：
