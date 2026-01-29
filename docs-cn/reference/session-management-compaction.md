---
summary: "深入探讨：会话存储 + 转录、生命周期和（自动）压缩内部机制"
read_when:
  - 您需要调试会话 id、转录 JSONL 或 sessions.json 字段
  - 您正在更改自动压缩行为或添加"预压缩"清理
  - 您想要实现内存刷新或静默系统轮次
---
# 会话管理与压缩（深入探讨）

本文档解释了 Moltbot 端到端如何管理会话：

- **会话路由**（入站消息如何映射到 `sessionKey`）
- **会话存储**（`sessions.json`）及其跟踪内容
- **转录持久性**（`*.jsonl`）及其结构
- **转录卫生**（提供商运行前的特定修复）
- **上下文限制**（上下文窗口与跟踪的令牌）
- **压缩**（手动 + 自动压缩）以及在何处挂钩预压缩工作
- **静默清理**（例如，不应该产生用户可见输出的内存写入）

如果您想先进行更高级别的概述，请从以下开始：
- [/concepts/session](/concepts/session)
- [/concepts/compaction](/concepts/compaction)
- [/concepts/session-pruning](/concepts/session-pruning)
- [/reference/transcript-hygiene](/reference/transcript-hygiene)

---

## 真实来源：网关

Moltbot 围绕拥有会话状态的单一**网关进程**设计。

- UI（macOS 应用、Web 控制 UI、TUI）应该查询网关以获取会话列表和令牌计数。
- 在远程模式下，会话文件位于远程主机上；"检查您的本地 Mac 文件"不会反映网关正在使用的内容。

---

## 两个持久层

Moltbot 以两层持久化会话：

1) **会话存储（`sessions.json`）**
   - 键/值映射：`sessionKey -> SessionEntry`
   - 小型、可变、安全编辑（或删除条目）
   - 跟踪会话元数据（当前会话 id、最后活动、切换、令牌计数器等）

2) **转录（`<sessionId>.jsonl`）**
   - 具有树结构的仅追加转录（条目具有 `id` + `parentId`）
   - 存储实际对话 + 工具调用 + 压缩摘要
   - 用于为未来的轮次重建模型上下文

---

## 磁盘位置

每个代理，在网关主机上：

- 存储：`~/.clawdbot/agents/<agentId>/sessions/sessions.json`
- 转录：`~/.clawdbot/agents/<agentId>/sessions/<sessionId>.jsonl`
  - Telegram 主题会话：`.../<sessionId>-topic-<threadId>.jsonl`

Moltbot 通过 `src/config/sessions.ts` 解析这些位置。

---

## 会话密钥（`sessionKey`）

`sessionKey` 识别您所在的*哪个对话桶*（路由 + 隔离）。

常见模式：

- 主/直接聊天（每个代理）：`agent:<agentId>:<mainKey>`（默认 `main`）
- 组：`agent:<agentId>:<channel>:group:<id>`
- 房间/频道（Discord/Slack）：`agent:<agentId>:<channel>:channel:<id>` 或 `...:room:<id>`
- Cron：`cron:<job.id>`
- Webhook：`hook:<uuid>`（除非被覆盖）

规范规则记录在 [/concepts/session](/concepts/session)。

---

## 会话 id（`sessionId`）

每个 `sessionKey` 指向当前 `sessionId`（继续对话的转录文件）。

经验法则：
- **重置**（`/new`、`/reset`）为该 `sessionKey` 创建新的 `sessionId`。
- **每日重置**（默认网关主机上的本地时间凌晨 4 点）在重置边界后的下一条消息创建新的 `sessionId`。
- **空闲过期**（`session.reset.idleMinutes` 或传统的 `session.idleMinutes`）在空闲窗口后消息到达时创建新的 `sessionId`。当同时配置每日 + 空闲时，先过期的获胜。

实现细节：决策发生在 `src/auto-reply/reply/session.ts` 中的 `initSessionState()` 内。

---

## 会话存储模式（`sessions.json`）

存储的值类型是 `src/config/sessions.ts` 中的 `SessionEntry`。

关键字段（非详尽）：

- `sessionId`：当前转录 id（文件名派生自此项，除非设置了 `sessionFile`）
- `updatedAt`：最后活动时间戳
- `sessionFile`：可选的显式转录路径覆盖
- `chatType`：`direct | group | room`（帮助 UI 和发送策略）
- `provider`、`subject`、`room`、`space`、`displayName`：用于组/频道标签的元数据
- 切换：
  - `thinkingLevel`、`verboseLevel`、`reasoningLevel`、`elevatedLevel`
  - `sendPolicy`（每会话覆盖）
- 模型选择：
  - `providerOverride`、`modelOverride`、`authProfileOverride`
- 令牌计数器（尽力而为/依赖于提供商）：
  - `inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`
- `compactionCount`：此会话密钥的自动压缩完成次数
- `memoryFlushAt`：上次预压缩内存刷新的时间戳
- `memoryFlushCompactionCount`：上次刷新运行时的压缩计数

存储可以安全编辑，但网关是权威：它可能会随着会话的运行重写或重新补充条目。

---

## 转录结构（`*.jsonl`）

转录由 `@mariozechner/pi-coding-agent` 的 `SessionManager` 管理。

文件是 JSONL：
- 第一行：会话标头（`type: "session"`，包括 `id`、`cwd`、`timestamp`、可选的 `parentSession`）
- 然后：具有 `id` + `parentId` 的会话条目（树）

值得注意的条目类型：
- `message`：用户/助手/工具Result 消息
- `custom_message`：扩展注入的消息，*确实*进入模型上下文（可以对 UI 隐藏）
- `custom`：不*进入*模型上下文的扩展状态
- `compaction`：持久压缩摘要，具有 `firstKeptEntryId` 和 `tokensBefore`
- `branch_summary`：导航树枝条目时的持久摘要

Moltbot 故意**不**"修复"转录；网关使用 `SessionManager` 来读取/写入它们。

---

## 上下文窗口与跟踪的令牌

两个不同的概念很重要：

1) **模型上下文窗口**：每个模型的硬性上限（模型可见的令牌）
2) **会话存储计数器**：写入 `sessions.json` 的滚动统计信息（用于 /status 和仪表板）

如果您正在调整限制：
- 上下文窗口来自模型目录（并且可以通过配置覆盖）
- 存储中的 `contextTokens` 是运行时估算/报告值；不要将其视为严格保证。

有关更多信息，请参阅 [/token-use](/token-use)。

---

## 压缩：它是什么

压缩将较早的对话总结为转录中的持久 `compaction` 条目，并保持最近的消息完整。

压缩后，未来的轮次看到：
- 压缩摘要
- `firstKeptEntryId` 之后的消息

压缩是**持久的**（不像会话修剪）。请参阅 [/concepts/session-pruning](/concepts/session-pruning)。

---

## 自动压缩何时发生（Pi 运行时）

在嵌入式 Pi 代理中，自动压缩在两种情况下触发：

1) **溢出恢复**：模型返回上下文溢出错误 → 压缩 → 重试。
2) **阈值维护**：成功轮次后，当：

`contextTokens > contextWindow - reserveTokens`

其中：
- `contextWindow` 是模型的上下文窗口
- `reserveTokens` 是为提示 + 下一个模型输出保留的缓冲空间

这些是 Pi 运行时语义（Moltbot 消费事件，但 Pi 决定何时压缩）。

---

## 压缩设置（`reserveTokens`、`keepRecentTokens`）

Pi 的压缩设置位于 Pi 设置中：

```json5
{
  compaction: {
    enabled: true,
    reserveTokens: 16384,
    keepRecentTokens: 20000
  }
}
```

Moltbot 还为嵌入式运行强制执行安全底线：

- 如果 `compaction.reserveTokens < reserveTokensFloor`，Moltbot 会提高它。
- 默认底线是 `20000` 令牌。
- 设置 `agents.defaults.compaction.reserveTokensFloor: 0` 以禁用底线。
- 如果它已经更高，Moltbot 将保持原样。

原因：为多轮"清理"（如内存写入）留出足够的缓冲空间，然后压缩变得不可避免。

实现：`src/agents/pi-settings.ts` 中的 `ensurePiCompactionReserveTokens()`
（从 `src/agents/pi-embedded-runner.ts` 调用）。

---

## 用户可见表面

您可以通过以下方式观察压缩和会话状态：

- `/status`（在任何聊天会话中）
- `moltbot status` (CLI)
- `moltbot sessions` / `sessions --json`
- 详细模式：`🧹 自动压缩完成` + 压缩计数

---

## 静默清理（`NO_REPLY`）

Moltbot 支持"静默"轮次，用于不应显示中间输出的后台任务。

约定：
- 助手以其输出开头使用 `NO_REPLY` 表示"不向用户传递回复"。
- Moltbot 在传递层剥离/抑制此内容。

截至 `2026.1.10`，Moltbot 还会在部分块以 `NO_REPLY` 开头时抑制**草稿/输入流式传输**，因此静默操作不会在轮次中途泄漏部分输出。

---

## 预压缩"内存刷新"（已实现）

目标：在自动压缩发生之前，运行一个静默的代理轮次，将持久状态
写入磁盘（例如代理工作区中的 `memory/YYYY-MM-DD.md`），这样压缩就
无法删除关键上下文。

Moltbot 使用**预阈值刷新**方法：

1. 监控会话上下文使用情况。
2. 当它越过"软阈值"（低于 Pi 的压缩阈值）时，向代理运行静默的
   "立即写入内存"指令。
3. 使用 `NO_REPLY`，以便用户什么也看不到。

配置（`agents.defaults.compaction.memoryFlush`）：
- `enabled`（默认：`true`）
- `softThresholdTokens`（默认：`4000`）
- `prompt`（刷新轮次的用户消息）
- `systemPrompt`（为刷新轮次附加的额外系统提示）

注意：
- 默认提示/系统提示包括 `NO_REPLY` 提示以抑制传递。
- 刷新每压缩周期运行一次（在 `sessions.json` 中跟踪）。
- 刷新仅适用于嵌入式 Pi 会话（CLI 后端会跳过它）。
- 当会话工作区为只读（`workspaceAccess: "ro"` 或 `"none"`）时跳过刷新。
- 有关工作区文件布局和写入模式，请参阅 [内存](/concepts/memory)。

Pi 还在扩展 API 中公开了 `session_before_compact` 挂钩，但 Moltbot 的
刷新逻辑目前位于网关端。

---

## 故障排除清单

- 会话密钥错误？从 [/concepts/session](/concepts/session) 开始，并通过 `/status` 确认 `sessionKey`。
- 存储与转录不匹配？从 `moltbot status` 确认网关主机和存储路径。
- 压缩垃圾？检查：
  - 模型上下文窗口（太小）
  - 压缩设置（`reserveTokens` 太高，对于模型窗口可能导致更早的压缩）
  - 工具结果膨胀：启用/调整会话修剪
- 静默轮次泄漏？确认回复以 `NO_REPLY`（确切令牌）开头，并且您使用的是包含流抑制修复的构建。
