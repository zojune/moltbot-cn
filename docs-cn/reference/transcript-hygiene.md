---
summary: "参考：特定于提供商的转录清理和修复规则"
read_when:
  - 您正在调试与转录形状相关的提供商请求拒绝
  - 您正在更改转录清理或工具调用修复逻辑
  - 您正在调查跨提供商的工具调用 id 不匹配问题
---
# 转录清理（提供商修复）

本文档描述了在运行之前（构建模型上下文）应用于转录的**特定于提供商的修复**。这些是**内存中**的调整，用于满足严格的提供商要求。它们**不会**重写磁盘上存储的 JSONL 转录。

范围包括：
- 工具调用 id 清理
- 工具结果配对修复
- 轮次验证/排序
- 思考签名清理
- 图像载荷清理

如果您需要转录存储详细信息，请参阅：
- [/reference/session-management-compaction](/reference/session-management-compaction)

---

## 运行位置

所有转录清理都集中在嵌入式运行器中：
- 策略选择：`src/agents/transcript-policy.ts`
- 清理/修复应用：`src/agents/pi-embedded-runner/google.ts` 中的 `sanitizeSessionHistory`

该策略使用 `provider`、`modelApi` 和 `modelId` 来决定应用什么。

---

## 全局规则：图像清理

图像载荷总是会被清理，以防止因大小限制导致提供商端拒绝（缩小/重新压缩过大的 base64 图像）。

实现：
- `src/agents/pi-embedded-helpers/images.ts` 中的 `sanitizeSessionMessagesImages`
- `src/agents/tool-images.ts` 中的 `sanitizeContentBlocksImages`

---

## 提供商矩阵（当前行为）

**OpenAI / OpenAI Codex**
- 仅图像清理。
- 切换到 OpenAI Responses/Codex 模型时，删除孤立的推理签名（没有后续内容块的独立推理项）。
- 无工具调用 id 清理。
- 无工具结果配对修复。
- 无轮次验证或重新排序。
- 无合成工具结果。
- 无思考签名剥离。

**Google (Generative AI / Gemini CLI / Antigravity)**
- 工具调用 id 清理：严格字母数字。
- 工具结果配对修复和合成工具结果。
- 轮次验证（Gemini 风格的轮次交替）。
- Google 轮次排序修复（如果历史记录以助手开头，则在前面添加一个微小的用户引导块）。
- Antigravity Claude：标准化思考签名；删除无符号的思考块。

**Anthropic / Minimax（Anthropic 兼容）**
- 工具结果配对修复和合成工具结果。
- 轮次验证（合并连续的用户轮次以满足严格的交替要求）。

**Mistral（包括基于模型 ID 的检测）**
- 工具调用 id 清理：strict9（长度为 9 的字母数字）。

**OpenRouter Gemini**
- 思考签名清理：剥离非 base64 的 `thought_signature` 值（保留 base64）。

**其他所有提供商**
- 仅图像清理。

---

## 历史行为（2026.1.22 之前）

在 2026.1.22 版本之前，Moltbot 应用了多层转录清理：

- 在每次上下文构建时运行的**transcript-sanitize 扩展**，它可以：
  - 修复工具使用/结果配对。
  - 清理工具调用 id（包括保留 `_`/`-` 的非严格模式）。
- 运行器还执行特定于提供商的清理，这重复了工作。
- 在提供商策略之外发生了额外的变更，包括：
  - 在持久化之前从助手文本中剥离 `<final>` 标签。
  - 删除空的助手错误轮次。
  - 在工具调用后修剪助手内容。

这种复杂性导致了跨提供商回归（特别是 `openai-responses` 的 `call_id|fc_id` 配对）。2026.1.22 的清理删除了扩展，在运行器中集中了逻辑，并使 OpenAI 在图像清理之外变为**不触控**。
