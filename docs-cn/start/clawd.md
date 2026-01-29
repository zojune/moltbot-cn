---
summary: "使用 Moltbot 构建个人助手的端到端指南（含安全注意事项）"
read_when:
  - 入门新的助手实例
  - 审查安全/权限影响
---
# 使用 Moltbot 构建个人助手（Clawd 风格）

Moltbot 是一个面向 **Pi** 代理的 WhatsApp + Telegram + Discord + iMessage 网关。插件添加 Mattermost。本指南是"个人助手"设置：一个专用的 WhatsApp 号码，表现得像你始终在线的代理。

## ⚠️ 安全第一

你将代理置于可以执行以下操作的位置：
- 在你的机器上运行命令（取决于你的 Pi 工具设置）
- 读取/写入工作区中的文件
- 通过 WhatsApp/Telegram/Discord/Mattermost（插件）发回消息

保守开始：
- 始终设置 `channels.whatsapp.allowFrom`（永远不要在个人 Mac 上向世界开放）。
- 为助手使用专用的 WhatsApp 号码。
- 心跳现在默认为每 30 分钟。在你信任设置之前，通过设置 `agents.defaults.heartbeat.every: "0m"` 来禁用。

## 前提条件

- Node **22+**
- Moltbot 在 PATH 上可用（推荐：全局安装）
- 助手的第二个电话号码（SIM/eSIM/预付费）

```bash
npm install -g moltbot@latest
# 或：pnpm add -g moltbot@latest
```

从源代码（开发）：

```bash
git clone https://github.com/moltbot/moltbot.git
cd moltbot
pnpm install
pnpm ui:build # 首次运行时自动安装 UI 依赖
pnpm build
pnpm link --global
```

## 双手机设置（推荐）

你想要这样：

```
你的手机（个人）          第二部手机（助手）
┌─────────────────┐           ┌─────────────────┐
│  你的 WhatsApp  │  ──────▶  │  助手 WA        │
│  +1-555-YOU     │  消息     │  +1-555-CLAWD   │
└─────────────────┘           └────────┬────────┘
                                       │ 通过 QR 链接
                                       ▼
                              ┌─────────────────┐
                              │  你的 Mac       │
                              │  (moltbot)      │
                              │    Pi 代理      │
                              └─────────────────┘
```

如果你将个人 WhatsApp 链接到 Moltbot，每条发给你的消息都会变成"代理输入"。这很少是你想要的。

## 5 分钟快速开始

1) 配对 WhatsApp Web（显示 QR；使用助手手机扫描）：

```bash
moltbot channels login
```

2) 启动网关（保持其运行）：

```bash
moltbot gateway --port 18789
```

3) 在 `~/.clawdbot/moltbot.json` 中放入最小配置：

```json5
{
  channels: { whatsapp: { allowFrom: ["+15555550123"] } }
}
```

现在从你的允许列表手机向助手号码发送消息。

当入门完成时，我们会自动打开带有网关令牌的仪表板并打印令牌化链接。以后重新打开：`moltbot dashboard`。

## 为代理提供一个工作区（AGENTS）

Clawd 从其工作区目录读取操作说明和"记忆"。

默认情况下，Moltbot 使用 `~/clawd` 作为代理工作区，并会在设置/首次代理运行时自动创建它（加上入门 `AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`）。`BOOTSTRAP.md` 仅在工作区全新时创建（删除后不应恢复）。

提示：将此文件夹视为 Clawd 的"记忆"，并将其设为 git 仓库（理想情况下是私有的），以便你的 `AGENTS.md` + 记忆文件得到备份。如果安装了 git，全新的工作区会自动初始化。

```bash
moltbot setup
```

完整的工作区布局 + 备份指南：[代理工作区](/concepts/agent-workspace)
记忆工作流：[记忆](/concepts/memory)

可选：使用 `agents.defaults.workspace` 选择不同的工作区（支持 `~`）。

```json5
{
  agent: {
    workspace: "~/clawd"
  }
}
```

如果你已经从仓库提供自己的工作区文件，可以完全禁用引导文件创建：

```json5
{
  agent: {
    skipBootstrap: true
  }
}
```

## 将其变成"助手"的配置

Moltbot 默认为良好的助手设置，但你通常需要调整：
- `SOUL.md` 中的角色/说明
- 思考默认值（如果需要）
- 心跳（一旦你信任它）

示例：

```json5
{
  logging: { level: "info" },
  agent: {
    model: "anthropic/claude-opus-4-5",
    workspace: "~/clawd",
    thinkingDefault: "high",
    timeoutSeconds: 1800,
    // 从 0 开始；以后启用。
    heartbeat: { every: "0m" }
  },
  channels: {
    whatsapp: {
      allowFrom: ["+15555550123"],
      groups: {
        "*": { requireMention: true }
      }
    }
  },
  routing: {
    groupChat: {
      mentionPatterns: ["@clawd", "clawd"]
    }
  },
  session: {
    scope: "per-sender",
    resetTriggers: ["/new", "/reset"],
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 10080
    }
  }
}
```

## 会话和记忆

- 会话文件：`~/.clawdbot/agents/<agentId>/sessions/{{SessionId}}.jsonl`
- 会话元数据（令牌使用、最后路由等）：`~/.clawdbot/agents/<agentId>/sessions/sessions.json`（旧版：`~/.clawdbot/sessions/sessions.json`）
- `/new` 或 `/reset` 为该聊天开始一个新会话（可通过 `resetTriggers` 配置）。如果单独发送，代理会回复一个简短的问候以确认重置。
- `/compact [instructions]` 压缩会话上下文并报告剩余的上下文预算。

## 心跳（主动模式）

默认情况下，Moltbot 每 30 分钟运行一次心跳，提示为：
`如果存在则读取 HEARTBEAT.md（工作区上下文）。严格遵循它。不要推断或重复先前聊天的旧任务。如果不需要注意，回复 HEARTBEAT_OK。`
设置 `agents.defaults.heartbeat.every: "0m"` 以禁用。

- 如果 `HEARTBEAT.md` 存在但实际上为空（只有空行和像 `# Heading` 这样的 markdown 标题），Moltbot 会跳过心跳运行以保存 API 调用。
- 如果文件丢失，心跳仍会运行，模型决定做什么。
- 如果代理回复 `HEARTBEAT_OK`（可选带有简短填充；见 `agents.defaults.heartbeat.ackMaxChars`），Moltbot 会抑制该心跳的出站传递。
- 心跳运行完整的代理轮次 — 较短的间隔会消耗更多令牌。

```json5
{
  agent: {
    heartbeat: { every: "30m" }
  }
}
```

## 媒体进出

入站附件（图像/音频/文档）可以通过模板传递给你的命令：
- `{{MediaPath}}`（本地临时文件路径）
- `{{MediaUrl}}`（伪 URL）
- `{{Transcript}}`（如果启用了音频转录）

来自代理的出站附件：在单独一行中包含 `MEDIA:<path-or-url>`（无空格）。示例：

```
这是截图。
MEDIA:/tmp/screenshot.png
```

Moltbot 提取这些并将它们作为媒体与文本一起发送。

## 运维检查清单

```bash
moltbot status          # 本地状态（凭据、会话、排队事件）
moltbot status --all    # 完整诊断（只读、可粘贴）
moltbot status --deep   # 添加网关健康探测（Telegram + Discord）
moltbot health --json   # 网关健康快照（WS）
```

日志位于 `/tmp/moltbot/` 下（默认：`moltbot-YYYY-MM-DD.log`）。

## 后续步骤

- WebChat：[WebChat](/web/webchat)
- 网关运维：[网关运行手册](/gateway)
- Cron + 唤醒：[Cron 作业](/automation/cron-jobs)
- macOS 菜单栏伴侣：[Moltbot macOS 应用](/platforms/macos)
- iOS 节点应用：[iOS 应用](/platforms/ios)
- Android 节点应用：[Android 应用](/platforms/android)
- Windows 状态：[Windows (WSL2)](/platforms/windows)
- Linux 状态：[Linux 应用](/platforms/linux)
- 安全：[安全](/gateway/security)
