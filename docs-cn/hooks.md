---
summary: "Hooks：用于命令和生命周期事件的事件驱动自动化"
read_when:
  - 您需要为 /new、/reset、/stop 和 agent 生命周期事件实现事件驱动的自动化
  - 您想要构建、安装或调试 hooks
---
# Hooks

Hooks 提供了一个可扩展的事件驱动系统，用于自动化响应 agent 命令和事件的操作。Hooks 会从目录中自动发现，并可以通过 CLI 命令进行管理，类似于 Moltbot 中 skills 的工作方式。

## 快速入门

Hooks 是在事件发生时运行的小脚本。有两种类型：

- **Hooks**（本页）：在 Gateway 内部当 agent 事件触发时运行，如 `/new`、`/reset`、`/stop` 或生命周期事件。
- **Webhooks**：外部 HTTP webhooks，允许其他系统在 Moltbot 中触发工作。参见 [Webhook Hooks](/automation/webhook) 或使用 `moltbot webhooks` 查看 Gmail 辅助命令。

Hooks 也可以打包在 plugins 内部；参见 [Plugins](/plugin#plugin-hooks)。

常见用途：
- 在重置会话时保存内存快照
- 保留命令的审计跟踪以便故障排除或合规
- 在会话开始或结束时触发后续自动化
- 当事件触发时将文件写入 agent 工作区或调用外部 API

如果您能编写一个小的 TypeScript 函数，就可以编写一个 hook。Hooks 会自动被发现，您可以通过 CLI 启用或禁用它们。

## 概述

hooks 系统允许您：
- 在发出 `/new` 时将会话上下文保存到内存
- 记录所有命令以进行审计
- 在 agent 生命周期事件上触发自定义自动化
- 在不修改核心代码的情况下扩展 Moltbot 的行为

## 入门指南

### 内置 Hooks

Moltbot 附带了四个会自动发现的内置 hooks：

- **💾 session-memory**：当您发出 `/new` 时，将会话上下文保存到您的 agent 工作区（默认为 `~/clawd/memory/`）
- **📝 command-logger**：将所有命令事件记录到 `~/.clawdbot/logs/commands.log`
- **🚀 boot-md**：在 gateway 启动时运行 `BOOT.md`（需要启用内部 hooks）
- **😈 soul-evil**：在清除窗口期间或随机机会下，将注入的 `SOUL.md` 内容与 `SOUL_EVIL.md` 交换

列出可用的 hooks：

```bash
moltbot hooks list
```

启用一个 hook：

```bash
moltbot hooks enable session-memory
```

检查 hook 状态：

```bash
moltbot hooks check
```

获取详细信息：

```bash
moltbot hooks info session-memory
```

### 入门配置

在入门配置期间（`moltbot onboard`），系统会提示您启用推荐的 hooks。向导会自动发现有资格的 hooks 并呈现供选择。

## Hook 发现

Hooks 会从三个目录自动发现（按优先级顺序）：

1. **工作区 hooks**：`<workspace>/hooks/`（每个 agent，最高优先级）
2. **托管 hooks**：`~/.clawdbot/hooks/`（用户安装的，在工作区之间共享）
3. **内置 hooks**：`<moltbot>/dist/hooks/bundled/`（随 Moltbot 附带）

托管的 hook 目录可以是**单个 hook**或**hook 包**（包目录）。

每个 hook 是一个包含以下内容的目录：

```
my-hook/
├── HOOK.md          # 元数据 + 文档
└── handler.ts       # 处理程序实现
```

## Hook 包（npm/archives）

Hook 包是标准的 npm 包，通过 `package.json` 中的 `moltbot.hooks` 导出一个或多个 hooks。使用以下命令安装：

```bash
moltbot hooks install <path-or-spec>
```

示例 `package.json`：

```json
{
  "name": "@acme/my-hooks",
  "version": "0.1.0",
  "moltbot": {
    "hooks": ["./hooks/my-hook", "./hooks/other-hook"]
  }
}
```

每个条目指向一个包含 `HOOK.md` 和 `handler.ts`（或 `index.ts`）的 hook 目录。Hook 包可以附带依赖项；它们将被安装在 `~/.clawdbot/hooks/<id>` 下。

## Hook 结构

### HOOK.md 格式

`HOOK.md` 文件包含 YAML frontmatter 中的元数据加上 Markdown 文档：

```markdown
---
name: my-hook
description: "对这个 hook 功能的简短描述"
homepage: https://docs.molt.bot/hooks#my-hook
metadata: {"moltbot":{"emoji":"🔗","events":["command:new"],"requires":{"bins":["node"]}}}
---

# My Hook

详细文档写在这里...

## 功能

- 监听 `/new` 命令
- 执行某些操作
- 记录结果

## 要求

- 必须安装 Node.js

## 配置

不需要配置。
```

### 元数据字段

`metadata.moltbot` 对象支持：

- **`emoji`**：CLI 的显示 emoji（例如 `"💾"`）
- **`events`**：要监听的事件数组（例如 `["command:new", "command:reset"]`）
- **`export`**：要使用的命名导出（默认为 `"default"`）
- **`homepage`**：文档 URL
- **`requires`**：可选要求
  - **`bins`**：PATH 上所需的二进制文件（例如 `["git", "node"]`）
  - **`anyBins`**：这些二进制文件中至少必须存在一个
  - **`env`**：所需的环境变量
  - **`config`**：所需的配置路径（例如 `["workspace.dir"]`）
  - **`os`**：所需的平台（例如 `["darwin", "linux"]`）
- **`always`**：绕过资格检查（布尔值）
- **`install`**：安装方法（对于内置 hooks：`[{"id":"bundled","kind":"bundled"}]`）

### 处理程序实现

`handler.ts` 文件导出一个 `HookHandler` 函数：

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const myHandler: HookHandler = async (event) => {
  // 仅在 'new' 命令时触发
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log(`[my-hook] New command triggered`);
  console.log(`  Session: ${event.sessionKey}`);
  console.log(`  Timestamp: ${event.timestamp.toISOString()}`);

  // 您的自定义逻辑写在这里

  // 可选地向用户发送消息
  event.messages.push('✨ My hook executed!');
};

export default myHandler;
```

#### 事件上下文

每个事件包括：

```typescript
{
  type: 'command' | 'session' | 'agent' | 'gateway',
  action: string,              // 例如 'new'、'reset'、'stop'
  sessionKey: string,          // 会话标识符
  timestamp: Date,             // 事件发生的时间
  messages: string[],          // 在这里推送消息以发送给用户
  context: {
    sessionEntry?: SessionEntry,
    sessionId?: string,
    sessionFile?: string,
    commandSource?: string,    // 例如 'whatsapp'、'telegram'
    senderId?: string,
    workspaceDir?: string,
    bootstrapFiles?: WorkspaceBootstrapFile[],
    cfg?: MoltbotConfig
  }
}
```

## 事件类型

### 命令事件

当发出 agent 命令时触发：

- **`command`**：所有命令事件（通用监听器）
- **`command:new`**：当发出 `/new` 命令时
- **`command:reset`**：当发出 `/reset` 命令时
- **`command:stop`**：当发出 `/stop` 命令时

### Agent 事件

- **`agent:bootstrap`**：在注入工作区引导文件之前（hooks 可以改变 `context.bootstrapFiles`）

### Gateway 事件

在 gateway 启动时触发：

- **`gateway:startup`**：在通道启动和 hooks 加载之后

### 工具结果 Hooks（Plugin API）

这些 hooks 不是事件流监听器；它们允许 plugins 在 Moltbot 持久化工具结果之前同步调整它们。

- **`tool_result_persist`**：在将工具结果写入会话记录之前转换它们。必须是同步的；返回更新的工具结果负载或 `undefined` 以保持原样。参见 [Agent Loop](/concepts/agent-loop)。

### 未来事件

计划中的事件类型：

- **`session:start`**：当新会话开始时
- **`session:end`**：当会话结束时
- **`agent:error`**：当 agent 遇到错误时
- **`message:sent`**：当消息发送时
- **`message:received`**：当消息接收时

## 创建自定义 Hooks

### 1. 选择位置

- **工作区 hooks**（`<workspace>/hooks/`）：每个 agent，最高优先级
- **托管 hooks**（`~/.clawdbot/hooks/`）：在工作区之间共享

### 2. 创建目录结构

```bash
mkdir -p ~/.clawdbot/hooks/my-hook
cd ~/.clawdbot/hooks/my-hook
```

### 3. 创建 HOOK.md

```markdown
---
name: my-hook
description: "做些有用的事情"
metadata: {"moltbot":{"emoji":"🎯","events":["command:new"]}}
---

# My Custom Hook

这个 hook 在您发出 `/new` 时做一些有用的事情。
```

### 4. 创建 handler.ts

```typescript
import type { HookHandler } from '../../src/hooks/hooks.js';

const handler: HookHandler = async (event) => {
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  console.log('[my-hook] Running!');
  // 您的逻辑写在这里
};

export default handler;
```

### 5. 启用和测试

```bash
# 验证 hook 已被发现
moltbot hooks list

# 启用它
moltbot hooks enable my-hook

# 重启您的 gateway 进程（macOS 上的菜单栏应用重启，或重启您的开发进程）

# 触发事件
# 通过您的消息渠道发送 /new
```

## 配置

### 新配置格式（推荐）

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

### 每个 Hook 的配置

Hooks 可以有自定义配置：

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

### 额外目录

从其他目录加载 hooks：

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

### 旧配置格式（仍然支持）

旧的配置格式仍然可以向后兼容：

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

**迁移**：对于新的 hooks，请使用新的基于发现的系统。旧的处理程序在基于目录的 hooks 之后加载。

## CLI 命令

### 列出 Hooks

```bash
# 列出所有 hooks
moltbot hooks list

# 仅显示有资格的 hooks
moltbot hooks list --eligible

# 详细输出（显示缺失的要求）
moltbot hooks list --verbose

# JSON 输出
moltbot hooks list --json
```

### Hook 信息

```bash
# 显示有关 hook 的详细信息
moltbot hooks info session-memory

# JSON 输出
moltbot hooks info session-memory --json
```

### 检查资格

```bash
# 显示资格摘要
moltbot hooks check

# JSON 输出
moltbot hooks check --json
```

### 启用/禁用

```bash
# 启用一个 hook
moltbot hooks enable session-memory

# 禁用一个 hook
moltbot hooks disable command-logger
```

## 内置 Hooks

### session-memory

当您发出 `/new` 时，将会话上下文保存到内存。

**事件**：`command:new`

**要求**：必须配置 `workspace.dir`

**输出**：`<workspace>/memory/YYYY-MM-DD-slug.md`（默认为 `~/clawd`）

**功能**：
1. 使用重置前的会话条目来定位正确的记录
2. 提取对话的最后 15 行
3. 使用 LLM 生成描述性的文件名 slug
4. 将会话元数据保存到带日期的内存文件

**示例输出**：

```markdown
# Session: 2026-01-16 14:30:00 UTC

- **Session Key**: agent:main:main
- **Session ID**: abc123def456
- **Source**: telegram
```

**文件名示例**：
- `2026-01-16-vendor-pitch.md`
- `2026-01-16-api-design.md`
- `2026-01-16-1430.md`（如果 slug 生成失败的后备时间戳）

**启用**：

```bash
moltbot hooks enable session-memory
```

### command-logger

将所有命令事件记录到集中的审计文件。

**事件**：`command`

**要求**：无

**输出**：`~/.clawdbot/logs/commands.log`

**功能**：
1. 捕获事件详细信息（命令操作、时间戳、会话键、发送者 ID、来源）
2. 以 JSONL 格式追加到日志文件
3. 在后台静默运行

**示例日志条目**：

```jsonl
{"timestamp":"2026-01-16T14:30:00.000Z","action":"new","sessionKey":"agent:main:main","senderId":"+1234567890","source":"telegram"}
{"timestamp":"2026-01-16T15:45:22.000Z","action":"stop","sessionKey":"agent:main:main","senderId":"user@example.com","source":"whatsapp"}
```

**查看日志**：

```bash
# 查看最近的命令
tail -n 20 ~/.clawdbot/logs/commands.log

# 使用 jq 美化输出
cat ~/.clawdbot/logs/commands.log | jq .

# 按操作过滤
grep '"action":"new"' ~/.clawdbot/logs/commands.log | jq .
```

**启用**：

```bash
moltbot hooks enable command-logger
```

### soul-evil

在清除窗口期间或随机机会下，将注入的 `SOUL.md` 内容与 `SOUL_EVIL.md` 交换。

**事件**：`agent:bootstrap`

**文档**：[SOUL Evil Hook](/hooks/soul-evil)

**输出**：不写入文件；交换仅在内存中进行。

**启用**：

```bash
moltbot hooks enable soul-evil
```

**配置**：

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

在 gateway 启动时运行 `BOOT.md`（在通道启动之后）。
必须启用内部 hooks 才能运行此功能。

**事件**：`gateway:startup`

**要求**：必须配置 `workspace.dir`

**功能**：
1. 从您的工作区读取 `BOOT.md`
2. 通过 agent 运行器运行指令
3. 通过消息工具发送任何请求的出站消息

**启用**：

```bash
moltbot hooks enable boot-md
```

## 最佳实践

### 保持处理程序快速

Hooks 在命令处理期间运行。保持它们轻量级：

```typescript
// ✓ 好 - 异步工作，立即返回
const handler: HookHandler = async (event) => {
  void processInBackground(event); // 即发即弃
};

// ✗ 坏 - 阻塞命令处理
const handler: HookHandler = async (event) => {
  await slowDatabaseQuery(event);
  await evenSlowerAPICall(event);
};
```

### 优雅地处理错误

始终包装有风险的操作：

```typescript
const handler: HookHandler = async (event) => {
  try {
    await riskyOperation(event);
  } catch (err) {
    console.error('[my-handler] Failed:', err instanceof Error ? err.message : String(err));
    // 不要抛出 - 让其他处理程序运行
  }
};
```

### 尽早过滤事件

如果事件不相关，则提前返回：

```typescript
const handler: HookHandler = async (event) => {
  // 仅处理 'new' 命令
  if (event.type !== 'command' || event.action !== 'new') {
    return;
  }

  // 您的逻辑写在这里
};
```

### 使用特定的事件键

尽可能在元数据中指定确切的事件：

```yaml
metadata: {"moltbot":{"events":["command:new"]}}  # 具体
```

而不是：

```yaml
metadata: {"moltbot":{"events":["command"]}}      # 通用 - 更多开销
```

## 调试

### 启用 Hook 日志记录

Gateway 在启动时记录 hook 加载情况：

```
Registered hook: session-memory -> command:new
Registered hook: command-logger -> command
Registered hook: boot-md -> gateway:startup
```

### 检查发现

列出所有已发现的 hooks：

```bash
moltbot hooks list --verbose
```

### 检查注册

在您的处理程序中，记录调用时间：

```typescript
const handler: HookHandler = async (event) => {
  console.log('[my-handler] Triggered:', event.type, event.action);
  // 您的逻辑
};
```

### 验证资格

检查 hook 为什么没有资格：

```bash
moltbot hooks info my-hook
```

在输出中查找缺失的要求。

## 测试

### Gateway 日志

监控 gateway 日志以查看 hook 执行：

```bash
# macOS
./scripts/clawlog.sh -f

# 其他平台
tail -f ~/.clawdbot/gateway.log
```

### 直接测试 Hooks

单独测试您的处理程序：

```typescript
import { test } from 'vitest';
import { createHookEvent } from './src/hooks/hooks.js';
import myHandler from './hooks/my-hook/handler.js';

test('my handler works', async () => {
  const event = createHookEvent('command', 'new', 'test-session', {
    foo: 'bar'
  });

  await myHandler(event);

  // 断言副作用
});
```

## 架构

### 核心组件

- **`src/hooks/types.ts`**：类型定义
- **`src/hooks/workspace.ts`**：目录扫描和加载
- **`src/hooks/frontmatter.ts`**：HOOK.md 元数据解析
- **`src/hooks/config.ts`**：资格检查
- **`src/hooks/hooks-status.ts`**：状态报告
- **`src/hooks/loader.ts`**：动态模块加载器
- **`src/cli/hooks-cli.ts`**：CLI 命令
- **`src/gateway/server-startup.ts`**：在 gateway 启动时加载 hooks
- **`src/auto-reply/reply/commands-core.ts`**：触发命令事件

### 发现流程

```
Gateway 启动
    ↓
扫描目录（工作区 → 托管 → 内置）
    ↓
解析 HOOK.md 文件
    ↓
检查资格（bins、env、config、os）
    ↓
从有资格的 hooks 加载处理程序
    ↓
为事件注册处理程序
```

### 事件流程

```
用户发送 /new
    ↓
命令验证
    ↓
创建 hook 事件
    ↓
触发 hook（所有已注册的处理程序）
    ↓
命令处理继续
    ↓
会话重置
```

## 故障排除

### Hook 未被发现

1. 检查目录结构：
   ```bash
   ls -la ~/.clawdbot/hooks/my-hook/
   # 应该显示：HOOK.md、handler.ts
   ```

2. 验证 HOOK.md 格式：
   ```bash
   cat ~/.clawdbot/hooks/my-hook/HOOK.md
   # 应该有 YAML frontmatter 包含 name 和 metadata
   ```

3. 列出所有已发现的 hooks：
   ```bash
   moltbot hooks list
   ```

### Hook 没有资格

检查要求：

```bash
moltbot hooks info my-hook
```

查找缺失的：
- 二进制文件（检查 PATH）
- 环境变量
- 配置值
- 操作系统兼容性

### Hook 未执行

1. 验证 hook 已启用：
   ```bash
   moltbot hooks list
   # 应该在启用的 hooks 旁边显示 ✓
   ```

2. 重启您的 gateway 进程以便 hooks 重新加载。

3. 检查 gateway 日志中的错误：
   ```bash
   ./scripts/clawlog.sh | grep hook
   ```

### 处理程序错误

检查 TypeScript/import 错误：

```bash
# 直接测试导入
node -e "import('./path/to/handler.ts').then(console.log)"
```

## 迁移指南

### 从旧配置迁移到发现

**之前**：

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

**之后**：

1. 创建 hook 目录：
   ```bash
   mkdir -p ~/.clawdbot/hooks/my-hook
   mv ./hooks/handlers/my-handler.ts ~/.clawdbot/hooks/my-hook/handler.ts
   ```

2. 创建 HOOK.md：
   ```markdown
   ---
   name: my-hook
   description: "My custom hook"
   metadata: {"moltbot":{"emoji":"🎯","events":["command:new"]}}
   ---

   # My Hook

   Does something useful.
   ```

3. 更新配置：
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

4. 验证并重启您的 gateway 进程：
   ```bash
   moltbot hooks list
   # 应该显示：🎯 my-hook ✓
   ```

**迁移的好处**：
- 自动发现
- CLI 管理
- 资格检查
- 更好的文档
- 一致的结构

## 另请参阅

- [CLI 参考：hooks](/cli/hooks)
- [内置 Hooks README](https://github.com/moltbot/moltbot/tree/main/src/hooks/bundled)
- [Webhook Hooks](/automation/webhook)
- [配置](/gateway/configuration#hooks)
