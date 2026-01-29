---
summary: "Microsoft Teams 机器人支持状态、功能和配置"
read_when:
  - 开发 MS Teams 频道功能时
---
# Microsoft Teams (插件)

> "放弃所有希望，进入这里的人。"

更新时间: 2026-01-21

状态: 支持文本 + 私信附件；频道/群组文件发送需要 `sharePointSiteId` + Graph 权限(参见 [在群组聊天中发送文件](#在群组聊天中发送文件))。投票通过 Adaptive Cards 发送。

## 需要插件
Microsoft Teams 作为插件提供，不包含在核心安装中。

**重大变更(2026.1.15):** MS Teams 从核心移出。如果您使用它，必须安装插件。

可解释的原因: 保持核心安装更轻量，并让 MS Teams 依赖项独立更新。

通过 CLI 安装(npm 注册表):
```bash
moltbot plugins install @moltbot/msteams
```

本地检出(从 git 仓库运行时):
```bash
moltbot plugins install ./extensions/msteams
```

如果您在配置/入职过程中选择了 Teams，并且检测到 git 检出，Moltbot 将自动提供本地安装路径。

详情: [插件](/plugin)

## 快速设置(初学者)
1) 安装 Microsoft Teams 插件。
2) 创建 **Azure Bot**(App ID + 客户端密钥 + 租户 ID)。
3) 使用这些凭据配置 Moltbot。
4) 通过公共 URL 或隧道公开 `/api/messages`(默认端口 3978)。
5) 安装 Teams 应用包并启动网关。

最小配置:
```json5
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" }
    }
  }
}
```
注意: 默认阻止群组聊天(`channels.msteams.groupPolicy: "allowlist"`)。要允许群组回复，设置 `channels.msteams.groupAllowFrom`(或使用 `groupPolicy: "open"` 允许任何成员，提及限制)。

## 目标
- 通过 Teams 私信、群组聊天或频道与 Moltbot 对话。
- 保持路由确定性: 回复总是返回到它们到达的频道。
- 默认安全频道行为(除非另有配置，否则需要提及)。

## 配置写入
默认情况下，允许 Microsoft Teams 写入由 `/config set|unset` 触发的配置更新(需要 `commands.config: true`)。

禁用:
```json5
{
  channels: { msteams: { configWrites: false } }
}
```

## 访问控制(私信 + 群组)

**私信访问**
- 默认: `channels.msteams.dmPolicy = "pairing"`。未知发送者将被忽略，直到被批准。
- `channels.msteams.allowFrom` 接受 AAD 对象 ID、UPN 或显示名称。当凭据允许时，向导通过 Microsoft Graph 将名称解析为 ID。

**群组访问**
- 默认: `channels.msteams.groupPolicy = "allowlist"`(除非您添加 `groupAllowFrom`，否则被阻止)。使用 `channels.defaults.groupPolicy` 在未设置时覆盖默认值。
- `channels.msteams.groupAllowFrom` 控制哪些发送者可以在群组聊天/频道中触发(回退到 `channels.msteams.allowFrom`)。
- 设置 `groupPolicy: "open"` 允许任何成员(默认仍提及限制)。
- 要允许**无频道**，设置 `channels.msteams.groupPolicy: "disabled"`。

示例:
```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"]
    }
  }
}
```

**Teams + 频道允许列表**
- 通过在 `channels.msteams.teams` 下列出团队和频道来限制群组/频道回复。
- 键可以是团队 ID 或名称；频道键可以是对话 ID 或名称。
- 当 `groupPolicy="allowlist"` 并且存在 teams 允许列表时，仅接受列出的团队/频道(提及限制)。
- 配置向导接受 `Team/Channel` 条目并为您存储它们。
- 启动时，Moltbot 将团队/频道和用户允许列表名称解析为 ID(当 Graph 权限允许时)并记录映射；未解析的条目保持输入状态。

示例:
```json5
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            "General": { requireMention: true }
          }
        }
      }
    }
  }
}
```

## 工作原理
1. 安装 Microsoft Teams 插件。
2. 创建 **Azure Bot**(App ID + 密钥 + 租户 ID)。
3. 构建一个引用机器人并包含以下 RSC 权限的 **Teams 应用包**。
4. 将 Teams 应用上传/安装到团队(或用于私信的个人范围)。
5. 在 `~/.clawdbot/moltbot.json` 中配置 `msteams`(或环境变量)并启动网关。
6. 网关默认在 `/api/messages` 上监听 Bot Framework webhook 流量。

## Azure Bot 设置(先决条件)

在配置 Moltbot 之前，您需要创建一个 Azure Bot 资源。

### 步骤 1: 创建 Azure Bot

1. 访问 [创建 Azure Bot](https://portal.azure.com/#create/Microsoft.AzureBot)
2. 填写**基本信息**选项卡:

   | 字段 | 值 |
   |-------|-------|
   | **Bot handle** | 您的机器人名称，例如 `moltbot-msteams`(必须唯一) |
   | **订阅** | 选择您的 Azure 订阅 |
   | **资源组** | 创建新的或使用现有的 |
   | **定价层** | **免费**用于开发/测试 |
   | **应用类型** | **单租户**(推荐 - 见下文说明) |
   | **创建类型** | **创建新的 Microsoft App ID** |

> **弃用通知:** 2025-07-31 之后，不再支持创建新的多租户机器人。新机器人请使用**单租户**。

3. 点击**查看 + 创建** → **创建**(等待约 1-2 分钟)

### 步骤 2: 获取凭据

1. 转到您的 Azure Bot 资源 → **配置**
2. 复制 **Microsoft App ID** → 这是您的 `appId`
3. 点击**管理密码** → 转到应用注册
4. 在**证书和密钥**下 → **新客户端密钥** → 复制**值** → 这是您的 `appPassword`
5. 转到**概述** → 复制**目录(租户) ID** → 这是您的 `tenantId`

### 步骤 3: 配置消息传递端点

1. 在 Azure Bot → **配置**中
2. 将**消息传递端点**设置为您的 webhook URL:
   - 生产环境: `https://your-domain.com/api/messages`
   - 本地开发: 使用隧道(参见下面的[本地开发](#本地开发隧道))

### 步骤 4: 启用 Teams 频道

1. 在 Azure Bot → **频道**中
2. 点击 **Microsoft Teams** → 配置 → 保存
3. 接受服务条款

## 本地开发(隧道)

Teams 无法访问 `localhost`。使用隧道进行本地开发:

**选项 A: ngrok**
```bash
ngrok http 3978
# 复制 https URL，例如 https://abc123.ngrok.io
# 将消息传递端点设置为: https://abc123.ngrok.io/api/messages
```

**选项 B: Tailscale Funnel**
```bash
tailscale funnel 3978
# 使用您的 Tailscale funnel URL 作为消息传递端点
```

## Teams 开发者门户(替代方案)

您可以使用 [Teams 开发者门户](https://dev.teams.microsoft.com/apps)代替手动创建清单 ZIP:

1. 点击 **+ 新建应用**
2. 填写基本信息(名称、描述、开发者信息)
3. 转到**应用功能** → **Bot**
4. 选择**手动输入 bot ID** 并粘贴您的 Azure Bot App ID
5. 检查范围: **个人**、**团队**、**群组聊天**
6. 点击**分发** → **下载应用包**
7. 在 Teams 中: **应用** → **管理您的应用** → **上传自定义应用** → 选择 ZIP

这通常比手动编辑 JSON 清单更容易。

## 测试机器人

**选项 A: Azure Web 聊天(先验证 webhook)**
1. 在 Azure 门户中 → 您的 Azure Bot 资源 → **在 Web 聊天中测试**
2. 发送消息 - 您应该看到响应
3. 这确认您的 webhook 端点在 Teams 设置之前有效

**选项 B: Teams(应用安装后)**
1. 安装 Teams 应用(侧载或组织目录)
2. 在 Teams 中找到机器人并发送私信
3. 检查网关日志中的传入活动

## 设置(最小仅文本)
1. **安装 Microsoft Teams 插件**
   - 从 npm 安装: `moltbot plugins install @moltbot/msteams`
   - 从本地检出安装: `moltbot plugins install ./extensions/msteams`

2. **机器人注册**
   - 创建 Azure Bot(见上文)并记录:
     - App ID
     - 客户端密钥(App 密码)
     - 租户 ID(单租户)

3. **Teams 应用清单**
   - 包含一个 `bot` 条目，其中 `botId = <App ID>`。
   - 范围: `personal`、`team`、`groupChat`。
   - `supportsFiles: true`(个人范围文件处理所必需)。
   - 添加 RSC 权限(下文)。
   - 创建图标: `outline.png`(32x32) 和 `color.png`(192x192)。
   - 将所有三个文件打包在一起: `manifest.json`、`outline.png`、`color.png`。

4. **配置 Moltbot**
   ```json
   {
     "msteams": {
       "enabled": true,
       "appId": "<APP_ID>",
       "appPassword": "<APP_PASSWORD>",
       "tenantId": "<TENANT_ID>",
       "webhook": { "port": 3978, "path": "/api/messages" }
     }
   }
   ```

   您也可以使用环境变量代替配置键:
   - `MSTEAMS_APP_ID`
   - `MSTEAMS_APP_PASSWORD`
   - `MSTEAMS_TENANT_ID`

5. **机器人端点**
   - 将 Azure Bot 消息传递端点设置为:
     - `https://<host>:3978/api/messages`(或您选择的路径/端口)。

6. **运行网关**
   - 当插件安装并且存在带有凭据的 `msteams` 配置时，Teams 频道自动启动。

## 历史记录上下文
- `channels.msteams.historyLimit` 控制将多少最近的频道/群组消息包装到提示中。
- 回退到 `messages.groupChat.historyLimit`。设置 `0` 禁用(默认 50)。
- 私信历史记录可以通过 `channels.msteams.dmHistoryLimit` 限制(用户轮次)。每用户覆盖: `channels.msteams.dms["<user_id>"].historyLimit`。

## 当前 Teams RSC 权限(清单)
这些是 Teams 应用清单中的**现有 resourceSpecific 权限**。它们仅适用于安装应用的团队/聊天内。

**用于频道(团队范围):**
- `ChannelMessage.Read.Group`(应用程序) - 无需 @提及即可接收所有频道消息
- `ChannelMessage.Send.Group`(应用程序)
- `Member.Read.Group`(应用程序)
- `Owner.Read.Group`(应用程序)
- `ChannelSettings.Read.Group`(应用程序)
- `TeamMember.Read.Group`(应用程序)
- `TeamSettings.Read.Group`(应用程序)

**用于群组聊天:**
- `ChatMessage.Read.Chat`(应用程序) - 无需 @提及即可接收所有群组聊天消息

## 示例 Teams 清单(已编辑)
包含必填字段的最小有效示例。替换 ID 和 URL。

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "Moltbot" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "Moltbot in Teams", "full": "Moltbot in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### 清单注意事项(必填字段)
- `bots[].botId` **必须**匹配 Azure Bot App ID。
- `webApplicationInfo.id` **必须**匹配 Azure Bot App ID。
- `bots[].scopes` 必须包含您计划使用的表面(`personal`、`team`、`groupChat`)。
- `bots[].supportsFiles: true` 是个人范围内文件处理所必需的。
- `authorization.permissions.resourceSpecific` 必须包含频道读/写，如果您希望频道流量。

### 更新现有应用

要更新已安装的 Teams 应用(例如，添加 RSC 权限):

1. 使用新设置更新您的 `manifest.json`
2. **增加 `version` 字段**(例如，`1.0.0` → `1.1.0`)
3. **重新打包**清单和图标(`manifest.json`、`outline.png`、`color.png`)
4. 上传新的 zip:
   - **选项 A(Teams 管理中心):** Teams 管理中心 → Teams 应用 → 管理应用 → 找到您的应用 → 上传新版本
   - **选项 B(侧载):** 在 Teams 中 → 应用 → 管理您的应用 → 上传自定义应用
5. **对于团队频道:** 在每个团队中重新安装应用以使新权限生效
6. **完全退出并重新启动 Teams**(不仅是关闭窗口)以清除缓存的应用元数据

## 功能: 仅 RSC vs Graph

### **仅 Teams RSC**(应用已安装，无 Graph API 权限)
有效:
- 读取频道消息**文本**内容。
- 发送频道消息**文本**内容。
- 接收**个人(私信)**文件附件。

无效:
- 频道/群组**图像或文件内容**(负载仅包含 HTML 存根)。
- 下载存储在 SharePoint/OneDrive 中的附件。
- 读取消息历史记录(超出实时 webhook 事件)。

### **Teams RSC + Microsoft Graph 应用程序权限**
添加:
- 下载托管内容(粘贴到消息中的图像)。
- 下载存储在 SharePoint/OneDrive 中的文件附件。
- 通过 Graph 读取频道/聊天消息历史记录。

### RSC vs Graph API

| 功能 | RSC 权限 | Graph API |
|------------|-----------------|-----------|
| **实时消息** | 是(通过 webhook) | 否(仅轮询) |
| **历史消息** | 否 | 是(可以查询历史记录) |
| **设置复杂性** | 仅应用清单 | 需要管理员同意 + 令牌流 |
| **离线工作** | 否(必须运行) | 是(随时查询) |

**底线:** RSC 用于实时监听；Graph API 用于历史访问。要在离线时获取错过的消息，您需要带有 `ChannelMessage.Read.All` 的 Graph API(需要管理员同意)。

## 支持 Graph 的媒体 + 历史(频道所需)
如果您需要**频道**中的图像/文件或想要获取**消息历史记录**，必须启用 Microsoft Graph 权限并授予管理员同意。

1. 在 Entra ID (Azure AD)**应用注册**中，添加 Microsoft Graph **应用程序权限**:
   - `ChannelMessage.Read.All`(频道附件 + 历史记录)
   - `Chat.Read.All` 或 `ChatMessage.Read.All`(群组聊天)
2. **为租户授予管理员同意**。
3. 增加 Teams 应用**清单版本**，重新上传，并在 Teams 中**重新安装应用**。
4. **完全退出并重新启动 Teams** 以清除缓存的应用元数据。

## 已知限制

### Webhook 超时
Teams 通过 HTTP webhook 传递消息。如果处理时间过长(例如，缓慢的 LLM 响应)，您可能会看到:
- 网关超时
- Teams 重试消息(导致重复)
- 丢弃回复

Moltbot 通过快速返回并主动发送回复来处理此问题，但非常慢的响应仍可能导致问题。

### 格式化
Teams markdown 比 Slack 或 Discord 更有限:
- 基本格式有效: **粗体**、*斜体*、`代码`、链接
- 复杂 markdown(表格、嵌套列表)可能无法正确渲染
- 支持用于投票和任意卡片发送的 Adaptive Cards(见下文)

## 配置
关键设置(共享频道模式参见 `/gateway/configuration`):

- `channels.msteams.enabled`: 启用/禁用频道。
- `channels.msteams.appId`、`channels.msteams.appPassword`、`channels.msteams.tenantId`: 机器人凭据。
- `channels.msteams.webhook.port`(默认 `3978`)
- `channels.msteams.webhook.path`(默认 `/api/messages`)
- `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled`(默认: pairing)
- `channels.msteams.allowFrom`: 私信允许列表(AAD 对象 ID、UPN 或显示名称)。当 Graph 访问可用时，向导在设置期间将名称解析为 ID。
- `channels.msteams.textChunkLimit`: 出站文本块大小。
- `channels.msteams.chunkMode`: `length`(默认)或 `newline` 在长度分块之前按空行(段落边界)分割。
- `channels.msteams.mediaAllowHosts`: 入站附件主机的允许列表(默认为 Microsoft/Teams 域)。
- `channels.msteams.requireMention`: 在频道/群组中需要 @提及(默认 true)。
- `channels.msteams.replyStyle`: `thread | top-level`(参见 [回复样式](#回复样式线程 vs 帖子))。
- `channels.msteams.teams.<teamId>.replyStyle`: 每团队覆盖。
- `channels.msteams.teams.<teamId>.requireMention`: 每团队覆盖。
- `channels.msteams.teams.<teamId>.tools`: 每团队默认工具策略覆盖(当缺少频道覆盖时使用 `allow`/`deny`/`alsoAllow`)。
- `channels.msteams.teams.<teamId>.toolsBySender`: 每团队每发送者工具策略覆盖(支持 `"*"` 通配符)。
- `channels.msteams.teams.<teamId>.channels.<conversationId>.replyStyle`: 每频道覆盖。
- `channels.msteams.teams.<teamId>.channels.<conversationId>.requireMention`: 每频道覆盖。
- `channels.msteams.teams.<teamId>.channels.<conversationId>.tools`: 每频道工具策略覆盖(`allow`/`deny`/`alsoAllow`)。
- `channels.msteams.teams.<teamId>.channels.<conversationId>.toolsBySender`: 每频道每发送者工具策略覆盖(支持 `"*"` 通配符)。
- `channels.msteams.sharePointSiteId`: 用于在群组聊天/频道中上传文件的 SharePoint 站点 ID(参见 [在群组聊天中发送文件](#在群组聊天中发送文件))。

## 路由和会话
- 会话键遵循标准代理格式(参见 [/concepts/session](/concepts/session)):
  - 私信共享主会话(`agent:<agentId>:<mainKey>`)。
  - 频道/群组消息使用对话 id:
    - `agent:<agentId>:msteams:channel:<conversationId>`
    - `agent:<agentId>:msteams:group:<conversationId>`

## 回复样式: 线程 vs 帖子

Teams 最近在同一基础数据模型上引入了两种频道 UI 样式:

| 样式 | 描述 | 推荐的 `replyStyle` |
|-------|-------------|--------------------------|
| **帖子**(经典) | 消息显示为卡片，下面有线程回复 | `thread`(默认) |
| **线程**(类似 Slack) | 消息线性流动，更像 Slack | `top-level` |

**问题:** Teams API 不公开频道使用的 UI 样式。如果您使用错误的 `replyStyle`:
- 线程样式频道中的 `thread` → 回复尴尬地嵌套显示
- 帖子样式频道中的 `top-level` → 回复显示为单独的顶级帖子，而不是在线程中

**解决方案:** 根据频道的设置方式配置每频道的 `replyStyle`:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## 附件和图像

**当前限制:**
- **私信:** 图像和文件附件通过 Teams bot 文件 API 工作。
- **频道/群组:** 附件驻留在 M365 存储(SharePoint/OneDrive)中。webhook 负载仅包含 HTML 存根，而不是实际文件字节。**需要 Graph API 权限**来下载频道附件。

如果没有 Graph 权限，包含图像的频道消息将仅作为文本接收(图像内容对机器人不可访问)。
默认情况下，Moltbot 仅从 Microsoft/Teams 主机名下载媒体。使用 `channels.msteams.mediaAllowHosts` 覆盖(使用 `["*"]` 允许任何主机)。

## 在群组聊天中发送文件

机器人可以使用 FileConsentCard 流程(内置)在私信中发送文件。但是，**在群组聊天/频道中发送文件**需要额外设置:

| 上下文 | 发送文件的方式 | 所需设置 |
|---------|-------------------|--------------|
| **私信** | FileConsentCard → 用户接受 → 机器人上传 | 开箱即用 |
| **群组聊天/频道** | 上传到 SharePoint → 共享链接 | 需要 `sharePointSiteId` + Graph 权限 |
| **图像(任何上下文)** | Base64 编码内联 | 开箱即用 |

### 为什么群组聊天需要 SharePoint

机器人没有个人 OneDrive 驱动器(对于应用程序身份，`/me/drive` Graph API 端点不起作用)。要在群组聊天/频道中发送文件，机器人上传到 **SharePoint 站点**并创建共享链接。

### 设置

1. **在 Entra ID (Azure AD) → 应用注册中添加 Graph API 权限:**
   - `Sites.ReadWrite.All`(应用程序) - 上传文件到 SharePoint
   - `Chat.Read.All`(应用程序) - 可选，启用每用户共享链接

2. **为租户授予管理员同意。**

3. **获取您的 SharePoint 站点 ID:**
   ```bash
   # 通过 Graph Explorer 或使用有效令牌的 curl:
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/{hostname}:/{site-path}"

   # 示例: 对于位于 "contoso.sharepoint.com/sites/BotFiles" 的站点
   curl -H "Authorization: Bearer $TOKEN" \
     "https://graph.microsoft.com/v1.0/sites/contoso.sharepoint.com:/sites/BotFiles"

   # 响应包括: "id": "contoso.sharepoint.com,guid1,guid2"
   ```

4. **配置 Moltbot:**
   ```json5
   {
     channels: {
       msteams: {
         // ... 其他配置 ...
         sharePointSiteId: "contoso.sharepoint.com,guid1,guid2"
       }
     }
   }
   ```

### 共享行为

| 权限 | 共享行为 |
|------------|------------------|
| 仅 `Sites.ReadWrite.All` | 组织范围内共享链接(组织中的任何人都可以访问) |
| `Sites.ReadWrite.All` + `Chat.Read.All` | 每用户共享链接(仅聊天成员可以访问) |

每用户共享更安全，因为只有聊天参与者可以访问文件。如果缺少 `Chat.Read.All` 权限，机器人回退到组织范围共享。

### 回退行为

| 场景 | 结果 |
|----------|--------|
| 群组聊天 + 文件 + 配置了 `sharePointSiteId` | 上传到 SharePoint，发送共享链接 |
| 群组聊天 + 文件 + 无 `sharePointSiteId` | 尝试 OneDrive 上传(可能失败)，仅发送文本 |
| 个人聊天 + 文件 | FileConsentCard 流程(无需 SharePoint 即可工作) |
| 任何上下文 + 图像 | Base64 编码内联(无需 SharePoint 即可工作) |

### 文件存储位置

上传的文件存储在配置的 SharePoint 站点的默认文档库中的 `/MoltbotShared/` 文件夹中。

## 投票(Adaptive Cards)
Moltbot 将 Teams 投票作为 Adaptive Cards 发送(没有原生的 Teams 投票 API)。

- CLI: `moltbot message poll --channel msteams --target conversation:<id> ...`
- 投票由网关记录在 `~/.clawdbot/msteams-polls.json` 中。
- 网关必须保持在线以记录投票。
- 投票不会自动发布结果摘要(如需要，检查存储文件)。

## Adaptive Cards(任意)
使用 `message` 工具或 CLI 向 Teams 用户或对话发送任何 Adaptive Card JSON。

`card` 参数接受 Adaptive Card JSON 对象。当提供 `card` 时，消息文本是可选的。

**代理工具:**
```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:<id>",
  "card": {
    "type": "AdaptiveCard",
    "version": "1.5",
    "body": [{"type": "TextBlock", "text": "Hello!"}]
  }
}
```

**CLI:**
```bash
moltbot message send --channel msteams \
  --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello!"}]}'
```

参见 [Adaptive Cards 文档](https://adaptivecards.io/)获取卡片架构和示例。有关目标格式详细信息，参见下面的[目标格式](#目标格式)。

## 目标格式

MSTeams 目标使用前缀来区分用户和对话:

| 目标类型 | 格式 | 示例 |
|-------------|--------|---------|
| 用户(通过 ID) | `user:<aad-object-id>` | `user:40a1a0ed-4ff2-4164-a219-55518990c197` |
| 用户(通过名称) | `user:<display-name>` | `user:John Smith`(需要 Graph API) |
| 群组/频道 | `conversation:<conversation-id>` | `conversation:19:abc123...@thread.tacv2` |
| 群组/频道(原始) | `<conversation-id>` | `19:abc123...@thread.tacv2`(如果包含 `@thread`) |

**CLI 示例:**
```bash
# 通过 ID 向用户发送
moltbot message send --channel msteams --target "user:40a1a0ed-..." --message "Hello"

# 通过显示名称向用户发送(触发 Graph API 查找)
moltbot message send --channel msteams --target "user:John Smith" --message "Hello"

# 向群组聊天或频道发送
moltbot message send --channel msteams --target "conversation:19:abc...@thread.tacv2" --message "Hello"

# 向对话发送 Adaptive Card
moltbot message send --channel msteams --target "conversation:19:abc...@thread.tacv2" \
  --card '{"type":"AdaptiveCard","version":"1.5","body":[{"type":"TextBlock","text":"Hello"}]}'
```

**代理工具示例:**
```json
{
  "action": "send",
  "channel": "msteams",
  "target": "user:John Smith",
  "message": "Hello!"
}
```

```json
{
  "action": "send",
  "channel": "msteams",
  "target": "conversation:19:abc...@thread.tacv2",
  "card": {"type": "AdaptiveCard", "version": "1.5", "body": [{"type": "TextBlock", "text": "Hello"}]}
}
```

注意: 没有 `user:` 前缀，名称默认为群组/团队解析。按显示名称定位人员时，始终使用 `user:`。

## 主动消息传递
- 主动消息仅在用户交互后**才可能**，因为我们在那时存储对话引用。
- 有关 `dmPolicy` 和允许列表限制，参见 `/gateway/configuration`。

## 团队和频道 ID(常见陷阱)

Teams URL 中的 `groupId` 查询参数**不是**配置使用的团队 ID。从 URL 路径中提取 ID:

**团队 URL:**
```
https://teams.microsoft.com/l/team/19%3ABk4j...%40thread.tacv2/conversations?groupId=...
                                    └────────────────────────────┘
                                    团队 ID(URL 解码此部分)
```

**频道 URL:**
```
https://teams.microsoft.com/l/channel/19%3A15bc...%40thread.tacv2/ChannelName?groupId=...
                                      └─────────────────────────┘
                                      频道 ID(URL 解码此部分)
```

**用于配置:**
- 团队 ID = `/team/` 之后的路径段(URL 解码，例如 `19:Bk4j...@thread.tacv2`)
- 频道 ID = `/channel/` 之后的路径段(URL 解码)
- **忽略** `groupId` 查询参数

## 私人频道

机器人在私人频道中的支持有限:

| 功能 | 标准频道 | 私人频道 |
|---------|-------------------|------------------|
| 机器人安装 | 是 | 有限 |
| 实时消息(webhook) | 是 | 可能不起作用 |
| RSC 权限 | 是 | 可能表现不同 |
| @提及 | 是 | 如果机器人可访问 |
| Graph API 历史 | 是 | 是(需要权限) |

**如果私人频道不起作用的解决方法:**
1. 使用标准频道进行机器人交互
2. 使用私信 - 用户可以直接向机器人发送消息
3. 使用 Graph API 进行历史访问(需要 `ChannelMessage.Read.All`)

## 故障排除

### 常见问题

- **频道中不显示图像:** 缺少 Graph 权限或管理员同意。重新安装 Teams 应用并完全退出/重新打开 Teams。
- **频道中没有回复:** 默认需要提及；设置 `channels.msteams.requireMention=false` 或按团队/频道配置。
- **版本不匹配(Teams 仍显示旧清单):** 删除 + 重新添加应用并完全退出 Teams 以刷新。
- **来自 webhook 的 401 Unauthorized:** 手动测试而没有 Azure JWT 时是预期的 - 表示端点可达但身份验证失败。使用 Azure Web 聊天进行正确测试。

### 清单上传错误

- **"图标文件不能为空":** 清单引用的图标文件为 0 字节。创建有效的 PNG 图标(`outline.png` 为 32x32，`color.png` 为 192x192)。
- **"webApplicationInfo.Id 已在使用中":** 应用仍安装在另一个团队/聊天中。先找到并卸载它，或等待 5-10 分钟以传播。
- **上传时"出问题了":** 通过 https://admin.teams.microsoft.com 上传，打开浏览器 DevTools(F12) → 网络选项卡，并检查响应正文以获取实际错误。
- **侧载失败:** 尝试"将应用上传到组织的应用目录"而不是"上传自定义应用" - 这通常绕过侧载限制。

### RSC 权限不起作用

1. 验证 `webApplicationInfo.id` 与您的机器人的 App ID 完全匹配
2. 重新上传应用并在团队/聊天中重新安装
3. 检查您的组织管理员是否阻止了 RSC 权限
4. 确认您使用正确的范围: 团队使用 `ChannelMessage.Read.Group`，群组聊天使用 `ChatMessage.Read.Chat`

## 参考
- [创建 Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/bot-service-quickstart-registration) - Azure Bot 设置指南
- [Teams 开发者门户](https://dev.teams.microsoft.com/apps) - 创建/管理 Teams 应用
- [Teams 应用清单架构](https://learn.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema)
- [使用 RSC 接收频道消息](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/channel-messages-with-rsc)
- [RSC 权限参考](https://learn.microsoft.com/en-us/microsoftteams/platform/graph-api/rsc/resource-specific-consent)
- [Teams bot 文件处理](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/bots-filesv4)(频道/群组需要 Graph)
- [主动消息传递](https://learn.microsoft.com/en-us/microsoftteams/platform/bots/how-to/conversations/send-proactive-messages)
