---
summary: "Nextcloud Talk 支持状态、功能和配置"
read_when:
  - 开发 Nextcloud Talk 频道功能时
---
# Nextcloud Talk (插件)

状态: 通过插件支持(webhook bot)。支持私信、房间、表情回应和 markdown 消息。

## 需要插件
Nextcloud Talk 作为插件提供，不包含在核心安装中。

通过 CLI 安装(npm 注册表):
```bash
moltbot plugins install @moltbot/nextcloud-talk
```

本地检出(从 git 仓库运行时):
```bash
moltbot plugins install ./extensions/nextcloud-talk
```

如果您在配置/入职过程中选择了 Nextcloud Talk，并且检测到 git 检出，Moltbot 将自动提供本地安装路径。

详情: [插件](/plugin)

## 快速设置(初学者)
1) 安装 Nextcloud Talk 插件。
2) 在您的 Nextcloud 服务器上，创建一个 bot:
   ```bash
   ./occ talk:bot:install "Moltbot" "<shared-secret>" "<webhook-url>" --feature reaction
   ```
3) 在目标房间设置中启用 bot。
4) 配置 Moltbot:
   - 配置: `channels.nextcloud-talk.baseUrl` + `channels.nextcloud-talk.botSecret`
   - 或环境: `NEXTCLOUD_TALK_BOT_SECRET`(仅默认账户)
5) 重启网关(或完成入职)。

最小配置:
```json5
{
  channels: {
    "nextcloud-talk": {
      enabled: true,
      baseUrl: "https://cloud.example.com",
      botSecret: "shared-secret",
      dmPolicy: "pairing"
    }
  }
}
```

## 注意事项
- Bot 无法发起私信。用户必须先向 bot 发送消息。
- Webhook URL 必须可以被网关访问；如果在代理后面，设置 `webhookPublicUrl`。
- Bot API 不支持媒体上传；媒体作为 URL 发送。
- Webhook 负载不区分私信与房间；设置 `apiUser` + `apiPassword` 以启用房间类型查找(否则私信被视为房间)。

## 访问控制(私信)
- 默认: `channels.nextcloud-talk.dmPolicy = "pairing"`。未知发送者会收到配对代码。
- 通过以下方式批准:
  - `moltbot pairing list nextcloud-talk`
  - `moltbot pairing approve nextcloud-talk <CODE>`
- 公开私信: `channels.nextcloud-talk.dmPolicy="open"` 加上 `channels.nextcloud-talk.allowFrom=["*"]`。

## 房间(群组)
- 默认: `channels.nextcloud-talk.groupPolicy = "allowlist"`(提及限制)。
- 使用 `channels.nextcloud-talk.rooms` 允许列表房间:
```json5
{
  channels: {
    "nextcloud-talk": {
      rooms: {
        "room-token": { requireMention: true }
      }
    }
  }
}
```
- 要不允许房间，保持允许列表为空或设置 `channels.nextcloud-talk.groupPolicy="disabled"`。

## 功能
| 功能 | 状态 |
|---------|--------|
| 私信 | 支持 |
| 房间 | 支持 |
| 线程 | 不支持 |
| 媒体 | 仅 URL |
| 表情回应 | 支持 |
| 原生命令 | 不支持 |

## 配置参考(Nextcloud Talk)
完整配置: [配置](/gateway/configuration)

提供程序选项:
- `channels.nextcloud-talk.enabled`: 启用/禁用频道启动。
- `channels.nextcloud-talk.baseUrl`: Nextcloud 实例 URL。
- `channels.nextcloud-talk.botSecret`: bot 共享密钥。
- `channels.nextcloud-talk.botSecretFile`: 密钥文件路径。
- `channels.nextcloud-talk.apiUser`: 用于房间查找的 API 用户(私信检测)。
- `channels.nextcloud-talk.apiPassword`: 用于房间查找的 API/应用密码。
- `channels.nextcloud-talk.apiPasswordFile`: API 密码文件路径。
- `channels.nextcloud-talk.webhookPort`: webhook 监听器端口(默认 8788)。
- `channels.nextcloud-talk.webhookHost`: webhook 主机(默认 0.0.0.0)。
- `channels.nextcloud-talk.webhookPath`: webhook 路径(默认 /nextcloud-talk-webhook)。
- `channels.nextcloud-talk.webhookPublicUrl`: 外部可访问的 webhook URL。
- `channels.nextcloud-talk.dmPolicy`: `pairing | allowlist | open | disabled`。
- `channels.nextcloud-talk.allowFrom`: 私信允许列表(用户 ID)。`open` 需要 `"*"`。
- `channels.nextcloud-talk.groupPolicy`: `allowlist | open | disabled`。
- `channels.nextcloud-talk.groupAllowFrom`: 群组允许列表(用户 ID)。
- `channels.nextcloud-talk.rooms`: 每房间设置和允许列表。
- `channels.nextcloud-talk.historyLimit`: 群组历史记录限制(0 禁用)。
- `channels.nextcloud-talk.dmHistoryLimit`: 私信历史记录限制(0 禁用)。
- `channels.nextcloud-talk.dms`: 每私信覆盖(historyLimit)。
- `channels.nextcloud-talk.textChunkLimit`: 出站文本块大小(字符)。
- `channels.nextcloud-talk.chunkMode`: `length`(默认)或 `newline` 在长度分块之前按空行(段落边界)分割。
- `channels.nextcloud-talk.blockStreaming`: 为此频道禁用块流式传输。
- `channels.nextcloud-talk.blockStreamingCoalesce`: 块流式传输合并调整。
- `channels.nextcloud-talk.mediaMaxMb`: 入站媒体上限(MB)。
