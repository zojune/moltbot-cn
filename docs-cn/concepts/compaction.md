---
summary: "上下文窗口 + 压缩：Moltbot 如何保持会话在模型限制内"
read_when:
  - 您想了解自动压缩和 /compact
  - 您正在调试触及上下文限制的长会话
---
# 上下文窗口和压缩

每个模型都有一个**上下文窗口**（它可以看到的最大令牌数）。长时间运行的聊天会累积消息和工具结果；一旦窗口紧张，Moltbot 会**压缩**旧历史记录以保持在限制内。

## 压缩是什么

压缩将**较旧的对话**总结为一个紧凑的摘要条目，并保持最近的消息完整。摘要存储在会话历史记录中，因此未来的请求使用：
- 压缩摘要
- 压缩点之后的最近消息

压缩**持久化**在会话的 JSONL 历史记录中。

## 配置

请参阅 [Compaction config & modes](/concepts/compaction) 了解 `agents.defaults.compaction` 设置。

## 自动压缩（默认开启）

当会话接近或超过模型的上下文窗口时，Moltbot 触发自动压缩，并可能使用压缩的上下文重试原始请求。

您将看到：
- 在详细模式下显示 `🧹 Auto-compaction complete`
- `/status` 显示 `🧹 Compactions: <count>`

在压缩之前，Moltbot 可以运行**静默记忆刷新**轮，以将持久的注释存储到磁盘。有关详细信息和配置，请参阅 [Memory](/concepts/memory)。

## 手动压缩

使用 `/compact`（可选带说明）强制压缩通过：
```
/compact Focus on decisions and open questions
```

## 上下文窗口来源

上下文窗口是特定于模型的。Moltbot 使用配置的提供程序目录中的模型定义来确定限制。

## 压缩 vs 修剪

- **压缩**：总结并**持久化**在 JSONL 中。
- **会话修剪**：仅修剪旧的**工具结果**，**在内存中**，每次请求。

有关修剪详细信息，请参阅 [/concepts/session-pruning](/concepts/session-pruning)。

## 提示

- 当会话感觉陈旧或上下文膨胀时使用 `/compact`。
- 大型工具输出已被截断；修剪可以进一步减少工具结果累积。
- 如果您需要全新的状态，`/new` 或 `/reset` 启动新的会话 id。
