---
summary: "每个代理的沙箱 + 工具限制、优先级和示例"
title: 多代理沙箱与工具
read_when: "您想在多代理网关中为每个代理配置沙箱或每个代理的工具允许/拒绝策略"
status: active
---

# 多代理沙箱与工具配置

## 概述

在多代理设置中，每个代理现在可以拥有自己的配置：
- **沙箱配置**（`agents.list[].sandbox` 覆盖 `agents.defaults.sandbox`）
- **工具限制**（`tools.allow` / `tools.deny`，以及 `agents.list[].tools`）

这允许您运行具有不同安全配置文件的多个代理：
- 具有完全访问权限的个人助手
- 工具受限的家庭/工作代理
- 沙箱中的面向公众的代理

`setupCommand` 属于 `sandbox.docker`（全局或每个代理）的一部分，并在容器创建时运行一次。

认证是每个代理的：每个代理从自己的 `agentDir` 认证存储中读取，位于：

```
~/.clawdbot/agents/<agentId>/agent/auth-profiles.json
```

凭据在代理之间**不**共享。切勿在代理之间重用 `agentDir`。
如果您想共享凭据，请将 `auth-profiles.json` 复制到另一个代理的 `agentDir` 中。

关于沙箱在运行时的行为，请参阅 [沙箱](/gateway/sandboxing)。
关于调试"为什么这个被阻止？"，请参阅 [沙箱与工具策略与提升模式](/gateway/sandbox-vs-tool-policy-vs-elevated) 和 `moltbot sandbox explain`。

---

## 配置示例

### 示例 1：个人 + 受限家庭代理

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "Personal Assistant",
        "workspace": "~/clawd",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "Family Bot",
        "workspace": "~/clawd-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**结果：**
- `main` 代理：在主机上运行，具有完整的工具访问权限
- `family` 代理：在 Docker 中运行（每个代理一个容器），只有 `read` 工具

---

### 示例 2：具有共享沙箱的工作代理

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/clawd-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/clawd-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

---

### 示例 2b：全局编码配置文件 + 仅消息传递代理

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**结果：**
- 默认代理获得编码工具
- `support` 代理仅用于消息传递（+ Slack 工具）

---

### 示例 3：每个代理的不同沙箱模式

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // 全局默认值
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/clawd",
        "sandbox": {
          "mode": "off"  // 覆盖：main 从不沙箱化
        }
      },
      {
        "id": "public",
        "workspace": "~/clawd-public",
        "sandbox": {
          "mode": "all",  // 覆盖：public 始终沙箱化
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

---

## 配置优先级

当同时存在全局（`agents.defaults.*`）和代理特定（`agents.list[].*`）配置时：

### 沙箱配置
代理特定设置覆盖全局设置：
```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**注意：**
- `agents.list[].sandbox.{docker,browser,prune}.*` 覆盖该代理的 `agents.defaults.sandbox.{docker,browser,prune}.*`（当沙箱范围解析为 `"shared"` 时忽略）。

### 工具限制
过滤顺序为：
1. **工具配置文件**（`tools.profile` 或 `agents.list[].tools.profile`）
2. **提供商工具配置文件**（`tools.byProvider[provider].profile` 或 `agents.list[].tools.byProvider[provider].profile`）
3. **全局工具策略**（`tools.allow` / `tools.deny`）
4. **提供商工具策略**（`tools.byProvider[provider].allow/deny`）
5. **代理特定工具策略**（`agents.list[].tools.allow/deny`）
6. **代理提供商策略**（`agents.list[].tools.byProvider[provider].allow/deny`）
7. **沙箱工具策略**（`tools.sandbox.tools` 或 `agents.list[].tools.sandbox.tools`）
8. **子代理工具策略**（`tools.subagents.tools`，如果适用）

每一层都可以进一步限制工具，但不能恢复早期层中拒绝的工具。
如果设置了 `agents.list[].tools.sandbox.tools`，它将替换该代理的 `tools.sandbox.tools`。
如果设置了 `agents.list[].tools.profile`，它将覆盖该代理的 `tools.profile`。
提供商工具键接受 `provider`（例如 `google-antigravity`）或 `provider/model`（例如 `openai/gpt-5.2`）。

### 工具组（简写）

工具策略（全局、代理、沙箱）支持 `group:*` 条目，可扩展为多个具体工具：

- `group:runtime`：`exec`、`bash`、`process`
- `group:fs`：`read`、`write`、`edit`、`apply_patch`
- `group:sessions`：`sessions_list`、`sessions_history`、`sessions_send`、`sessions_spawn`、`session_status`
- `group:memory`：`memory_search`、`memory_get`
- `group:ui`：`browser`、`canvas`
- `group:automation`：`cron`、`gateway`
- `group:messaging`：`message`
- `group:nodes`：`nodes`
- `group:moltbot`：所有内置 Moltbot 工具（不包括提供商插件）

### 提升模式
`tools.elevated` 是全局基线（基于发送者的允许列表）。`agents.list[].tools.elevated` 可以进一步限制特定代理的提升模式（两者都必须允许）。

缓解模式：
- 拒绝不受信任代理的 `exec`（`agents.list[].tools.deny: ["exec"]`）
- 避免将路由到受限代理的发送者列入允许列表
- 如果您只想沙箱化执行，请在全局禁用提升模式（`tools.elevated.enabled: false`）
- 为敏感配置文件禁用每个代理的提升模式（`agents.list[].tools.elevated.enabled: false`）

---

## 从单个代理迁移

**之前（单个代理）：**
```json
{
  "agents": {
    "defaults": {
      "workspace": "~/clawd",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**之后（具有不同配置文件的多代理）：**
```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/clawd",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

传统的 `agent.*` 配置由 `moltbot doctor` 迁移；今后请使用 `agents.defaults` + `agents.list`。

---

## 工具限制示例

### 只读代理
```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### 安全执行代理（无文件修改）
```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### 仅通信代理
```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

---

## 常见陷阱："non-main"

`agents.defaults.sandbox.mode: "non-main"` 基于 `session.mainKey`（默认为 `"main"`），
而不是代理 ID。群组/频道会话总是获得自己的键，因此
它们被视为非主代理并将被沙箱化。如果您希望代理从不
沙箱化，请设置 `agents.list[].sandbox.mode: "off"`。

---

## 测试

配置多代理沙箱和工具后：

1. **检查代理解析：**
   ```bash
   moltbot agents list --bindings
   ```

2. **验证沙箱容器：**
   ```bash
   docker ps --filter "label=moltbot.sandbox=1"
   ```

3. **测试工具限制：**
   - 发送需要受限工具的消息
   - 验证代理无法使用被拒绝的工具

4. **监控日志：**
   ```bash
   tail -f "${CLAWDBOT_STATE_DIR:-$HOME/.clawdbot}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

---

## 故障排除

### 尽管设置了 `mode: "all"`，代理仍未沙箱化
- 检查是否存在覆盖它的全局 `agents.defaults.sandbox.mode`
- 代理特定配置优先，因此请设置 `agents.list[].sandbox.mode: "all"`

### 尽管有拒绝列表，工具仍然可用
- 检查工具过滤顺序：全局 → 代理 → 沙箱 → 子代理
- 每一层只能进一步限制，不能恢复
- 使用日志验证：`[tools] filtering tools for agent:${agentId}`

### 容器未按代理隔离
- 在代理特定的沙箱配置中设置 `scope: "agent"`
- 默认为 `"session"`，这会为每个会话创建一个容器

---

## 另请参阅

- [多代理路由](/concepts/multi-agent)
- [沙箱配置](/gateway/configuration#agentsdefaults-sandbox)
- [会话管理](/concepts/session)
