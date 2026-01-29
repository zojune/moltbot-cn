---
summary: "Agent 循环生命周期、流和等待语义"
read_when:
  - 您需要 agent 循环或生命周期事件的精确演练
---
# Agent 循环 (Moltbot)

Agent 循环是 agent 的完整"真实"运行：接收 → 上下文组装 → 模型推理 →
工具执行 → 流式回复 → 持久化。它是将消息转换为操作和最终回复的权威路径，同时保持会话状态一致。

在 Moltbot 中，循环是每个会话的单个序列化运行，当模型思考、调用工具和流式传输输出时，发出生命周期和流事件。本文档解释了该真实循环的端到端连接。

## 入口点
- Gateway RPC：`agent` 和 `agent.wait`。
- CLI：`agent` 命令。

## 工作原理（高级）
1) `agent` RPC 验证参数、解析会话（sessionKey/sessionId）、持久化会话元数据、立即返回 `{ runId, acceptedAt }`。
2) `agentCommand` 运行 agent：
   - 解析模型 + 思考/详细默认值
   - 加载 skills 快照
   - 调用 `runEmbeddedPiAgent`（pi-agent-core 运行时）
   - 如果嵌入式循环未发出，则发出**生命周期结束/错误**
3) `runEmbeddedPiAgent`：
   - 通过每个会话 + 全局队列序列化运行
   - 解析模型 + 认证配置文件并构建 pi 会话
   - 订阅 pi 事件并流式传输助手/工具增量
   - 强制执行超时 → 如果超过则中止运行
   - 返回负载 + 使用元数据
4) `subscribeEmbeddedPiSession` 将 pi-agent-core 事件桥接到 Moltbot `agent` 流：
   - 工具事件 => `stream: "tool"`
   - 助手增量 => `stream: "assistant"`
   - 生命周期事件 => `stream: "lifecycle"` (`phase: "start" | "end" | "error"`)
5) `agent.wait` 使用 `waitForAgentJob`：
   - 等待 **lifecycle end/error** 对于 `runId`
   - 返回 `{ status: ok|error|timeout, startedAt, endedAt, error? }`

## 队列 + 并发
- 运行按每个会话密钥（会话通道）和可选的全局通道进行序列化。
- 这可以防止工具/会话竞争并保持会话历史一致。
- 消息传递渠道可以选择队列模式（collect/steer/followup），以馈送到此通道系统。
  请参阅 [Command Queue](/concepts/queue)。

## 会话 + 工作区准备
- 工作区被解析并创建；沙盒运行可能会重定向到沙盒工作区根目录。
- Skills 被加载（或从快照重用）并注入到环境和提示中。
- 引导/上下文文件被解析并注入到系统提示报告中。
- 获取会话写锁；在流式传输之前打开并准备 `SessionManager`。

## 提示组装 + 系统提示
- 系统提示由 Moltbot 的基础提示、skills 提示、引导上下文和每次运行覆盖构建。
- 强制执行特定模型的限制和压缩保留令牌。
- 有关模型看到的内容，请参阅 [System prompt](/concepts/system-prompt)。

## 挂钩点（您可以拦截的地方）

Moltbot 有两个挂钩系统：
- **内部挂钩**（Gateway 挂钩）：用于命令和生命周期事件的事件驱动脚本。
- **插件挂钩**：agent/工具生命周期和 gateway 管道内的扩展点。

### 内部挂钩（Gateway 挂钩）
- **`agent:bootstrap`**：在系统提示最终化之前构建引导文件时运行。
  使用此功能添加/删除引导上下文文件。
- **命令挂钩**：`/new`、`/reset`、`/stop` 和其他命令事件（参见 Hooks 文档）。

请参阅 [Hooks](/hooks) 了解设置和示例。

### 插件挂钩（agent + gateway 生命周期）
这些在 agent 循环或 gateway 管道内运行：
- **`before_agent_start`**：在运行开始之前注入上下文或覆盖系统提示。
- **`agent_end`**：完成后检查最终消息列表和运行元数据。
- **`before_compaction` / `after_compaction`**：观察或注释压缩周期。
- **`before_tool_call` / `after_tool_call`**：拦截工具参数/结果。
- **`tool_result_persist`**：在将工具结果写入会话记录之前同步转换它们。
- **`message_received` / `message_sending` / `message_sent`**：入站 + 出站消息挂钩。
- **`session_start` / `session_end`**：会话生命周期边界。
- **`gateway_start` / `gateway_stop`**：gateway 生命周期事件。

请参阅 [Plugins](/plugin#plugin-hooks) 了解挂钩 API 和注册详细信息。

## 流式传输 + 部分回复
- 助手增量从 pi-agent-core 流式传输并作为 `assistant` 事件发出。
- 块流式传输可以在 `text_end` 或 `message_end` 上发出部分回复。
- 推理流式传输可以作为单独的流或块回复发出。
- 有关分块和块回复行为，请参阅 [Streaming](/concepts/streaming)。

## 工具执行 + 消息传递工具
- 工具启动/更新/结束事件在 `tool` 流上发出。
- 在记录/发出之前，工具结果会根据大小和图像负载进行清理。
- 消息传递工具发送被跟踪以抑制重复的助手确认。

## 回复塑造 + 抑制
- 最终负载由以下内容组装：
  - 助手文本（和可选的推理）
  - 内联工具摘要（当详细 + 允许时）
  - 模型错误时的助手错误文本
- `NO_REPLY` 被视为静默令牌并从出站负载中过滤。
- 消息传递工具重复项从最终负载列表中删除。
- 如果没有可渲染的负载并且工具出错，则发出回退工具错误回复
  （除非消息传递工具已经发送了用户可见的回复）。

## 压缩 + 重试
- 自动压缩发出 `compaction` 流事件并可以触发重试。
- 重试时，内存缓冲区和工具摘要被重置以避免重复输出。
- 有关压缩管道，请参阅 [Compaction](/concepts/compaction)。

## 事件流（当前）
- `lifecycle`：由 `subscribeEmbeddedPiSession` 发出（并且由 `agentCommand` 作为回退）
- `assistant`：来自 pi-agent-core 的流式增量
- `tool`：来自 pi-agent-core 的流式工具事件

## 聊天频道处理
- 助手增量被缓冲到聊天 `delta` 消息中。
- 在**生命周期结束/错误**上发出聊天 `final`。

## 超时
- `agent.wait` 默认：30s（仅等待）。`timeoutMs` 参数覆盖。
- Agent 运行时：`agents.defaults.timeoutSeconds` 默认 600s；在 `runEmbeddedPiAgent` 中中止计时器内强制执行。

## 提前结束的地方
- Agent 超时（中止）
- AbortSignal（取消）
- Gateway 断开连接或 RPC 超时
- `agent.wait` 超时（仅等待，不停止 agent）
