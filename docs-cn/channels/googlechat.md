---
summary: "Google Chat 应用支持状态、功能和配置"
read_when:
  - 处理 Google Chat 渠道功能
---
# Google Chat (Chat API)

状态：已准备好通过 Google Chat API webhook 使用私聊 + 群组（仅 HTTP）。

## 快速设置（初学者）
1) 创建一个 Google Cloud 项目并启用 **Google Chat API**。
   - 前往：[Google Chat API 凭据](https://console.cloud.google.com/apis/api/chat.googleapis.com/credentials)
   - 如果尚未启用，请启用 API。
2) 创建一个 **Service Account**：
   - 按 **Create Credentials** > **Service Account**。
   - 随意命名（例如，`moltbot-chat`）。
   - 将权限留空（按 **Continue**）。
   - 将具有访问权限的主体留空（按 **Done**）。
3) 创建并下载 **JSON 密钥**：
   - 在服务帐户列表中，单击您刚创建的服务帐户。
   - 转到 **Keys** 选项卡。
   - 单击 **Add Key** > **Create new key**。
   - 选择 **JSON** 并按 **Create**。
4) 将下载的 JSON 文件存储在您的网关主机上（例如，`~/.clawdbot/googlechat-service-account.json`）。
5) 在 [Google Cloud Console Chat Configuration](https://console.cloud.google.com/apis/api/chat.googleapis.com/hangouts-chat) 中创建一个 Google Chat 应用：
   - 填写 **Application info**：
     - **App name**：（例如，`Moltbot`）
     - **Avatar URL**：（例如，`https://molt.bot/logo.png`）
     - **Description**：（例如，`Personal AI Assistant`）
   - 启用 **Interactive features**。
   - 在 **Functionality** 下，检查 **Join spaces and group conversations**。
   - 在 **Connection settings** 下，选择 **HTTP endpoint URL**。
   - 在 **Triggers** 下，选择 **Use a common HTTP endpoint URL for all triggers** 并将其设置为网关的公共 URL，后跟 `/googlechat`。
     - *提示：运行 `moltbot status` 以查找您的网关公共 URL。*
   - 在 **Visibility** 下，检查 **Make this Chat app available to specific people and groups in &lt;Your Domain&gt;**。
   - 在文本框中输入您的电子邮件地址（例如，`user@example.com`）。
   - 单击底部的 **Save**。
6) **启用应用状态**：
   - 保存后，**刷新页面**。
   - 查找 **App status** 部分（通常在保存后靠近顶部或底部）。
   - 将状态更改为 **Live - available to users**。
   - 再次单击 **Save**。
7) 使用服务帐户路径 + webhook 受众配置 Moltbot：
   - 环境变量：`GOOGLE_CHAT_SERVICE_ACCOUNT_FILE=/path/to/service-account.json`
   - 或配置：`channels.googlechat.serviceAccountFile: "/path/to/service-account.json"`。
8) 设置 webhook 受众类型 + 值（与您的 Chat 应用配置匹配）。
9) 启动网关。Google Chat 将 POST 到您的 webhook 路径。

## 添加到 Google Chat
一旦网关运行并且您的电子邮件被添加到可见性列表：
1) 前往 [Google Chat](https://chat.google.com/)。
2) 单击 **Direct Messages** 旁边的 **+**（加号）图标。
3) 在搜索栏中（通常是添加人员的地方），输入您在 Google Cloud Console 中配置的 **App name**。
   - **注意**：机器人将*不会*出现在"Marketplace"浏览列表中，因为它是私人应用。您必须按名称搜索它。
4) 从结果中选择您的机器人。
5) 单击 **Add** 或 **Chat** 开始 1:1 对话。
6) 发送"Hello"以触发助手！

## 公共 URL（仅 Webhook）
Google Chat webhook 需要公共 HTTPS 端点。为了安全起见，**仅将 `/googlechat` 路径**暴露到互联网。将 Moltbot 仪表板和其他敏感端点保留在您的专用网络上。

### 选项 A：Tailscale Funnel（推荐）
使用 Tailscale Serve 进行私人仪表板，使用 Funnel 进行公共 webhook 路径。这保持 `/` 私有，同时仅暴露 `/googlechat`。

1. **检查您的网关绑定到的地址：**
   ```bash
   ss -tlnp | grep 18789
   ```
   注意 IP 地址（例如，`127.0.0.1`、`0.0.0.0` 或您的 Tailscale IP，如 `100.x.x.x`）。

2. **仅将仪表板暴露到 tailnet（端口 8443）：**
   ```bash
   # 如果绑定到 localhost (127.0.0.1 或 0.0.0.0)：
   tailscale serve --bg --https 8443 http://127.0.0.1:18789

   # 如果仅绑定到 Tailscale IP（例如，100.106.161.80）：
   tailscale serve --bg --https 8443 http://100.106.161.80:18789
   ```

3. **仅公开暴露 webhook 路径：**
   ```bash
   # 如果绑定到 localhost (127.0.0.1 或 0.0.0.0)：
   tailscale funnel --bg --set-path /googlechat http://127.0.0.1:18789/googlechat

   # 如果仅绑定到 Tailscale IP（例如，100.106.161.80）：
   tailscale funnel --bg --set-path /googlechat http://100.106.161.80:18789/googlechat
   ```

4. **授权节点进行 Funnel 访问：**
   如果出现提示，请访问输出中显示的授权 URL，以便为您的 tailnet 策略中的此节点启用 Funnel。

5. **验证配置：**
   ```bash
   tailscale serve status
   tailscale funnel status
   ```

您的公共 webhook URL 将是：
`https://<node-name>.<tailnet>.ts.net/googlechat`

您的私人仪表板保持仅限 tailnet：
`https://<node-name>.<tailnet>.ts.net:8443/`

在 Google Chat 应用配置中使用公共 URL（不带 `:8443`）。

> 注意：此配置在重启后持久存在。要稍后删除它，请运行 `tailscale funnel reset` 和 `tailscale serve reset`。

### 选项 B：反向代理（Caddy）
如果您使用像 Caddy 这样的反向代理，仅代理特定路径：
```caddy
your-domain.com {
    reverse_proxy /googlechat* localhost:18789
}
```
使用此配置，对 `your-domain.com/` 的任何请求都将被忽略或返回为 404，而 `your-domain.com/googlechat` 则安全地路由到 Moltbot。

### 选项 C：Cloudflare Tunnel
配置您的隧道入口规则以仅路由 webhook 路径：
- **Path**：`/googlechat` -> `http://localhost:18789/googlechat`
- **Default Rule**：HTTP 404 (Not Found)

## 工作原理

1. Google Chat 向网关发送 webhook POST。每个请求包括一个 `Authorization: Bearer <token>` 标头。
2. Moltbot 根据配置的 `audienceType` + `audience` 验证令牌：
   - `audienceType: "app-url"` → 受众是您的 HTTPS webhook URL。
   - `audienceType: "project-number"` → 受众是 Cloud 项目编号。
3. 消息按空间路由：
   - DM 使用会话密钥 `agent:<agentId>:googlechat:dm:<spaceId>`。
   - Spaces 使用会话密钥 `agent:<agentId>:googlechat:group:<spaceId>`。
4. DM 访问默认为配对。未知发送者收到配对码；通过以下方式批准：
   - `moltbot pairing approve googlechat <code>`
5. 默认情况下，群组空间需要 @-提及。如果提及检测需要应用的用户名，请使用 `botUser`。

## 目标
使用这些标识符进行投递和白名单：
- 直接消息：`users/<userId>` 或 `users/<email>`（接受电子邮件地址）。
- Spaces：`spaces/<spaceId>`。

## 配置要点
```json5
{
  channels: {
    "googlechat": {
      enabled: true,
      serviceAccountFile: "/path/to/service-account.json",
      audienceType: "app-url",
      audience: "https://gateway.example.com/googlechat",
      webhookPath: "/googlechat",
      botUser: "users/1234567890", // 可选；有助于提及检测
      dm: {
        policy: "pairing",
        allowFrom: ["users/1234567890", "name@example.com"]
      },
      groupPolicy: "allowlist",
      groups: {
        "spaces/AAAA": {
          allow: true,
          requireMention: true,
          users: ["users/1234567890"],
          systemPrompt: "Short answers only."
        }
      },
      actions: { reactions: true },
      typingIndicator: "message",
      mediaMaxMb: 20
    }
  }
}
```

注意：
- 服务帐户凭据也可以通过 `serviceAccount`（JSON 字符串）内联传递。
- 如果未设置 `webhookPath`，默认 webhook 路径为 `/googlechat`。
- 当启用 `actions.reactions` 时，回应可通过 `reactions` 工具和 `channels action` 使用。
- `typingIndicator` 支持 `none`、`message`（默认）和 `reaction`（反应需要用户 OAuth）。
- 附件通过 Chat API 下载并存储在媒体管道中（大小由 `mediaMaxMb` 限制）。

## 故障排除

### 405 Method Not Allowed
如果 Google Cloud Logs Explorer 显示如下错误：
```
status code: 405, reason phrase: HTTP error response: HTTP/1.1 405 Method Not Allowed
```

这意味着 webhook 处理程序未注册。常见原因：
1. **未配置渠道**：您的配置中缺少 `channels.googlechat` 部分。验证：
   ```bash
   moltbot config get channels.googlechat
   ```
   如果返回"Config path not found"，请添加配置（请参阅 [配置要点](#配置要点)）。

2. **未启用插件**：检查插件状态：
   ```bash
   moltbot plugins list | grep googlechat
   ```
   如果显示"disabled"，请添加 `plugins.entries.googlechat.enabled: true` 到您的配置。

3. **未重启网关**：添加配置后，重启网关：
   ```bash
   moltbot gateway restart
   ```

验证渠道正在运行：
```bash
moltbot channels status
# 应该显示：Google Chat default: enabled, configured, ...
```

### 其他问题
- 检查 `moltbot channels status --probe` 以获取身份验证错误或缺少的受众配置。
- 如果没有消息到达，请确认 Chat 应用的 webhook URL + 事件订阅。
- 如果提及门控阻止回复，请将 `botUser` 设置为应用的用户资源名称并验证 `requireMention`。
- 在发送测试消息时使用 `moltbot logs --follow` 以查看请求是否到达网关。

相关文档：
- [网关配置](/gateway/configuration)
- [安全性](/gateway/security)
- [回应](/tools/reactions)
