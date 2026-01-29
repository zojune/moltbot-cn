---
summary: "执行批准、允许列表和沙箱逃逸提示"
read_when:
  - 配置执行批准或允许列表
  - 在 macOS 应用中实现执行批准 UX
  - 审查沙箱逃逸提示和影响
---

# 执行批准

执行批准是**配套应用 / 节点主机防护**，用于让沙箱代理在真实主机（`gateway` 或 `node`）上运行命令。将其视为安全联锁：只有当策略 + 允许列表 + （可选）用户批准都一致时，才允许命令。执行批准是工具策略和提升限制**之外**的附加（除非提升设置为 `full`，这会跳过批准）。有效策略是 `tools.exec.*` 和批准默认值的**更严格**者；如果省略批准字段，则使用 `tools.exec` 值。

如果配套应用 UI **不可用**，任何需要提示的请求都由**询问回退**解决（默认：拒绝）。

## 应用位置

执行批准在执行主机上本地强制执行：
- **网关主机** → 网关机器上的 `moltbot` 进程
- **节点主机** → 节点运行程序（macOS 配套应用或无头节点主机）

macOS 拆分：
- **节点主机服务**通过本地 IPC 将 `system.run` 转发到 **macOS 应用**。
- **macOS 应用**强制执行批准 + 在 UI 上下文中执行命令。

## 设置和存储

批准存储在执行主机上的本地 JSON 文件中：

`~/.clawdbot/exec-approvals.json`

示例架构：
```json
{
  "version": 1,
  "socket": {
    "path": "~/.clawdbot/exec-approvals.sock",
    "token": "base64url-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny",
    "autoAllowSkills": false
  },
  "agents": {
    "main": {
      "security": "allowlist",
      "ask": "on-miss",
      "askFallback": "deny",
      "autoAllowSkills": true,
      "allowlist": [
        {
          "id": "B0C8C0B3-2C2D-4F8A-9A3C-5A4B3C2D1E0F",
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 1737150000000,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```

## 策略控制

### 安全性（`exec.security`）
- **deny**：阻止所有主机 exec 请求。
- **allowlist**：仅允许允许列表中的命令。
- **full**：允许所有内容（等同于提升）。

### 询问（`exec.ask`）
- **off**：从不提示。
- **on-miss**：仅当允许列表不匹配时提示。
- **always**：在每个命令上提示。

### 询问回退（`askFallback`）
如果需要提示但无法访问 UI，回退决定：
- **deny**：阻止。
- **allowlist**：仅当允许列表匹配时允许。
- **full**：允许。

## 允许列表（每代理）

允许列表是**每代理**的。如果存在多个代理，请在 macOS 应用中切换您正在编辑的代理。模式是**不区分大小写的 glob 匹配**。模式应解析为**二进制路径**（仅基名称的条目将被忽略）。加载时，旧的 `agents.default` 条目会迁移到 `agents.main`。

示例：
- `~/Projects/**/bin/bird`
- `~/.local/bin/*`
- `/opt/homebrew/bin/rg`

每个允许列表条目跟踪：
- **id** 用于 UI 身份的稳定 UUID（可选）
- **最后使用**时间戳
- **最后使用命令**
- **最后解析路径**

## 自动允许技能 CLI

当启用**自动允许技能 CLI**时，已知技能引用的可执行文件在节点上被视为允许列表（macOS 节点或无头节点主机）。这通过网关 RPC 使用 `skills.bins` 获取技能 bin 列表。如果您想要严格的手动允许列表，请禁用此功能。

## 安全 bin（仅 stdin）

`tools.exec.safeBins` 定义了一个小的**仅 stdin**二进制文件列表（例如 `jq`），可以在允许列表模式下运行而**无需**显式允许列表条目。安全 bin 拒绝位置文件参数和类似路径的令牌，因此它们只能对传入流进行操作。Shell 链接和重定向在允许列表模式下不会自动允许。

当每个顶级段都满足允许列表（包括安全 bin 或技能自动允许）时，允许 Shell 链接（`&&`、`||`、`;`）。在允许列表模式下仍不支持重定向。

默认安全 bin：`jq`、`grep`、`cut`、`sort`、`uniq`、`head`、`tail`、`tr`、`wc`。

## 控制 UI 编辑

使用**控制 UI → 节点 → 执行批准**卡片来编辑默认值、每代理覆盖和允许列表。选择一个范围（默认值或代理），调整策略，添加/删除允许列表模式，然后**保存**。UI 显示每个模式的**最后使用**元数据，以便您可以保持列表整洁。

目标选择器选择**网关**（本地批准）或**节点**。节点必须公告 `system.execApprovals.get/set`（macOS 应用或无头节点主机）。如果节点尚未公告执行批准，请直接编辑其本地 `~/.clawdbot/exec-approvals.json`。

CLI：`moltbot approvals` 支持网关或节点编辑（请参阅[批准 CLI](/cli/approvals)）。

## 批准流程

当需要提示时，网关向操作员客户端广播 `exec.approval.requested`。控制 UI 和 macOS 应用通过 `exec.approval.resolve` 解决它，然后网关将批准的请求转发到节点主机。

当需要批准时，exec 工具立即返回批准 id。使用该 id 关联后续系统事件（`Exec finished` / `Exec denied`）。如果在超时之前没有做出决定，请求将被视为批准超时并作为拒绝原因公开。

确认对话框包括：
- 命令 + 参数
- cwd
- 代理 id
- 解析的可执行文件路径
- 主机 + 策略元数据

操作：
- **允许一次** → 现在运行
- **始终允许** → 添加到允许列表 + 运行
- **拒绝** → 阻止

## 转发批准到聊天频道

您可以将执行批准提示转发到任何聊天频道（包括插件频道）并使用 `/approve` 批准它们。这使用正常的出站投递管道。

配置：
```json5
{
  approvals: {
    exec: {
      enabled: true,
      mode: "session", // "session" | "targets" | "both"
      agentFilter: ["main"],
      sessionFilter: ["discord"], // 子字符串或正则表达式
      targets: [
        { channel: "slack", to: "U12345678" },
        { channel: "telegram", to: "123456789" }
      ]
    }
  }
}
```

在聊天中回复：
```
/approve <id> allow-once
/approve <id> allow-always
/approve <id> deny
```

### macOS IPC 流程
```
Gateway -> Node Service (WS)
                 |  IPC (UDS + token + HMAC + TTL)
                 v
             Mac App (UI + approvals + system.run)
```

安全说明：
- Unix 套接字模式 `0600`，令牌存储在 `exec-approvals.json` 中。
- 相同 UID 对等检查。
- 挑战/响应（nonce + HMAC 令牌 + 请求哈希）+ 短 TTL。

## 系统事件

执行生命周期作为系统消息公开：
- `Exec running`（仅当命令超过运行通知阈值时）
- `Exec finished`
- `Exec denied`

这些在节点报告事件后发布到代理的会话。网关主机执行批准在命令完成时（以及在运行时间超过阈值时可选地）发出相同的生命周期事件。批准限制的 exec 在这些消息中重用批准 id 作为 `runId` 以便于关联。

## 影响

- **full** 功能强大；尽可能首选允许列表。
- **ask** 让您保持了解，同时仍然允许快速批准。
- 每代理允许列表防止一个代理的批准泄漏到其他代理。
- 批准仅适用于来自**授权发送者**的主机 exec 请求。未经授权的发送者无法发出 `/exec`。
- `/exec security=full` 是授权操作员的会话级别便利，并设计为跳过批准。
  要硬阻止主机 exec，请将批准安全性设置为 `deny` 或通过工具策略拒绝 `exec` 工具。

相关：
- [Exec 工具](/tools/exec)
- [提升模式](/tools/elevated)
- [技能](/tools/skills)
