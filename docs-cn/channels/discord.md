---
summary: "Discord 机器人支持状态、功能和配置"
read_when:
  - 处理 Discord 渠道功能
---
# Discord (Bot API)

状态：已准备好通过官方 Discord 机器人网关使用私聊和服务器文本频道。

## 快速设置（初学者）
1) 创建一个 Discord 机器人并复制机器人令牌。
2) 在 Discord 应用设置中，启用 **Message Content Intent**（如果您计划使用白名单或名称查找，请启用 **Server Members Intent**）。
3) 为 Moltbot 设置令牌：
   - 环境变量：`DISCORD_BOT_TOKEN=...`
   - 或配置：`channels.discord.token: "..."`。
   - 如果两者都设置了，配置优先（环境变量回退仅适用于默认账户）。
4) 使用消息权限将机器人邀请到您的服务器（如果您只想要 DM，请创建一个私人服务器）。
5) 启动网关。
6) DM 访问默认为配对；在首次联系时批准配对码。

最小配置：
```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

## 目标
- 通过 Discord DM 或服务器频道与 Moltbot 对话。
- 直接聊天折叠到 agent 的主会话（默认 `agent:main:main`）；服务器频道保持隔离状态为 `agent:<agentId>:discord:channel:<channelId>`（显示名称使用 `discord:<guildSlug>#<channelSlug>`）。
- 群组 DM 默认被忽略；通过 `channels.discord.dm.groupEnabled` 启用，并可选择通过 `channels.discord.dm.groupChannels` 限制。
- 保持路由确定性：回复始终返回到它们到达的频道。

## 工作原理
1. 创建一个 Discord 应用程序 → Bot，启用您所需的 intents（DM + 服务器消息 + 消息内容），并获取机器人令牌。
2. 使用读取/发送消息所需的权限将机器人邀请到您的服务器，以便在您想要使用的地方使用它。
3. 使用 `channels.discord.token`（或 `DISCORD_BOT_TOKEN` 作为回退）配置 Moltbot。
4. 运行网关；当令牌可用（配置优先，环境变量回退）并且 `channels.discord.enabled` 不为 `false` 时，它会自动启动 Discord 渠道。
   - 如果您更喜欢环境变量，请设置 `DISCORD_BOT_TOKEN`（配置块是可选的）。
5. 直接聊天：在投递时使用 `user:<id>`（或 `<@id>` 提及）；所有回合都落在共享的 `main` 会话中。纯数字 ID 是有歧义的，将被拒绝。
6. 服务器频道：使用 `channel:<channelId>` 进行投递。默认情况下需要提及，并且可以按服务器或按频道设置。
7. 直接聊天：通过 `channels.discord.dm.policy`（默认：`"pairing"`）默认安全。未知发送者获得配对码（1 小时后过期）；通过 `moltbot pairing approve discord <code>` 批准。
   - 要保持旧的"对任何人开放"行为：设置 `channels.discord.dm.policy="open"` 和 `channels.discord.dm.allowFrom=["*"]`。
   - 要硬白名单：设置 `channels.discord.dm.policy="allowlist"` 并在 `channels.discord.dm.allowFrom` 中列出发送者。
   - 要忽略所有 DM：设置 `channels.discord.dm.enabled=false` 或 `channels.discord.dm.policy="disabled"`。
8. 群组 DM 默认被忽略；通过 `channels.discord.dm.groupEnabled` 启用，并可选择通过 `channels.discord.dm.groupChannels` 限制。
9. 可选的服务器规则：设置 `channels.discord.guilds`，按服务器 ID（首选）或 slug 键控，具有每个频道的规则。
10. 可选的原生命令：`commands.native` 默认为 `"auto"`（Discord/Telegram 开启，Slack 关闭）。使用 `channels.discord.commands.native: true|false|"auto"` 覆盖；`false` 清除先前注册的命令。文本命令由 `commands.text` 控制，必须作为独立的 `/...` 消息发送。使用 `commands.useAccessGroups: false` 绕过命令的访问组检查。
    - 完整命令列表 + 配置：[斜杠命令](/tools/slash-commands)
11. 可选的服务器上下文历史：设置 `channels.discord.historyLimit`（默认 20，回退到 `messages.groupChat.historyLimit`），在回复提及时将最后 N 条服务器消息作为上下文包含在内。设置 `0` 以禁用。
12. 回应：agent 可以通过 `discord` 工具触发回应（由 `channels.discord.actions.*` 控制）。
    - 回应移除语义：请参阅 [/tools/reactions](/tools/reactions)。
    - `discord` 工具仅在当前频道是 Discord 时公开。
13. 原生命令使用隔离的会话密钥（`agent:<agentId>:discord:slash:<userId>`），而不是共享的 `main` 会话。

注意：名称 → id 解析使用服务器成员搜索，需要 Server Members Intent；如果机器人无法搜索成员，请使用 id 或 `<@id>` 提及。
注意：Slugs 是小写的，空格替换为 `-`。频道名称在没有前导 `#` 的情况下被 slug 化。
注意：服务器上下文 `[from:]` 行包括 `author.tag` + `id`，以便轻松进行 ping 就绪回复。

## 配置写入
默认情况下，Discord 被允许写入由 `/config set|unset` 触发的配置更新（需要 `commands.config: true`）。

禁用：
```json5
{
  channels: { discord: { configWrites: false } }
}
```

## 如何创建您自己的机器人

这是在服务器（服务器）频道（如 `#help`）中运行 Moltbot 的"Discord 开发者门户"设置。

### 1) 创建 Discord 应用 + 机器人用户
1. Discord 开发者门户 → **应用程序** → **新建应用程序**
2. 在您的应用程序中：
   - **Bot** → **Add Bot**
   - 复制 **Bot Token**（这是您放入 `DISCORD_BOT_TOKEN` 的内容）

### 2) 启用 Moltbot 所需的网关 Intents
除非您明确启用，否则 Discord 会阻止"特权 Intents"。

在 **Bot** → **Privileged Gateway Intents** 中，启用：
- **Message Content Intent**（读取大多数服务器中的消息文本所必需；如果没有它，您将看到"Used disallowed intents"或机器人将连接但不会对消息做出反应）
- **Server Members Intent**（推荐；某些成员/用户查找和服务器中的白名单匹配所必需）

您通常**不**需要 **Presence Intent**。

### 3) 生成邀请 URL（OAuth2 URL 生成器）
在您的应用程序中：**OAuth2** → **URL Generator**

**Scopes**
- ✅ `bot`
- ✅ `applications.commands`（原生命令所需）

**Bot Permissions**（最小基线）
- ✅ View Channels
- ✅ Send Messages
- ✅ Read Message History
- ✅ Embed Links
- ✅ Attach Files
- ✅ Add Reactions（可选但推荐）
- ✅ Use External Emojis / Stickers（可选；仅当您想要它们时）

避免 **Administrator**，除非您正在调试并完全信任机器人。

复制生成的 URL，打开它，选择您的服务器，并安装机器人。

### 4) 获取 id（服务器/用户/频道）
Discord 到处都使用数字 id；Moltbot 配置首选 id。

1. Discord（桌面/Web）→ **用户设置** → **高级** → 启用 **开发者模式**
2. 右键单击：
   - 服务器名称 → **Copy Server ID**（服务器 id）
   - 频道（例如，`#help`）→ **Copy Channel ID**
   - 您的用户 → **Copy User ID**

### 5) 配置 Moltbot

#### 令牌
通过环境变量（在服务器上推荐）设置机器人令牌：
- `DISCORD_BOT_TOKEN=...`

或通过配置：

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN"
    }
  }
}
```

多账户支持：使用 `channels.discord.accounts` 和每个账户的令牌以及可选的 `name`。请参阅 [`gateway/configuration`](/gateway/configuration#telegramaccounts--discordaccounts--slackaccounts--signalaccounts--imessageaccounts) 以了解共享模式。

#### 白名单 + 频道路由
示例"单服务器，仅允许我，仅允许 #help"：

```json5
{
  channels: {
    discord: {
      enabled: true,
      dm: { enabled: false },
      guilds: {
        "YOUR_GUILD_ID": {
          users: ["YOUR_USER_ID"],
          requireMention: true,
          channels: {
            help: { allow: true, requireMention: true }
          }
        }
      },
      retry: {
        attempts: 3,
        minDelayMs: 500,
        maxDelayMs: 30000,
        jitter: 0.1
      }
    }
  }
}
```

注意：
- `requireMention: true` 表示机器人仅在被提及时回复（推荐用于共享频道）。
- `agents.list[].groupChat.mentionPatterns`（或 `messages.groupChat.mentionPatterns`）也计入服务器消息的提及。
- 多 agent 覆盖：在 `agents.list[].groupChat.mentionPatterns` 上设置每个 agent 的模式。
- 如果存在 `channels`，默认情况下拒绝未列出的任何频道。
- 使用 `"*"` 频道条目在所有频道上应用默认值；显式频道条目覆盖通配符。
- 线程继承父频道配置（白名单、`requireMention`、技能、提示等），除非您明确添加线程频道 id。
- 默认情况下，机器人创作的消息被忽略；设置 `channels.discord.allowBots=true` 以允许它们（自己的消息仍然被过滤）。
- 警告：如果您允许对其他机器人的回复（`channels.discord.allowBots=true`），请使用 `requireMention`、`channels.discord.guilds.*.channels.<id>.users` 白名单和/或 `AGENTS.md` 和 `SOUL.md` 中的清晰护栏防止机器人到机器人的回复循环。

### 6) 验证它工作
1. 启动网关。
2. 在您的服务器频道中，发送：`@Krill hello`（或您的机器人名称）。
3. 如果没有任何反应：请参阅下面的**故障排除**。

### 故障排除
- 首先：运行 `moltbot doctor` 和 `moltbot channels status --probe`（可操作的警告 + 快速审计）。
- **"Used disallowed intents"**：在开发者门户中启用 **Message Content Intent**（可能还有 **Server Members Intent**），然后重启网关。
- **机器人连接但从未在服务器频道中回复**：
  - 缺少 **Message Content Intent**，或
  - 机器人缺乏频道权限（View/Send/Read History），或
  - 您的配置需要提及，而您没有提及它，或
  - 您的服务器/频道白名单拒绝了频道/用户。
- **`requireMention: false` 但仍然没有回复**：
- `channels.discord.groupPolicy` 默认为 **allowlist**；将其设置为 `"open"` 或在 `channels.discord.guilds` 下添加服务器条目（可以选择在 `channels.discord.guilds.<id>.channels` 下列出频道以进行限制）。
  - 如果您仅设置 `DISCORD_BOT_TOKEN` 并且从未创建 `channels.discord` 部分，运行时
    将 `groupPolicy` 默认为 `open`。添加 `channels.discord.groupPolicy`、
    `channels.defaults.groupPolicy` 或服务器/频道白名单以锁定它。
- `requireMention` 必须位于 `channels.discord.guilds`（或特定频道）下。顶层的 `channels.discord.requireMention` 被忽略。
- **权限审计**（`channels status --probe`）仅检查数字频道 ID。如果您使用 slugs/名称作为 `channels.discord.guilds.*.channels` 键，审计无法验证权限。
- **DM 不工作**：`channels.discord.dm.enabled=false`、`channels.discord.dm.policy="disabled"`，或者您尚未被批准（`channels.discord.dm.policy="pairing"`）。

## 功能和限制
- DM 和服务器文本频道（线程被视为单独的频道；不支持语音）。
- 尽力发送输入指示器；消息分块使用 `channels.discord.textChunkLimit`（默认 2000）并按行数拆分长回复（`channels.discord.maxLinesPerMessage`，默认 17）。
- 可选的换行符分块：设置 `channels.discord.chunkMode="newline"` 在长度分块之前在空行（段落边界）处拆分。
- 支持高达配置的 `channels.discord.mediaMaxMb`（默认 8 MB）的文件上传。
- 默认情况下，提及门控的服务器回复以避免嘈杂的机器人。
- 当消息引用另一条消息时（引用的内容 + id），会注入回复上下文。
- 原生回复线程**默认关闭**；通过 `channels.discord.replyToMode` 和回复标签启用。

## 重试策略
出站 Discord API 调用使用 Discord 的 `retry_after`（如果可用）对速率限制（429）进行重试，具有指数退避和抖动。通过 `channels.discord.retry` 配置。请参阅 [重试策略](/concepts/retry)。

## 配置

```json5
{
  channels: {
    discord: {
      enabled: true,
      token: "abc.123",
      groupPolicy: "allowlist",
      guilds: {
        "*": {
          channels: {
            general: { allow: true }
          }
        }
      },
      mediaMaxMb: 8,
      actions: {
        reactions: true,
        stickers: true,
        emojiUploads: true,
        stickerUploads: true,
        polls: true,
        permissions: true,
        messages: true,
        threads: true,
        pins: true,
        search: true,
        memberInfo: true,
        roleInfo: true,
        roles: false,
        channelInfo: true,
        channels: true,
        voiceStatus: true,
        events: true,
        moderation: false
      },
      replyToMode: "off",
      dm: {
        enabled: true,
        policy: "pairing", // pairing | allowlist | open | disabled
        allowFrom: ["123456789012345678", "steipete"],
        groupEnabled: false,
        groupChannels: ["clawd-dm"]
      },
      guilds: {
        "*": { requireMention: true },
        "123456789012345678": {
          slug: "friends-of-clawd",
          requireMention: false,
          reactionNotifications: "own",
          users: ["987654321098765432", "steipete"],
          channels: {
            general: { allow: true },
            help: {
              allow: true,
              requireMention: true,
              users: ["987654321098765432"],
              skills: ["search", "docs"],
              systemPrompt: "Keep answers short."
            }
          }
        }
      }
    }
  }
}
```

确认反应通过 `messages.ackReaction` +
`messages.ackReactionScope` 全局控制。使用 `messages.removeAckAfterReply` 在机器人回复后清除
确认反应。

- `dm.enabled`：设置 `false` 忽略所有 DM（默认 `true`）。
- `dm.policy`：DM 访问控制（推荐 `pairing`）。`"open"` 需要 `dm.allowFrom=["*"]`。
- `dm.allowFrom`：DM 白名单（用户 id 或名称）。由 `dm.policy="allowlist"` 和用于 `dm.policy="open"` 验证。向导接受用户名并在机器人可以搜索成员时将其解析为 id。
- `dm.groupEnabled`：启用群组 DM（默认 `false`）。
- `dm.groupChannels`：群组 DM 频道 id 或 slug 的可选白名单。
- `groupPolicy`：控制服务器频道处理（`open|disabled|allowlist`）；`allowlist` 需要频道白名单。
- `guilds`：按服务器 ID（首选）或 slug 键控的每个服务器规则。
- `guilds."*"`：当不存在显式条目时应用的默认每个服务器设置。
- `guilds.<id>.slug`：用于显示名称的可选友好 slug。
- `guilds.<id>.users`：可选的每个服务器用户白名单（id 或名称）。
- `guilds.<id>.tools`：可选的每个服务器工具策略覆盖（`allow`/`deny`/`alsoAllow`），当缺少频道覆盖时使用。
- `guilds.<id>.toolsBySender`：可选的每个发送者工具策略覆盖，在服务器级别（当缺少频道覆盖时应用；支持 `"*"` 通配符）。
- `guilds.<id>.channels.<channel>.allow`：当 `groupPolicy="allowlist"` 时允许/拒绝频道。
- `guilds.<id>.channels.<channel>.requireMention`：频道的提及门控。
- `guilds.<id>.channels.<channel>.tools`：可选的每个频道工具策略覆盖（`allow`/`deny`/`alsoAllow`）。
- `guilds.<id>.channels.<channel>.toolsBySender`：可选的每个频道内每个发送者工具策略覆盖（支持 `"*"` 通配符）。
- `guilds.<id>.channels.<channel>.users`：可选的每个频道用户白名单。
- `guilds.<id>.channels.<channel>.skills`：技能过滤器（省略 = 所有技能，空 = 无）。
- `guilds.<id>.channels.<channel>.systemPrompt`：频道的额外系统提示（与频道主题结合）。
- `guilds.<id>.channels.<channel>.enabled`：设置 `false` 以禁用频道。
- `guilds.<id>.channels`：频道规则（键是频道 slugs 或 id）。
- `guilds.<id>.requireMention`：每个服务器的提及要求（每个频道可覆盖）。
- `guilds.<id>.reactionNotifications`：反应系统事件模式（`off`、`own`、`all`、`allowlist`）。
- `textChunkLimit`：出站文本分块大小（字符）。默认：2000。
- `chunkMode`：`length`（默认）仅在超过 `textChunkLimit` 时拆分；`newline` 在长度分块之前在空行（段落边界）处拆分。
- `maxLinesPerMessage`：每条消息的软最大行数。默认：17。
- `mediaMaxMb`：限制保存到磁盘的入站媒体。
- `historyLimit`：在回复提及时要包括的最近服务器消息的数量（默认 20；回退到 `messages.groupChat.historyLimit`；`0` 禁用）。
- `dmHistoryLimit`：以用户回合为单位的 DM 历史限制。每个用户覆盖：`dms["<user_id>"].historyLimit`。
- `retry`：出站 Discord API 调用的重试策略（attempts、minDelayMs、maxDelayMs、jitter）。
- `actions`：每个操作的工具门；省略以允许所有（设置 `false` 以禁用）。
  - `reactions`（涵盖 react + read reactions）
  - `stickers`、`emojiUploads`、`stickerUploads`、`polls`、`permissions`、`messages`、`threads`、`pins`、`search`
  - `memberInfo`、`roleInfo`、`channelInfo`、`voiceStatus`、`events`
  - `channels`（创建/编辑/删除频道 + 类别 + 权限）
  - `roles`（角色添加/移除，默认 `false`）
  - `moderation`（超时/踢出/封禁，默认 `false`）

反应通知使用 `guilds.<id>.reactionNotifications`：
- `off`：没有反应事件。
- `own`：机器人自己消息上的反应（默认）。
- `all`：所有消息上的所有反应。
- `allowlist`：来自 `guilds.<id>.users` 的所有消息上的反应（空列表禁用）。

### 工具操作默认值

| 操作组 | 默认 | 注意 |
| --- | --- | --- |
| reactions | 已启用 | React + list reactions + emojiList |
| stickers | 已启用 | 发送贴纸 |
| emojiUploads | 已启用 | 上传表情符号 |
| stickerUploads | 已启用 | 上传贴纸 |
| polls | 已启用 | 创建投票 |
| permissions | 已启用 | 频道权限快照 |
| messages | 已启用 | 读取/发送/编辑/删除 |
| threads | 已启用 | 创建/列表/回复 |
| pins | 已启用 | 固定/取消固定/列表 |
| search | 已启用 | 消息搜索（预览功能） |
| memberInfo | 已启用 | 成员信息 |
| roleInfo | 已启用 | 角色列表 |
| channelInfo | 已启用 | 频道信息 + 列表 |
| channels | 已启用 | 频道/类别管理 |
| voiceStatus | 已启用 | 语音状态查找 |
| events | 已启用 | 列出/创建计划的事件 |
| roles | 已禁用 | 角色添加/移除 |
| moderation | 已禁用 | 超时/踢出/封禁 |
- `replyToMode`：`off`（默认）、`first` 或 `all`。仅当模型包括回复标签时应用。

## 回复标签
要请求线程回复，模型可以在其输出中包括一个标签：
- `[[reply_to_current]]` — 回复触发 Discord 消息。
- `[[reply_to:<id>]]` — 回复来自上下文/历史的特定消息 id。
当前消息 id 作为 `[message_id: …]` 附加到提示；历史条目已经包括 id。

行为由 `channels.discord.replyToMode` 控制：
- `off`：忽略标签。
- `first`：只有第一个出站块/附件是回复。
- `all`：每个出站块/附件都是回复。

白名单匹配注意事项：
- `allowFrom`/`users`/`groupChannels` 接受 id、名称、标签或提及，如 `<@id>`。
- 支持前缀，如 `discord:`/`user:`（用户）和 `channel:`（群组 DM）。
- 使用 `*` 允许任何发送者/频道。
- 当存在 `guilds.<id>.channels` 时，默认情况下拒绝未列出的频道。
- 当省略 `guilds.<id>.channels` 时，白名单服务器中的所有频道都被允许。
- 要允许**没有频道**，设置 `channels.discord.groupPolicy: "disabled"`（或保持白名单为空）。
- 配置向导接受 `Guild/Channel` 名称（公共 + 私有）并在可能时将它们解析为 ID。
- 启动时，Moltbot 将白名单中的频道/用户名称解析为 ID（当机器人可以搜索成员时）
  并记录映射；未解析的条目保持原样输入。

原生命令注意事项：
- 注册的命令镜像 Moltbot 的聊天命令。
- 原生命令遵循与 DM/服务器消息相同的白名单（`channels.discord.dm.allowFrom`、`channels.discord.guilds`、每个频道规则）。
- 斜杠命令对于未被白名单的用户在 Discord UI 中仍然可见；Moltbot 在执行时强制执行白名单并回复"未授权"。

## 工具操作
agent 可以使用以下操作调用 `discord`：
- `react` / `reactions`（添加或列出反应）
- `sticker`、`poll`、`permissions`
- `readMessages`、`sendMessage`、`editMessage`、`deleteMessage`
- 读取/搜索/固定工具负载包括原始 Discord `timestamp` 旁边的规范化 `timestampMs`（UTC 纪元毫秒）和 `timestampUtc`。
- `threadCreate`、`threadList`、`threadReply`
- `pinMessage`、`unpinMessage`、`listPins`
- `searchMessages`、`memberInfo`、`roleInfo`、`roleAdd`、`roleRemove`、`emojiList`
- `channelInfo`、`channelList`、`voiceStatus`、`eventList`、`eventCreate`
- `timeout`、`kick`、`ban`

Discord 消息 id 在注入的上下文（`[discord message id: …]` 和历史行）中显示，以便 agent 可以定位它们。
表情符号可以是 unicode（例如，`✅`）或自定义表情符号语法，如 `<:party_blob:1234567890>`。

## 安全性和运维
- 将机器人令牌视为密码；在受监督的主机上首选 `DISCORD_BOT_TOKEN` 环境变量或锁定配置文件权限。
- 仅授予机器人所需的权限（通常是读取/发送消息）。
- 如果机器人被卡住或速率受限，在确认没有其他进程拥有 Discord 会话后重启网关（`moltbot gateway --force`）。
