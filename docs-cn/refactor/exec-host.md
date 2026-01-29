---
summary: "重构计划：exec 主机路由、节点审批和无头运行器"
read_when:
  - 设计 exec 主机路由或 exec 审批
  - 实现节点运行器 + UI IPC
  - 添加 exec 主机安全模式和斜杠命令
---

# Exec 主机重构计划

## 目标
- 添加 `exec.host` + `exec.security` 以在**沙箱**、**网关**和**节点**之间路由执行。
- 保持默认值**安全**：除非明确启用，否则不进行跨主机执行。
- 将执行拆分为**无头运行器服务**，通过本地 IPC 和可选的 UI（macOS 应用）。
- 提供**每代理**策略、允许列表、询问模式和节点绑定。
- 支持**询问模式**，可以有或没有允许列表。
- 跨平台：Unix 套接字 + 令牌认证（macOS/Linux/Windows 平等）。

## 非目标
- 没有旧版允许列表迁移或旧版架构支持。
- 没有 PTY/流式传输用于节点 exec（仅聚合输出）。
- 除了现有的 Bridge + Gateway 之外，没有新的网络层。

## 决策（已锁定）
- **配置键：**`exec.host` + `exec.security`（允许每代理覆盖）。
- **提升：**保留 `/elevated` 作为网关完全访问的别名。
- **询问默认：**`on-miss`。
- **审批存储：**`~/.clawdbot/exec-approvals.json`（JSON，没有旧版迁移）。
- **运行器：**无头系统服务；UI 应用托管用于审批的 Unix 套接字。
- **节点身份：**使用现有的 `nodeId`。
- **套接字认证：**Unix 套接字 + 令牌（跨平台）；如果需要以后拆分。
- **节点主机状态：**`~/.clawdbot/node.json`（节点 id + 配对令牌）。
- **macOS exec 主机：**在 macOS 应用内运行 `system.run`；节点主机服务通过本地 IPC 转发请求。
- **无 XPC 助手：**坚持使用 Unix 套接字 + 令牌 + 对等检查。

## 核心概念
### 主机
- `sandbox`：Docker exec（当前行为）。
- `gateway`：在网关主机上执行。
- `node`：通过 Bridge 在节点运行器上执行（`system.run`）。

### 安全模式
- `deny`：始终阻止。
- `allowlist`：仅允许匹配项。
- `full`：允许所有内容（相当于提升）。

### 询问模式
- `off`：从不询问。
- `on-miss`：仅当允许列表不匹配时询问。
- `always`：每次都询问。

询问**独立于**允许列表；允许列表可以与 `always` 或 `on-miss` 一起使用。

### 策略解决（每次执行）
1) 解决 `exec.host`（工具参数 → 代理覆盖 → 全局默认）。
2) 解决 `exec.security` 和 `exec.ask`（相同的优先级）。
3) 如果主机是 `sandbox`，继续本地沙箱 exec。
4) 如果主机是 `gateway` 或 `node`，在该主机上应用安全 + 询问策略。

## 默认安全性
- 默认 `exec.host = sandbox`。
- 默认 `exec.security = deny` 用于 `gateway` 和 `node`。
- 默认 `exec.ask = on-miss`（仅当安全允许时相关）。
- 如果未设置节点绑定，**代理可以针对任何节点**，但仅当策略允许时。

## 配置表面
### 工具参数
- `exec.host`（可选）：`sandbox | gateway | node`。
- `exec.security`（可选）：`deny | allowlist | full`。
- `exec.ask`（可选）：`off | on-miss | always`。
- `exec.node`（可选）：当 `host=node` 时使用的节点 id/名称。

### 配置键（全局）
- `tools.exec.host`
- `tools.exec.security`
- `tools.exec.ask`
- `tools.exec.node`（默认节点绑定）

### 配置键（每代理）
- `agents.list[].tools.exec.host`
- `agents.list[].tools.exec.security`
- `agents.list[].tools.exec.ask`
- `agents.list[].tools.exec.node`

### 别名
- `/elevated on` = 为代理会话设置 `tools.exec.host=gateway`、`tools.exec.security=full`。
- `/elevated off` = 为代理会话恢复以前的 exec 设置。

## 审批存储（JSON）
路径：`~/.clawdbot/exec-approvals.json`

目的：
- **执行主机**（网关或节点运行器）的本地策略 + 允许列表。
- 当没有 UI 可用时的询问回退。
- UI 客户端的 IPC 凭据。

提议的架构（v1）：
```json
{
  "version": 1,
  "socket": {
    "path": "~/.clawdbot/exec-approvals.sock",
    "token": "base64-opaque-token"
  },
  "defaults": {
    "security": "deny",
    "ask": "on-miss",
    "askFallback": "deny"
  },
  "agents": {
    "agent-id-1": {
      "security": "allowlist",
      "ask": "on-miss",
      "allowlist": [
        {
          "pattern": "~/Projects/**/bin/rg",
          "lastUsedAt": 0,
          "lastUsedCommand": "rg -n TODO",
          "lastResolvedPath": "/Users/user/Projects/.../bin/rg"
        }
      ]
    }
  }
}
```
注意事项：
- 没有旧版允许列表格式。
- `askFallback` 仅在需要 `ask` 且无法访问 UI 时应用。
- 文件权限：`0600`。

## 运行器服务（无头）
### 角色
- 本地强制执行 `exec.security` + `exec.ask`。
- 执行系统命令并返回输出。
- 为 exec 生命周期发出 Bridge 事件（可选但推荐）。

### 服务生命周期
- macOS 上的 launchd/守护进程；Linux/Windows 上的系统服务。
- 审批 JSON 是执行主机的本地。
- UI 托管本地 Unix 套接字；运行器按需连接。

## UI 集成（macOS 应用）
### IPC
- 位于 `~/.clawdbot/exec-approvals.sock` (0600) 的 Unix 套接字。
- 令牌存储在 `exec-approvals.json` (0600) 中。
- 对等检查：仅相同 UID。
- 挑战/响应：nonce + HMAC(token, request-hash) 以防止重放。
- 短 TTL（例如 10s）+ 最大负载 + 速率限制。

### 询问流程（macOS 应用 exec 主机）
1) 节点服务从网关接收 `system.run`。
2) 节点服务连接到本地套接字并发送提示/exec 请求。
3) 应用验证对等 + 令牌 + HMAC + TTL，然后在需要时显示对话框。
4) 应用在 UI 上下文中执行命令并返回输出。
5) 节点服务将输出返回到网关。

如果 UI 缺失：
- 应用 `askFallback`（`deny|allowlist|full`）。

### 图表（SCI）
```
Agent -> Gateway -> Bridge -> Node Service (TS)
                         |  IPC (UDS + token + HMAC + TTL)
                         v
                     Mac App (UI + TCC + system.run)
```

## 节点身份 + 绑定
- 使用来自 Bridge 配对的现有 `nodeId`。
- 绑定模型：
  - `tools.exec.node` 将代理限制为特定节点。
  - 如果未设置，代理可以选择任何节点（策略仍然强制执行默认值）。
- 节点选择解析：
  - `nodeId` 精确匹配
  - `displayName`（标准化）
  - `remoteIp`
  - `nodeId` 前缀（>= 6 个字符）

## 事件
### 谁看到事件
- 系统事件是**每个会话**的，并在下一个提示时显示给代理。
- 存储在网关内存队列中（`enqueueSystemEvent`）。

### 事件文本
- `Exec started (node=<id>, id=<runId>)`
- `Exec finished (node=<id>, id=<runId>, code=<code>)` + 可选输出尾部
- `Exec denied (node=<id>, id=<runId>, <reason>)`

### 传输
选项 A（推荐）：
- 运行器发送 Bridge `event` 帧 `exec.started` / `exec.finished`。
- 网关 `handleBridgeEvent` 将这些映射到 `enqueueSystemEvent`。

选项 B：
- 网关 `exec` 工具直接处理生命周期（仅同步）。

## Exec 流程
### 沙箱主机
- 现有的 `exec` 行为（Docker 或未沙箱化时为主机）。
- PTY 仅在非沙箱模式下受支持。

### 网关主机
- 网关进程在自己的机器上执行。
- 强制执行本地 `exec-approvals.json`（security/ask/allowlist）。

### 节点主机
- 网关使用 `system.run` 调用 `node.invoke`。
- 运行器强制执行本地审批。
- 运行器返回聚合的 stdout/stderr。
- 可选的 Bridge 事件用于开始/完成/拒绝。

## 输出限制
- 将组合的 stdout+stderr 限制在 **200k**；保留 **tail 20k** 用于事件。
- 使用清晰的后缀截断（例如，`"… (truncated)"`）。

## 斜杠命令
- `/exec host=<sandbox|gateway|node> security=<deny|allowlist|full> ask=<off|on-miss|always> node=<id>`
- 每代理、每会话覆盖；除非通过配置保存，否则非持久。
- `/elevated on|off|ask|full` 仍然是 `host=gateway security=full` 的快捷方式（`full` 跳过审批）。

## 跨平台方案
- 运行器服务是可移植的执行目标。
- UI 是可选的；如果缺失，应用 `askFallback`。
- Windows/Linux 支持相同的审批 JSON + 套接字协议。

## 实现阶段
### 第 1 阶段：配置 + exec 路由
- 为 `exec.host`、`exec.security`、`exec.ask`、`exec.node` 添加配置架构。
- 更新工具管道以尊重 `exec.host`。
- 添加 `/exec` 斜杠命令并保留 `/elevated` 别名。

### 第 2 阶段：审批存储 + 网关强制执行
- 实现 `exec-approvals.json` 读取器/写入器。
- 为 `gateway` 主机强制执行允许列表 + 询问模式。
- 添加输出限制。

### 第 3 阶段：节点运行器强制执行
- 更新节点运行器以强制执行允许列表 + 询问。
- 将 Unix 套接字提示桥接添加到 macOS 应用 UI。
- 连接 `askFallback`。

### 第 4 阶段：事件
- 为 exec 生命周期添加节点 → 网关 Bridge 事件。
- 映射到 `enqueueSystemEvent` 以用于代理提示。

### 第 5 阶段：UI 打磨
- Mac 应用：允许列表编辑器、每代理切换器、询问策略 UI。
- 节点绑定控件（可选）。

## 测试计划
- 单元测试：允许列表匹配（glob + 不区分大小写）。
- 单元测试：策略解决优先级（工具参数 → 代理覆盖 → 全局）。
- 集成测试：节点运行器拒绝/允许/询问流程。
- Bridge 事件测试：节点事件 → 系统事件路由。

## 开放风险
- UI 不可用：确保尊重 `askFallback`。
- 长时间运行的命令：依赖超时 + 输出限制。
- 多节点模糊：除非节点绑定或明确的节点参数，否则出错。

## 相关文档
- [Exec 工具](/tools/exec)
- [Exec 审批](/tools/exec-approvals)
- [节点](/nodes)
- [提升模式](/tools/elevated)
