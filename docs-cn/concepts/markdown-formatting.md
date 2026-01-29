---
summary: "出站通道的 Markdown 格式化管道"
read_when:
  - 你正在更改出站通道的 markdown 格式或分块
  - 你正在添加新的通道格式化程序或样式映射
  - 你正在调试跨通道的格式回归
---
# Markdown 格式化

Moltbot 通过将出站 Markdown 转换为共享的中间表示（IR）来格式式化，然后再渲染特定于通道的输出。IR 保持源文本完整，同时携带样式/链接跨度，以便分块和渲染在通道间保持一致。

## 目标

- **一致性**：一个解析步骤，多个渲染器。
- **安全分块**：在渲染之前分割文本，以便内联格式永远不会跨分块分裂。
- **通道适配**：将相同的 IR 映射到 Slack mrkdwn、Telegram HTML 和 Signal 样式范围，而无需重新解析 Markdown。

## 管道

1. **解析 Markdown -> IR**
   - IR 是纯文本加上样式跨度（粗体/斜体/删除线/代码/剧透）和链接跨度。
   - 偏移量是 UTF-16 代码单元，因此 Signal 样式范围与其 API 对齐。
   - 仅当通道选择加入表转换时才解析表。
2. **分块 IR（格式优先）**
   - 分块发生在渲染之前的 IR 文本上。
   - 内联格式不会跨分块分裂；跨度按分块切片。
3. **每个通道渲染**
   - **Slack**：mrkdwn 标记（粗体/斜体/删除线/代码），链接为 `<url|label>`。
   - **Telegram**：HTML 标签（`<b>`、`<i>`、`<s>`、`<code>`、`<pre><code>`、`<a href>`）。
   - **Signal**：纯文本 + `text-style` 范围；当标签不同时，链接变为 `label (url)`。

## IR 示例

输入 Markdown：

```markdown
Hello **world** — see [docs](https://docs.molt.bot).
```

IR（示意图）：

```json
{
  "text": "Hello world — see docs.",
  "styles": [
    { "start": 6, "end": 11, "style": "bold" }
  ],
  "links": [
    { "start": 19, "end": 23, "href": "https://docs.molt.bot" }
  ]
}
```

## 使用位置

- Slack、Telegram 和 Signal 出站适配器从 IR 渲染。
- 其他通道（WhatsApp、iMessage、MS Teams、Discord）仍然使用纯文本或其自己的格式规则，在启用时在分块之前应用 Markdown 表转换。

## 表处理

聊天客户端对 Markdown 表的支持不一致。使用 `markdown.tables` 控制每个通道（和每个账户）的转换。

- `code`：将表渲染为代码块（大多数通道的默认值）。
- `bullets`：将每一行转换为项目符号（Signal + WhatsApp 的默认值）。
- `off`：禁用表解析和转换；原始表文本通过。

配置键：

```yaml
channels:
  discord:
    markdown:
      tables: code
    accounts:
      work:
        markdown:
          tables: off
```

## 分块规则

- 分块限制来自通道适配器/配置，应用于 IR 文本。
- 代码围栏保留为单个块，带有尾随换行符，以便通道正确渲染它们。
- 列表前缀和块引用前缀是 IR 文本的一部分，因此分块不会在前缀中间分裂。
- 内联样式（粗体/斜体/删除线/内联代码/剧透）永远不会跨分块分裂；渲染器在每个分块中重新打开样式。

如果你需要更多关于跨通道分块行为的信息，请参见[流式传输 + 分块](/concepts/streaming)。

## 链接策略

- **Slack**：`[label](url)` -> `<url|label>`；裸 URL 保持裸露。解析期间禁用自动链接以避免双重链接。
- **Telegram**：`[label](url)` -> `<a href="url">label</a>`（HTML 解析模式）。
- **Signal**：`[label](url)` -> `label (url)`，除非标签与 URL 匹配。

## 剧透

剧透标记（`||spoiler||`）仅为 Signal 解析，其中它们映射到 SPOILER 样式范围。其他通道将它们视为纯文本。

## 如何添加或更新通道格式化程序

1. **解析一次**：使用共享的 `markdownToIR(...)` 帮助程序，带有适合通道的选项（自动链接、标题样式、块引用前缀）。
2. **渲染**：使用 `renderMarkdownWithMarkers(...)` 实现渲染器和样式标记映射（或 Signal 样式范围）。
3. **分块**：在渲染之前调用 `chunkMarkdownIR(...)`；渲染每个分块。
4. **连接适配器**：更新通道出站适配器以使用新的分块器和渲染器。
5. **测试**：添加或更新格式测试，如果通道使用分块，则添加出站交付测试。

## 常见陷阱

- Slack 尖括号标记（`<@U123>`、`<#C123>`、`<https://...>`）必须保留；安全地转义原始 HTML。
- Telegram HTML 需要转义标签外的文本以避免破坏的标记。
- Signal 样式范围取决于 UTF-16 偏移量；不要使用代码点偏移量。
- 保留围栏代码块的尾随换行符，以便关闭标记落在自己的行上。
