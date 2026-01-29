---
summary: "钩子: 事件-driven automation for 命令 and lifecycle 事件"
read_when: 
  - You want event-driven automation for /new, /reset, /stop, and agent lifecycle events
  - You want to build, install, or debug hooks
---
# 钩子

钩子 provide an extensible 事件-driven 系统 for automating 操作 in 响应 to 代理 命令 and 事件. 钩子 are automatically discovered from 目录 and can be managed via CLI 命令, similar to how 技能 work in Moltbot.

## Getting Oriented

钩子 are small 脚本 that run when something happens. There are two kinds:

- **Hooks** (this page): run inside the Gateway when agent events fire, like `/new`, `/reset`, `/stop`, or lifecycle 事件.
- **Webhooks**: external HTTP webhooks that let other systems trigger work in Moltbot. See [Webhook Hooks](/automation/webhook) or use `moltbot webhooks` for Gmail helper 命令.
  
钩子 can also be bundled inside 插件; 参见 [插件](/插件#插件-钩子).

Common uses:
- Save a memory snapshot when you reset a 会话
- Keep an audit trail of 命令 for 故障排除 or compliance
- 触发器 follow-up automation when a 会话 starts or ends
- Write 文件 into the 代理 工作空间 or call external APIs when 事件 fire

If you can write a small TypeScript 函数, you can write a 钩子. 钩子 are discovered automatically, and you enable or disable them via the CLI.

## 概述

The 钩子 系统 allows you to:
- Save session context to memory when `/new` is issued
- 日志 all 命令 for auditing
- 触发器 custom automations on 代理 lifecycle 事件
- Extend Moltbot's 行为 without modifying core code

## 入门指南

### Bundled 钩子

Moltbot ships with four bundled 钩子 that are automatically discovered:

- **💾 session-memory**: Saves session context to your agent workspace (default `~/clawd/memory/`) when you issue `/new`
- **📝 command-logger**: Logs all command events to `~/.clawdbot/logs/commands.log`
- **🚀 boot-md**: Runs `BOOT.md` when the Gateway starts (requires internal 钩子 已启用)
- **😈 soul-evil**: Swaps injected `SOUL.md` content with `SOUL_EVIL.md` during a purge window or by random chance

List available 钩子:

```bash
moltbot hooks list
```

Enable a 钩子:

```bash
moltbot hooks enable session-memory
```

Check 钩子 状态:

```bash
moltbot hooks check
```

Get detailed information:

```bash
moltbot hooks info session-memory
```

### Onboarding

During onboarding (`moltbot onboard`), you'll be prompted to enable recommended 钩子. The wizard automatically discovers eligible 钩子 and presents them for selection.

## 钩子 Discovery

钩子 are automatically discovered from three 目录 (in order of precedence):

1. **Workspace hooks**: `<workspace>/hooks/` (per-代理, highest precedence)
2. **Managed hooks**: `~/.clawdbot/hooks/` (用户-installed, shared across workspaces)
3. **Bundled hooks**: `<moltbot>/dist/hooks/bundled/` (shipped with Moltbot)

Managed 钩子 目录 can be either a **single 钩子** or a **钩子 pack** (包 目录).

Each 钩子 is a 目录 containing:

```
my-hook/
├── HOOK.md          # Metadata + documentation
└── handler.ts       # Handler implementation
```

## 钩子 Packs (npm/archives)

Hook packs are standard npm packages that export one or more hooks via `moltbot.hooks` in
`package.json`. 安装 them with:

```bash
moltbot hooks install <path-or-spec>
```

Example `package.json`:

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "moltbot": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

Each entry points to a hook directory containing `HOOK.md` and `handler.ts` (or `index.ts`).
Hook packs can ship dependencies; they will be installed under `~/.clawdbot/hooks/<id>`.

## 钩子 Structure

### 钩子.md 格式

The `HOOK.md` 文件 contains 元数据 in YAML frontmatter plus Markdown 文档:

```markdown
---
name: my-hook
description: "Short description of what this hook does"
homepage: https://docs.molt.bot/hooks#my-hook
metadata: {"moltbot":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

Detailed documentation goes here...

## What It Does

- Listens for `/new` commands
- Performs some action
- Logs the result

## Requirements

- Node.js must be installed

## Configuration

No configuration needed.
```

### 元数据 Fields

The `metadata.moltbot` 对象 supports:

- **`emoji`**: Display emoji for CLI (e.g., `"💾"`)
- **`events`**: Array of events to listen for (e.g., `["command:new", "command:reset"]`)
- **`export`**: Named export to use (defaults to `"default"`)
- **`homepage`**: 文档 URL
- **`requires`**: 可选 要求
  - **`bins`**: Required binaries on PATH (e.g., `["git", "node"]`)
  - **`anyBins`**: At least one of these binaries must be present
  - **`env`**: 必需 环境 变量
  - **`config`**: Required config paths (e.g., `["workspace.dir"]`)
  - **`os`**: Required platforms (e.g., `["darwin", "linux"]`)
- **`always`**: Bypass eligibility checks (布尔值)
- **`install`**: Installation methods (for bundled hooks: `[{"id":"bundled","kind":"bundled"}]`)

### 处理器 实现

The `handler.ts` file exports a `HookHandler` 函数:

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // Only trigger on 'new' command
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // Your custom logic here

  // Optionally send message to user
  event.messages.push('✨ My hook executed!');
};

export default myHandler;
```

#### 事件 上下文

Each 事件 includes:

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // e.g., 'new', 'reset', 'stop'
  sessionKey: string,          // Session identifier
  timestamp: Date,             // When the event occurred
  messages: string[],          // Push messages here to send to user
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // e.g., 'whatsapp', 'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: MoltbotConfig
  }
}
```

## 事件 Types

### 命令 事件

Triggered when 代理 命令 are issued:

- **`command`**: All 命令 事件 (general 监听器)
- **`command:new`**: When `/new` 命令 is issued
- **`command:reset`**: When `/reset` 命令 is issued
- **`command:stop`**: When `/stop` 命令 is issued

### 代理 事件

- **`agent:bootstrap`**: Before workspace bootstrap files are injected (hooks may mutate `context.bootstrapFiles`)

### Gateway 事件

Triggered when the Gateway starts:

- **`gateway:startup`**: After 渠道 start and 钩子 are loaded

### 工具 结果 钩子 (插件 API)

These 钩子 are not 事件-流 监听器; they let 插件 synchronously adjust 工具 结果 before Moltbot persists them.

- **`tool_result_persist`**: transform tool results before they are written to the session transcript. Must be synchronous; return the updated tool result payload or `undefined` to keep it as-is. 参见 [代理 Loop](/concepts/代理-loop).

### Future 事件

Planned 事件 types:

- **`session:start`**: When a new 会话 begins
- **`session:end`**: When a 会话 ends
- **`agent:error`**: When an 代理 encounters an 错误
- **`message:sent`**: When a 消息 is sent
- **`message:received`**: When a 消息 is received

## Creating Custom 钩子

### 1. Choose Location

- **Workspace hooks** (`<workspace>/hooks/`): Per-代理, highest precedence
- **Managed hooks** (`~/.clawdbot/hooks/`): Shared across workspaces

### 2. Create 目录 Structure

```bash
mkdir -p ~/.clawdbot/hooks/my-hook
cd ~/.clawdbot/hooks/my-hook
```

### 3. Create 钩子.md

```markdown
---
name: my-hook
description: "Does something useful"
metadata: {"moltbot":{"emoji":"🎯","events":["command:new"]}}
---

# My Custom Hook

This hook does something useful when you issue `/new`.
```

### 4. Create 处理器.ts

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // Your logic here
};

export default handler;
```

### 5. Enable and 测试

```bash
# Verify hook is discovered
moltbot hooks list

# Enable it
moltbot hooks enable my-hook

# Restart your gateway process (menu bar app restart on macOS, or restart your dev process)

# Trigger the event
# Send /new via your messaging channel
```

## 配置

### New 配置 格式 (Recommended)

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": { "enabled": true },
        "command-logger": { "enabled": false }
      }
    }
  }
}
```

### Per-钩子 配置

钩子 can have custom 配置:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "my-hook": {
          "enabled": true,
          "env": {
            "MY_CUSTOM_VAR": "value"
          }
        }
      }
    }
  }
}
```

### Extra 目录

Load 钩子 from additional 目录:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "load": {
        "extraDirs": ["/path/to/more/hooks"]
      }
    }
  }
}
```

### Legacy 配置 格式 (Still Supported)

The old 配置 格式 still works for backwards compatibility:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts",
          "export": "default"
        }
      ]
    }
  }
}
```

**迁移**: Use the new discovery-based 系统 for new 钩子. Legacy 处理器 are loaded after 目录-based 钩子.

## CLI 命令

### List 钩子

```bash
# List all hooks
moltbot hooks list

# Show only eligible hooks
moltbot hooks list --eligible

# Verbose output (show missing requirements)
moltbot hooks list --verbose

# JSON output
moltbot hooks list --json
```

### 钩子 Information

```bash
# Show detailed info about a hook
moltbot hooks info session-memory

# JSON output
moltbot hooks info session-memory --json
```

### Check Eligibility

```bash
# Show eligibility summary
moltbot hooks check

# JSON output
moltbot hooks check --json
```

### Enable/Disable

```bash
# Enable a hook
moltbot hooks enable session-memory

# Disable a hook
moltbot hooks disable command-logger
```

## Bundled 钩子

### 会话-memory

Saves session context to memory when you issue `/new`.

**Events**: `command:new`

**Requirements**: `workspace.dir` must be configured

**Output**: `<workspace>/memory/YYYY-MM-DD-slug.md` (defaults to `~/clawd`)

**What it does**:
1. Uses the pre-reset 会话 entry to locate the correct transcript
2. Extracts the last 15 lines of conversation
3. Uses LLM to generate a descriptive filename slug
4. Saves 会话 元数据 to a dated memory 文件

**示例 输出**:

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**Filename 示例**:
- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md` (fallback timestamp if slug generation fails)

**Enable**:

```bash
moltbot hooks enable session-memory
```

### 命令-logger

日志 all 命令 事件 to a centralized audit 文件.

**Events**: `command`

**要求**: None

**Output**: `~/.clawdbot/logs/commands.log`

**What it does**:
1. Captures 事件 details (命令 操作, timestamp, 会话 键, sender ID, source)
2. Appends to 日志 文件 in JSONL 格式
3. Runs silently in the background

**示例 日志 entries**:

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**视图 日志**:

```bash
# View recent commands
tail -n 20 ~/.clawdbot/logs/commands.log

# Pretty-print with jq
cat ~/.clawdbot/logs/commands.log | jq .

# Filter by action
grep '"action":"new"' ~/.clawdbot/logs/commands.log | jq .
```

**Enable**:

```bash
moltbot hooks enable command-logger
```

### soul-evil

Swaps injected `SOUL.md` content with `SOUL_EVIL.md` during a purge window or by random chance.

**Events**: `agent:bootstrap`

**Docs**: [SOUL Evil 钩子](/钩子/soul-evil)

**输出**: No 文件 written; swaps happen in-memory only.

**Enable**:

```bash
moltbot hooks enable soul-evil
```

**配置**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "soul-evil": {
          "enabled": true,
          "file": "SOUL_EVIL.md",
          "chance": 0.1,
          "purge": { "at": "21:00", "duration": "15m" }
        }
      }
    }
  }
}
```

### boot-md

Runs `BOOT.md` when the Gateway starts (after 渠道 start).
Internal 钩子 must be 已启用 for this to run.

**Events**: `gateway:startup`

**Requirements**: `workspace.dir` must be configured

**What it does**:
1. Reads `BOOT.md` from your 工作空间
2. Runs the instructions via the 代理 runner
3. Sends any requested outbound 消息 via the 消息 工具

**Enable**:

```bash
moltbot hooks enable boot-md
```

## 最佳实践

### Keep 处理器 Fast

钩子 run during 命令 processing. Keep them lightweight:

```typescript
// ✓ Good - async work, returns immediately
const handler: HookHandler = async (event) => {
  void processInBackground(event); // Fire and forget
};

// ✗ Bad - blocks command processing
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### Handle 错误 Gracefully

Always wrap risky 操作:

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // Don't throw - let other handlers run
  }
};
```

### 过滤器 事件 Early

Return early if the 事件 isn't relevant:

```typescript
const handler: HookHandler = async (event) => {
  // Only handle 'new' commands
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // Your logic here
};
```

### Use Specific 事件 Keys

Specify exact 事件 in 元数据 when possible:

```yaml
metadata: {"moltbot":{"events":["command:new"]}}  # Specific
```

Rather than:

```yaml
metadata: {"moltbot":{"events":["command"]}}      # General - more overhead
```

## 调试

### Enable 钩子 日志记录

The Gateway 日志 钩子 loading at startup:

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### Check Discovery

List all discovered 钩子:

```bash
moltbot hooks list --verbose
```

### Check Registration

In your 处理器, 日志 when it's called:

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // Your logic
};
```

### Verify Eligibility

Check why a 钩子 isn't eligible:

```bash
moltbot hooks info my-hook
```

Look for missing 要求 in the 输出.

## 测试

### Gateway 日志

Monitor Gateway 日志 to 参见 钩子 execution:

```bash
# macOS
./scripts/clawlog.sh -f

# Other platforms
tail -f ~/.clawdbot/gateway.log
```

### 测试 钩子 Directly

测试 your 处理器 in isolation:

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('my handler works', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // Assert side effects
});
```

## Architecture

### Core Components

- **`src/hooks/types.ts`**: 类型 definitions
- **`src/hooks/workspace.ts`**: 目录 scanning and loading
- **`src/hooks/frontmatter.ts`**: 钩子.md 元数据 parsing
- **`src/hooks/config.ts`**: Eligibility checking
- **`src/hooks/hooks-status.ts`**: 状态 reporting
- **`src/hooks/loader.ts`**: Dynamic 模块 loader
- **`src/cli/hooks-cli.ts`**: CLI 命令
- **`src/gateway/server-startup.ts`**: Loads 钩子 at Gateway start
- **`src/auto-reply/reply/commands-core.ts`**: 触发 命令 事件

### Discovery Flow

```
Gateway startup
    ↓
Scan directories (workspace → managed → bundled)
    ↓
Parse HOOK.md files
    ↓
Check eligibility (bins, env, config, os)
    ↓
Load handlers from eligible hooks
    ↓
Register handlers for events
```

### 事件 Flow

```
User sends /new
    ↓
Command validation
    ↓
Create hook event
    ↓
Trigger hook (all registered handlers)
    ↓
Command processing continues
    ↓
Session reset
```

## 故障排除

### 钩子 Not Discovered

1. Check 目录 structure:
   ```bash
   ls -la ~/.clawdbot/hooks/my-hook/
   # Should show: HOOK.md, handler.ts
   ```

2. Verify 钩子.md 格式:
   ```bash
   cat ~/.clawdbot/hooks/my-hook/HOOK.md
   # Should have YAML frontmatter with name and metadata
   ```

3. List all discovered 钩子:
   ```bash
   moltbot hooks list
   ```

### 钩子 Not Eligible

Check 要求:

```bash
moltbot hooks info my-hook
```

Look for missing:
- Binaries (check 路径)
- 环境 变量
- 配置 values
- OS compatibility

### 钩子 Not Executing

1. Verify 钩子 is 已启用:
   ```bash
   moltbot hooks list
   # Should show ✓ next to enabled hooks
   ```

2. Restart your Gateway 进程 so 钩子 reload.

3. Check Gateway 日志 for 错误:
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### 处理器 错误

Check for TypeScript/import 错误:

```bash
# Test import directly
node -e "import('./path/to/handler.ts').then(console.log)"
```

## 迁移 指南

### From Legacy 配置 to Discovery

**Before**:

```json
{
  "hooks": {
    "internal": {
      "enabled": true,
      "handlers": [
        {
          "event": "command:new",
          "module": "./hooks/handlers/my-handler.ts"
        }
      ]
    }
  }
}
```

**After**:

1. Create 钩子 目录:
   ```bash
   mkdir -p ~/.clawdbot/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.clawdbot/hooks/my-hook/handler.ts
   ```

2. Create 钩子.md:
   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: {"moltbot":{"emoji":"🎯","events":["command:new"]}}
   ---

   # My Hook

   Does something useful.
   ```

3. 更新 配置:
   ```json
   {
     "hooks": {
       "internal": {
         "enabled": true,
         "entries": {
           "my-hook": { "enabled": true }
         }
       }
     }
   }
   ```

4. Verify and restart your Gateway 进程:
   ```bash
   moltbot hooks list
   # Should show: 🎯 my-hook ✓
   ```

**Benefits of 迁移**:
- Automatic discovery
- CLI management
- Eligibility checking
- Better 文档
- Consistent structure

## 另请参阅

- [CLI 参考: 钩子](/cli/钩子)
- [Bundled 钩子 README](https://github.com/moltbot/moltbot/tree/main/src/钩子/bundled)
- [Webhook 钩子](/automation/Webhook)
- [配置](/Gateway/配置#钩子)
