---
summary: "计划：添加 OpenResponses /v1/responses 端点并优雅地弃用聊天补全"
owner: "moltbot"
status: "draft"
last_updated: "2026-01-19"
---

# OpenResponses 网关集成计划

## 背景

Moltbot 网关目前在 `/v1/chat/completions` 处暴露了一个最小的 OpenAI 兼容聊天补全端点
（参见 [OpenAI 聊天补全](/gateway/openai-http-api)）。

Open Responses 是一个基于 OpenAI Responses API 的开放推理标准。它是为代理工作流程设计的，
使用基于项目的输入和语义流式事件。OpenResponses 规范定义的是 `/v1/responses`，而不是
`/v1/chat/completions`。

## 目标

- 添加一个符合 OpenResponses 语义的 `/v1/responses` 端点。
- 将聊天补全作为兼容性层保留，使其易于禁用并最终移除。
- 使用隔离的、可重用的模式标准化验证和解析。

## 非目标

- 在第一阶段实现完整的 OpenResponses 功能对等（图片、文件、托管工具）。
- 替换内部代理执行逻辑或工具编排。
- 在第一阶段更改现有的 `/v1/chat/completions` 行为。

## 研究摘要

来源：OpenResponses OpenAPI、OpenResponses 规范网站和 Hugging Face 博客文章。

提取的关键点：

- `POST /v1/responses` 接受 `CreateResponseBody` 字段，如 `model`、`input`（字符串或
  `ItemParam[]`）、`instructions`、`tools`、`tool_choice`、`stream`、`max_output_tokens` 和
  `max_tool_calls`。
- `ItemParam` 是一个可区分联合，包括：
  - 角色为 `system`、`developer`、`user`、`assistant` 的 `message` 项
  - `function_call` 和 `function_call_output`
  - `reasoning`
  - `item_reference`
- 成功的响应返回一个 `ResponseResource`，其中包含 `object: "response"`、`status` 和
  `output` 项。
- 流式传输使用语义事件，例如：
  - `response.created`、`response.in_progress`、`response.completed`、`response.failed`
  - `response.output_item.added`、`response.output_item.done`
  - `response.content_part.added`、`response.content_part.done`
  - `response.output_text.delta`、`response.output_text.done`
- 规范要求：
  - `Content-Type: text/event-stream`
  - `event:` 必须与 JSON `type` 字段匹配
  - 终端事件必须是字面值 `[DONE]`
- 推理项可能暴露 `content`、`encrypted_content` 和 `summary`。
- HF 示例在请求中包含 `OpenResponses-Version: latest`（可选标头）。

## 提议的架构

- 添加 `src/gateway/open-responses.schema.ts`，仅包含 Zod 模式（无网关导入）。
- 添加 `src/gateway/openresponses-http.ts`（或 `open-responses-http.ts`）用于 `/v1/responses`。
- 保持 `src/gateway/openai-http.ts` 完整，作为传统兼容性适配器。
- 添加配置 `gateway.http.endpoints.responses.enabled`（默认为 `false`）。
- 保持 `gateway.http.endpoints.chatCompletions.enabled` 独立；允许两个端点分别切换。
- 当聊天补全已启用时，发出启动警告以指示传统状态。

## 聊天补全的弃用路径

- 维护严格的模块边界：响应和聊天补全之间没有共享的模式类型。
- 通过配置使聊天补全成为可选项，以便可以在没有代码更改的情况下禁用它。
- 更新文档，在 `/v1/responses` 稳定后将聊天补全标记为传统。
- 可选的未来步骤：将聊天补全请求映射到响应处理器，以实现更简单的移除路径。

## 第一阶段支持子集

- 接受 `input` 为字符串或 `ItemParam[]`，具有消息角色和 `function_call_output`。
- 将系统和开发者消息提取到 `extraSystemPrompt` 中。
- 使用最近的 `user` 或 `function_call_output` 作为代理运行的当前消息。
- 使用 `invalid_request_error` 拒绝不支持的内容部分（图片/文件）。
- 返回带有 `output_text` 内容的单个助手消息。
- 返回 `usage`，值设为零，直到令牌计费完成。

## 验证策略（无 SDK）

- 为支持的部分实现 Zod 模式：
  - `CreateResponseBody`
  - `ItemParam` + 消息内容部分联合
  - `ResponseResource`
  - 网关使用的流式事件形状
- 将模式保存在单个隔离模块中，以避免漂移并允许未来的代码生成。

## 流式实现（第一阶段）

- 带有 `event:` 和 `data:` 的 SSE 行。
- 必需序列（最小可行）：
  - `response.created`
  - `response.output_item.added`
  - `response.content_part.added`
  - `response.output_text.delta`（根据需要重复）
  - `response.output_text.done`
  - `response.content_part.done`
  - `response.completed`
  - `[DONE]`

## 测试和验证计划

- 为 `/v1/responses` 添加端到端覆盖：
  - 需要认证
  - 非流式响应形状
  - 流式事件排序和 `[DONE]`
  - 使用标头和 `user` 进行会话路由
- 保持 `src/gateway/openai-http.e2e.test.ts` 不变。
- 手动：使用 `stream: true` 对 `/v1/responses` 进行 curl，并验证事件排序和终端
  `[DONE]`。

## 文档更新（后续）

- 为 `/v1/responses` 用法和示例添加新的文档页面。
- 使用传统说明更新 `/gateway/openai-http-api` 并指向 `/v1/responses`。
