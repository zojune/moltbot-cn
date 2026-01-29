---
summary: "Per-代理 沙箱 + 工具 restrictions, precedence, and 示例"
title: Multi-代理 沙箱 & 工具
read_when: "You want per-代理 sandboxing or per-代理 工具 allow/deny 策略 in a multi-代理 Gateway."
status: active
---

# Multi-代理 沙箱 & 工具 配置

## 概述

Each 代理 in a multi-代理 设置 can now have its own:
- **Sandbox configuration** (`agents.list[].sandbox` overrides `agents.defaults.sandbox`)
- **Tool restrictions** (`tools.allow` / `tools.deny`, plus `agents.list[].tools`)

This allows you to run multiple 代理 with different 安全 配置文件:
- Personal assistant with full access
- Family/work 代理 with restricted 工具
- Public-facing 代理 in 沙箱

`setupCommand` belongs under `sandbox.docker` (global or per-代理) and runs once
when the container is created.

Auth is per-agent: each agent reads from its own `agentDir` 认证 store at:

```
~/.clawdbot/agents/<agentId>/agent/auth-profiles.json
```

Credentials are **not** shared between agents. Never reuse `agentDir` across 代理.
If you want to share creds, copy `auth-profiles.json` into the other agent's `agentDir`.

For how sandboxing behaves at runtime, 参见 [Sandboxing](/Gateway/sandboxing).
For debugging “why is this blocked?”, see [Sandbox vs Tool Policy vs Elevated](/gateway/sandbox-vs-tool-policy-vs-elevated) and `moltbot sandbox explain`.

---

## 配置 示例

### 示例 1: Personal + Restricted Family 代理

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

**结果:**
- `main` 代理: Runs on 主机, full 工具 access
- `family` agent: Runs in Docker (one container per agent), only `read` 工具

---

### 示例 2: Work 代理 with Shared 沙箱

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

### 示例 2b: Global coding 配置文件 + messaging-only 代理

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

**结果:**
- 默认 代理 get coding 工具
- `support` 代理 is messaging-only (+ Slack 工具)

---

### 示例 3: Different 沙箱 Modes per 代理

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main",  // Global default
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/clawd",
        "sandbox": {
          "mode": "off"  // Override: main never sandboxed
        }
      },
      {
        "id": "public",
        "workspace": "~/clawd-public",
        "sandbox": {
          "mode": "all",  // Override: public always sandboxed
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

## 配置 Precedence

When both global (`agents.defaults.*`) and agent-specific (`agents.list[].*`) configs exist:

### 沙箱 配置
代理-specific 设置 override global:
```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**Notes:**
- `agents.list[].sandbox.{docker,browser,prune}.*` overrides `agents.defaults.sandbox.{docker,browser,prune}.*` for that agent (ignored when sandbox scope resolves to `"shared"`).

### 工具 Restrictions
The filtering order is:
1. **Tool profile** (`tools.profile` or `agents.list[].tools.profile`)
2. **Provider tool profile** (`tools.byProvider[provider].profile` or `agents.list[].tools.byProvider[provider].profile`)
3. **Global tool policy** (`tools.allow` / `tools.deny`)
4. **Provider tool policy** (`tools.byProvider[provider].allow/deny`)
5. **Agent-specific tool policy** (`agents.list[].tools.allow/deny`)
6. **Agent provider policy** (`agents.list[].tools.byProvider[provider].allow/deny`)
7. **Sandbox tool policy** (`tools.sandbox.tools` or `agents.list[].tools.sandbox.tools`)
8. **Subagent tool policy** (`tools.subagents.tools`, if applicable)

Each level can further restrict 工具, but cannot grant back denied 工具 from earlier levels.
If `agents.list[].tools.sandbox.tools` is set, it replaces `tools.sandbox.tools` for that 代理.
If `agents.list[].tools.profile` is set, it overrides `tools.profile` for that 代理.
Provider tool keys accept either `provider` (e.g. `google-antigravity`) or `provider/model` (e.g. `openai/gpt-5.2`).

### 工具 groups (shorthands)

Tool policies (global, agent, sandbox) support `group:*` entries that expand to multiple concrete 工具:

- `group:runtime`: `exec`, `bash`, `process`
- `group:fs`: `read`, `write`, `edit`, `apply_patch`
- `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
- `group:memory`: `memory_search`, `memory_get`
- `group:ui`: `browser`, `canvas`
- `group:automation`: `cron`, `gateway`
- `group:messaging`: `message`
- `group:nodes`: `nodes`
- `group:moltbot`: all built-in Moltbot 工具 (excludes 提供商 插件)

### Elevated Mode
`tools.elevated` is the global baseline (sender-based allowlist). `agents.list[].tools.elevated` can further restrict elevated for specific 代理 (both must allow).

Mitigation patterns:
- Deny `exec` for untrusted agents (`agents.list[].tools.deny: ["exec"]`)
- Avoid allowlisting senders that route to restricted 代理
- Disable elevated globally (`tools.elevated.enabled: false`) if you only want sandboxed execution
- Disable elevated per agent (`agents.list[].tools.elevated.enabled: false`) for sensitive 配置文件

---

## 迁移 from Single 代理

**Before (single 代理):**
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

**After (multi-代理 with different 配置文件):**
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

Legacy `agent.*` configs are migrated by `moltbot doctor`; prefer `agents.defaults` + `agents.list` going forward.

---

## 工具 Restriction 示例

### Read-only 代理
```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### Safe Execution 代理 (no 文件 modifications)
```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### Communication-only 代理
```json
{
  "tools": {
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

---

## Common Pitfall: "non-main"

`agents.defaults.sandbox.mode: "non-main"` is based on `session.mainKey` (default `"main"`),
not the 代理 id. Group/渠道 会话 always get their own keys, so they
are treated as non-main and will be sandboxed. If you want an 代理 to never
sandbox, set `agents.list[].sandbox.mode: "off"`.

---

## 测试

After configuring multi-代理 沙箱 and 工具:

1. **Check 代理 resolution:**
   ```exec
   moltbot agents list --bindings
   ```

2. **Verify 沙箱 containers:**
   ```exec
   docker ps --filter "label=moltbot.sandbox=1"
   ```

3. **测试 工具 restrictions:**
   - Send a 消息 requiring restricted 工具
   - Verify the 代理 cannot use denied 工具

4. **Monitor 日志:**
   ```exec
   tail -f "${CLAWDBOT_STATE_DIR:-$HOME/.clawdbot}/logs/gateway.log" | grep -E "routing|sandbox|tools"
   ```

---

## 故障排除

### Agent not sandboxed despite `mode: "all"`
- Check if there's a global `agents.defaults.sandbox.mode` that overrides it
- Agent-specific config takes precedence, so set `agents.list[].sandbox.mode: "all"`

### 工具 still available despite deny list
- Check 工具 filtering order: global → 代理 → 沙箱 → subagent
- Each level can only further restrict, not grant back
- Verify with logs: `[tools] filtering tools for agent:${agentId}`

### Container not isolated per 代理
- Set `scope: "agent"` in 代理-specific 沙箱 配置
- Default is `"session"` which creates one container per 会话

---

## 另请参阅

- [Multi-代理 Routing](/concepts/multi-代理)
- [沙箱 配置](/Gateway/配置#agentsdefaults-沙箱)
- [会话 Management](/concepts/会话)
