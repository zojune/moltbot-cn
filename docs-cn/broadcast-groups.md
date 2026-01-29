---
summary: "Broadcast a WhatsApp 消息 to multiple 代理"
read_when: 
  - Configuring broadcast groups
  - Debugging multi-agent replies in WhatsApp
status: experimental
---

# Broadcast Groups

**状态:** Experimental  
**版本:** Added in 2026.1.9

## 概述

Broadcast Groups enable multiple 代理 to 进程 and respond to the same 消息 simultaneously. This allows you to create specialized 代理 teams that work together in a single WhatsApp group or DM — all using one phone 数字.

Current scope: **WhatsApp only** (Web 渠道).

Broadcast groups are evaluated after 渠道 allowlists and group activation 规则. In WhatsApp groups, this means broadcasts happen when Moltbot would normally reply (例如: on mention, depending on your group 设置).

## 用例

### 1. Specialized 代理 Teams
Deploy multiple 代理 with atomic, focused responsibilities:
```
Group: "Development Team"
Agents:
  - CodeReviewer (reviews code snippets)
  - DocumentationBot (generates docs)
  - SecurityAuditor (checks for vulnerabilities)
  - TestGenerator (suggests test cases)
```

Each 代理 进程 the same 消息 and provides its specialized perspective.

### 2. Multi-Language Support
```
Group: "International Support"
Agents:
  - Agent_EN (responds in English)
  - Agent_DE (responds in German)
  - Agent_ES (responds in Spanish)
```

### 3. Quality Assurance Workflows
```
Group: "Customer Support"
Agents:
  - SupportAgent (provides answer)
  - QAAgent (reviews quality, only responds if issues found)
```

### 4. Task Automation
```
Group: "Project Management"
Agents:
  - TaskTracker (updates task database)
  - TimeLogger (logs time spent)
  - ReportGenerator (creates summaries)
```

## 配置

### Basic 设置

Add a top-level `broadcast` section (next to `bindings`). Keys are WhatsApp peer ids:
- group chats: group JID (e.g. `120363403215116621@g.us`)
- DMs: E.164 phone number (e.g. `+15551234567`)

```json
{
  "broadcast": {
    "120363403215116621@g.us": ["alfred", "baerbel", "assistant3"]
  }
}
```

**结果:** When Moltbot would reply in this chat, it will run all three 代理.

### Processing Strategy

Control how 代理 进程 消息:

#### Parallel (默认)
All 代理 进程 simultaneously:
```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

#### Sequential
代理 进程 in order (one waits for previous to finish):
```json
{
  "broadcast": {
    "strategy": "sequential",
    "120363403215116621@g.us": ["alfred", "baerbel"]
  }
}
```

### Complete 示例

```json
{
  "agents": {
    "list": [
      {
        "id": "code-reviewer",
        "name": "Code Reviewer",
        "workspace": "/path/to/code-reviewer",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "security-auditor",
        "name": "Security Auditor",
        "workspace": "/path/to/security-auditor",
        "sandbox": { "mode": "all" }
      },
      {
        "id": "docs-generator",
        "name": "Documentation Generator",
        "workspace": "/path/to/docs-generator",
        "sandbox": { "mode": "all" }
      }
    ]
  },
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": ["code-reviewer", "security-auditor", "docs-generator"],
    "120363424282127706@g.us": ["support-en", "support-de"],
    "+15555550123": ["assistant", "logger"]
  }
}
```

## 工作原理

### 消息 Flow

1. **Incoming 消息** arrives in a WhatsApp group
2. **Broadcast check**: System checks if peer ID is in `broadcast`
3. **If in broadcast list**:
   - All listed 代理 进程 the 消息
   - Each 代理 has its own 会话 键 and isolated 上下文
   - 代理 进程 in parallel (默认) or sequentially
4. **If not in broadcast list**:
   - Normal routing applies (first matching 绑定)

注意: broadcast groups do not bypass 渠道 allowlists or group activation 规则 (mentions/命令/etc). They only change *which 代理 run* when a 消息 is eligible for processing.

### 会话 Isolation

Each 代理 in a broadcast group maintains completely separate:

- **Session keys** (`agent:alfred:whatsapp:group:120363...` vs `agent:baerbel:whatsapp:group:120363...`)
- **Conversation history** (代理 doesn't 参见 other 代理' 消息)
- **工作空间** (separate 沙箱 if configured)
- **工具 access** (different allow/deny lists)
- **Memory/上下文** (separate IDENTITY.md, SOUL.md, etc.)
- **Group 上下文 缓冲区** (recent group 消息 used for 上下文) is shared per peer, so all broadcast 代理 参见 the same 上下文 when triggered

This allows each 代理 to have:
- Different personalities
- Different 工具 access (e.g., read-only vs. read-write)
- Different 模型 (e.g., opus vs. sonnet)
- Different 技能 installed

### 示例: Isolated 会话

In group `120363403215116621@g.us` with agents `["alfred", "baerbel"]`:

**Alfred's 上下文:**
```
Session: agent:alfred:whatsapp:group:120363403215116621@g.us
History: [user message, alfred's previous responses]
Workspace: /Users/pascal/clawd-alfred/
Tools: read, write, exec
```

**Bärbel's 上下文:**
```
Session: agent:baerbel:whatsapp:group:120363403215116621@g.us  
History: [user message, baerbel's previous responses]
Workspace: /Users/pascal/clawd-baerbel/
Tools: read only
```

## 最佳实践

### 1. Keep 代理 Focused

Design each 代理 with a single, clear responsibility:

```json
{
  "broadcast": {
    "DEV_GROUP": ["formatter", "linter", "tester"]
  }
}
```

✅ **Good:** Each 代理 has one job  
❌ **Bad:** One generic "dev-helper" 代理

### 2. Use Descriptive Names

Make it clear what each 代理 does:

```json
{
  "agents": {
    "security-scanner": { "name": "Security Scanner" },
    "code-formatter": { "name": "Code Formatter" },
    "test-generator": { "name": "Test Generator" }
  }
}
```

### 3. Configure Different 工具 Access

Give 代理 only the 工具 they need:

```json
{
  "agents": {
    "reviewer": {
      "tools": { "allow": ["read", "exec"] }  // Read-only
    },
    "fixer": {
      "tools": { "allow": ["read", "write", "edit", "exec"] }  // Read-write
    }
  }
}
```

### 4. Monitor 性能

With many 代理, consider:
- Using `"strategy": "parallel"` (默认) for speed
- Limiting broadcast groups to 5-10 代理
- Using faster 模型 for simpler 代理

### 5. Handle Failures Gracefully

代理 fail independently. One 代理's 错误 doesn't block others:

```
Message → [Agent A ✓, Agent B ✗ error, Agent C ✓]
Result: Agent A and C respond, Agent B logs error
```

## Compatibility

### 提供商

Broadcast groups currently work with:
- ✅ WhatsApp (implemented)
- 🚧 Telegram (planned)
- 🚧 Discord (planned)
- 🚧 Slack (planned)

### Routing

Broadcast groups work alongside existing routing:

```json
{
  "bindings": [
    { "match": { "channel": "whatsapp", "peer": { "kind": "group", "id": "GROUP_A" } }, "agentId": "alfred" }
  ],
  "broadcast": {
    "GROUP_B": ["agent1", "agent2"]
  }
}
```

- `GROUP_A`: Only alfred responds (normal routing)
- `GROUP_B`: agent1 AND agent2 respond (broadcast)

**Precedence:** `broadcast` takes priority over `bindings`.

## 故障排除

### 代理 Not Responding

**Check:**
1. Agent IDs exist in `agents.list`
2. Peer ID format is correct (e.g., `120363403215116621@g.us`)
3. 代理 are not in deny lists

**调试:**
```bash
tail -f ~/.clawdbot/logs/gateway.log | grep broadcast
```

### Only One 代理 Responding

**Cause:** Peer ID might be in `bindings` but not `broadcast`.

**Fix:** Add to broadcast 配置 or remove from 绑定.

### 性能 Issues

**If slow with many 代理:**
- Reduce 数字 of 代理 per group
- Use lighter 模型 (sonnet instead of opus)
- Check 沙箱 startup time

## 示例

### 示例 1: Code Review Team

```json
{
  "broadcast": {
    "strategy": "parallel",
    "120363403215116621@g.us": [
      "code-formatter",
      "security-scanner",
      "test-coverage",
      "docs-checker"
    ]
  },
  "agents": {
    "list": [
      { "id": "code-formatter", "workspace": "~/agents/formatter", "tools": { "allow": ["read", "write"] } },
      { "id": "security-scanner", "workspace": "~/agents/security", "tools": { "allow": ["read", "exec"] } },
      { "id": "test-coverage", "workspace": "~/agents/testing", "tools": { "allow": ["read", "exec"] } },
      { "id": "docs-checker", "workspace": "~/agents/docs", "tools": { "allow": ["read"] } }
    ]
  }
}
```

**用户 sends:** Code snippet  
**响应:**
- code-formatter: "Fixed indentation and added 类型 hints"
- 安全-scanner: "⚠️ SQL injection vulnerability in line 12"
- 测试-coverage: "Coverage is 45%, missing 测试 for 错误 cases"
- docs-checker: "Missing docstring for function `process_data`"

### 示例 2: Multi-Language Support

```json
{
  "broadcast": {
    "strategy": "sequential",
    "+15555550123": ["detect-language", "translator-en", "translator-de"]
  },
  "agents": {
    "list": [
      { "id": "detect-language", "workspace": "~/agents/lang-detect" },
      { "id": "translator-en", "workspace": "~/agents/translate-en" },
      { "id": "translator-de", "workspace": "~/agents/translate-de" }
    ]
  }
}
```

## API 参考

### 配置 模式

```typescript
interface MoltbotConfig {
  broadcast?: {
    strategy?: "parallel" | "sequential";
    [peerId: string]: string[];
  };
}
```

### Fields

- `strategy` (可选): 如何 进程 代理
  - `"parallel"` (默认): All 代理 进程 simultaneously
  - `"sequential"`: 代理 进程 in 数组 order
  
- `[peerId]`: WhatsApp group JID, E.164 数字, or other peer ID
  - 值: 数组 of 代理 IDs that should 进程 消息

## 限制

1. **Max 代理:** No hard limit, but 10+ 代理 may be slow
2. **Shared 上下文:** 代理 don't 参见 each other's 响应 (by design)
3. **消息 ordering:** Parallel 响应 may arrive in any order
4. **Rate limits:** All 代理 count toward WhatsApp rate limits

## Future Enhancements

Planned 功能:
- [ ] Shared 上下文 mode (代理 参见 each other's 响应)
- [ ] 代理 coordination (代理 can signal each other)
- [ ] Dynamic 代理 selection (choose 代理 based on 消息 content)
- [ ] 代理 priorities (some 代理 respond before others)

## 另请参阅

- [Multi-代理 配置](/multi-代理-沙箱-工具)
- [Routing 配置](/concepts/渠道-routing)
- [会话 Management](/concepts/会话)
