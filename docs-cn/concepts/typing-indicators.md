---
summary: "Moltbot 何时显示输入指示器以及如何调整它们"
read_when:
  - 更改输入指示器行为或默认值
---
# 输入指示器

输入指示器在运行期间发送到聊天频道。使用 `agents.defaults.typingMode` 控制**何时**开始输入，使用 `typingIntervalSeconds` 控制**多久**刷新一次。

## 默认值

当 `agents.defaults.typingMode` **未设置**时，Moltbot 保持旧行为：
- **直接聊天**：一旦模型循环开始立即开始输入。
- **带有提及的组聊天**：立即开始输入。
- **没有提及的组聊天**：仅在消息文本开始流式传输时开始输入。
- **心跳运行**：禁用输入。

## 模式

将 `agents.defaults.typingMode` 设置为以下之一：
- `never` — 永远不显示输入指示器。
- `instant` — 在模型循环开始时立即开始输入，即使运行后来只返回静默回复令牌。
- `thinking` — 在**第一个推理增量**上开始输入（需要运行的 `reasoningLevel: "stream"`）。
- `message` — 在**第一个非静默文本增量**上开始输入（忽略 `NO_REPLY` 静默令牌）。

"多久触发一次"的顺序：
`never` → `message` → `thinking` → `instant`

## 配置
```json5
{
  agent: {
    typingMode: "thinking",
    typingIntervalSeconds: 6
  }
}
```

您可以为每个会话覆盖模式或节奏：
```json5
{
  session: {
    typingMode: "message",
    typingIntervalSeconds: 4
  }
}
```

## 注意
- `message` 模式不会为仅静默的回复显示输入（例如用于抑制输出的 `NO_REPLY` 令牌）。
- `thinking` 仅在运行流式传输推理时触发（`reasoningLevel: "stream"`）。
  如果模型不发出推理增量，则不会开始输入。
- 无论模式如何，心跳从不显示输入。
- `typingIntervalSeconds` 控制**刷新节奏**，而不是开始时间。
  默认值为 6 秒。
