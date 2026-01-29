---
summary: "序列化入站自动回复运行的命令队列设计"
read_when:
  - 更改自动回复执行或并发
---
# 命令队列 (2026-01-16)

我们通过一个微小的进程内队列序列化入站自动回复运行（所有频道），以防止多个 agent 运行冲突，同时仍然允许跨会话的安全并行。

## 为什么
- 自动回复运行可能很昂贵（LLM 调用），并且当多个入站消息接近到达时可能会冲突。
- 序列化避免竞争共享资源（会话文件、日志、CLI stdin）并减少上游速率限制的机会。

## 工作原理
- 具有可配置并发上限的通道感知 FIFO 队列（未配置通道的默认为 1；main 默认为 4，subagent 为 8）。
- `runEmbeddedPiAgent` 通过**会话密钥**（通道 `session:<key>`）排队，以保证每个会话只有一个活动运行。
- 每个会话运行然后排队到**全局通道**（`main` 默认），因此总体并行性由 `agents.defaults.maxConcurrent` 限制。
- 当启用详细日志记录时，如果排队运行在开始前等待超过约 2 秒，则会发出简短通知。
- 输入指示器仍在排队时立即触发（当频道支持时），因此用户体验在我们等待时保持不变。

## 队列模式（每个频道）
入站消息可以引导当前运行、等待后续轮次或两者兼有：
- `steer`：立即注入当前运行（在下一个工具边界后取消挂起的工具调用）。如果不流式传输，则回退到后续。
- `followup`：排队等待当前运行结束后的下一个 agent 轮次。
- `collect`：将所有排队消息合并到**单个**后续轮次中（默认）。如果消息针对不同的频道/线程，它们将单独排出以保留路由。
- `steer-backlog`（aka `steer+backlog`）：现在引导**并**保留消息以进行后续轮次。
- `interrupt`（遗留）：中止该会话的活动运行，然后运行最新消息。
- `queue`（遗留别名）：与 `steer` 相同。

引导-后备意味着您可以在引导运行后获得后续响应，因此流式表面可能看起来像重复。如果您希望每条入站消息一个响应，请优先选择 `collect`/`steer`。
作为独立命令发送 `/queue collect`（每个会话）或设置 `messages.queue.byChannel.discord: "collect"`。

默认值（配置中未设置时）：
- 所有表面 → `collect`

通过 `messages.queue` 全局或每个频道配置：

```json5
{
  messages: {
    queue: {
      mode: "collect",
      debounceMs: 1000,
      cap: 20,
      drop: "summarize",
      byChannel: { discord: "collect" }
    }
  }
}
```

## 队列选项
选项适用于 `followup`、`collect` 和 `steer-backlog`（以及 `steer` 在它回退到后续时）：
- `debounceMs`：在开始后续轮次之前等待安静（防止"继续，继续"）。
- `cap`：每个会话的最大排队消息。
- `drop`：溢出策略（`old`、`new`、`summarize`）。

总结保留已删除消息的简短要点列表，并将其注入为合成的后续提示。
默认值：`debounceMs: 1000`、`cap: 20`、`drop: summarize`。

## 每个会话覆盖
- 发送 `/queue <mode>` 作为独立命令以存储当前会话的模式。
- 选项可以组合：`/queue collect debounce:2s cap:25 drop:summarize`
- `/queue default` 或 `/queue reset` 清除会话覆盖。

## 范围和保证
- 适用于使用 gateway 回复管道的所有入站频道的自动回复 agent 运行（WhatsApp web、Telegram、Slack、Discord、Signal、iMessage、webchat 等）。
- 默认通道（`main`）对于入站 + 主心跳是进程范围的；设置 `agents.defaults.maxConcurrent` 以允许多个会话并行。
- 可能存在其他通道（例如 `cron`、`subagent`），因此后台作业可以并行运行而不阻止入站回复。
- 每个会话通道保证一次只有一个 agent 运行触及给定的会话。
- 没有外部依赖或后台工作线程；纯 TypeScript + promises。

## 故障排除
- 如果命令看起来卡住了，请启用详细日志并查找"queued for …ms"行以确认队列正在排出。
- 如果需要队列深度，请启用详细日志并观察队列计时行。
