---
summary: "Agent 运行时（嵌入式 p-mono）、工作区合约和会话引导"
read_when:
  - 更改 agent 运行时、工作区引导或会话行为
---
# Agent 运行时 🤖

Moltbot 运行单个嵌入式 agent 运行时，派生自 **p-mono**。

## 工作区（必需）

Moltbot 使用单个 agent 工作区目录（`agents.defaults.workspace`）作为 agent 的**唯一**工作目录（`cwd`），用于工具和上下文。

推荐：使用 `moltbot setup` 创建 `~/.clawdbot/moltbot.json`（如果缺失）并初始化工作区文件。

完整工作区布局 + 备份指南：[Agent workspace](/concepts/agent-workspace)

如果启用了 `agents.defaults.sandbox`，非 main 会话可以使用 `agents.defaults.sandbox.workspaceRoot` 下的每个会话工作区来覆盖此设置（参见 [Gateway configuration](/gateway/configuration)）。

## 引导文件（注入）

在 `agents.defaults.workspace` 内，Moltbot 期望这些用户可编辑的文件：
- `AGENTS.md` — 操作说明 + "记忆"
- `SOUL.md` — 人格、边界、语气
- `TOOLS.md` — 用户维护的工具注释（例如 `imsg`、`sag`、约定）
- `BOOTSTRAP.md` — 一次性首次运行仪式（完成后删除）
- `IDENTITY.md` — agent 名称/氛围/表情符号
- `USER.md` — 用户档案 + 首选地址

在新会话的第一轮，Moltbot 将这些文件的内容直接注入到 agent 上下文中。

空文件会被跳过。大文件会被修剪并截断，带有标记，以保持提示精简（读取文件以获取完整内容）。

如果文件缺失，Moltbot 会注入单个"缺失文件"标记行（并且 `moltbot setup` 将创建安全的默认模板）。

`BOOTSTRAP.md` 仅在**全新工作区**（不存在其他引导文件）时创建。如果在完成仪式后删除它，不应在后续重启时重新创建。

要完全禁用引导文件创建（对于预播种的工作区），设置：

```json5
{ agent: { skipBootstrap: true } }
```

## 内置工具

核心工具（read/exec/edit/write 和相关系统工具）始终可用，受工具策略约束。`apply_patch` 是可选的，由 `tools.exec.applyPatch` 控制。`TOOLS.md` **不**控制哪些工具存在；它只是关于*您*希望如何使用它们的指导。

## Skills

Moltbot 从三个位置加载 skills（工作区在名称冲突时获胜）：
- 捆绑（随安装附带）
- 托管/本地：`~/.clawdbot/skills`
- 工作区：`<workspace>/skills`

Skills 可以通过配置/环境进行限制（参见 [Gateway configuration](/gateway/configuration) 中的 `skills`）。

## p-mono 集成

Moltbot 重用 p-mono 代码库的部分（模型/工具），但**会话管理、发现和工具连接由 Moltbot 拥有**。

- 没有 p-coding agent 运行时。
- 不查询 `~/.pi/agent` 或 `<workspace>/.pi` 设置。

## 会话

会话记录以 JSONL 格式存储在：
- `~/.clawdbot/agents/<agentId>/sessions/<SessionId>.jsonl`

会话 ID 是稳定的，由 Moltbot 选择。不读取旧的 Pi/Tau 会话文件夹。

## 流式传输时的引导

当队列模式为 `steer` 时，入站消息会注入到当前运行中。队列在**每次工具调用后**检查；如果存在排队消息，当前助手消息的剩余工具调用将被跳过（错误工具结果为"由于排队的用户消息而跳过"），然后在下一个助手响应之前注入排队的用户消息。

当队列模式为 `followup` 或 `collect` 时，入站消息将被保留，直到当前轮次结束，然后使用排队负载启动新的 agent 轮次。有关模式 + 防抖/上限行为，请参阅 [Queue](/concepts/queue)。

块流式传输在完成时立即发送已完成的助手块；默认**关闭**（`agents.defaults.blockStreamingDefault: "off"`）。
通过 `agents.defaults.blockStreamingBreak` 调整边界（`text_end` vs `message_end`；默认为 text_end）。
使用 `agents.defaults.blockStreamingChunk` 控制软块分块（默认为 800-1200 字符；优先选择段落分隔符，然后是换行符；句子最后）。
使用 `agents.defaults.blockStreamingCoalesce` 合并流式块，以减少单行垃圾邮件（发送前基于空闲的合并）。
非 Telegram 渠道需要显式设置 `*.blockStreaming: true` 来启用块回复。
详细工具摘要在工具启动时发出（无防抖）；Control UI 在可用时通过 agent 事件流式传输工具输出。
更多细节：[Streaming + chunking](/concepts/streaming)。

## 模型引用

配置中的模型引用（例如 `agents.defaults.model` 和 `agents.defaults.models`）通过在**第一个** `/` 处拆分来解析。

- 配置模型时使用 `provider/model`。
- 如果模型 ID 本身包含 `/`（OpenRouter 风格），请包含提供者前缀（例如：`openrouter/moonshotai/kimi-k2`）。
- 如果省略提供者，Moltbot 将输入视为别名或**默认提供者**的模型（仅当模型 ID 中没有 `/` 时才有效）。

## 配置（最小）

至少设置：
- `agents.defaults.workspace`
- `channels.whatsapp.allowFrom`（强烈推荐）

---

*下一步：[Group Chats](/concepts/group-messages)* 🦞
