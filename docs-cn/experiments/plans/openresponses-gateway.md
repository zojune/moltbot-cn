---
summary: "Plan: Add OpenResponses /v1/响应 端点 and deprecate chat completions cleanly"
owner: "moltbot"
status: "draft"
last_updated: "2026-01-19"
---

# OpenResponses Gateway Integration Plan

## 上下文

Moltbot Gateway currently exposes a minimal OpenAI-compatible Chat Completions 端点 at
`/v1/chat/completions` (参见 [OpenAI Chat Completions](/Gateway/openai-http-API)).

Open 响应 is an open inference standard based on the OpenAI 响应 API. It is designed
for agentic workflows and uses item-based inputs plus semantic 流式 事件. The OpenResponses
spec defines `/v1/responses`, not `/v1/chat/completions`.

## Goals

- Add a `/v1/responses` 端点 that adheres to OpenResponses semantics.
- Keep Chat Completions as a compatibility layer that is easy to disable and eventually remove.
- Standardize 验证 and parsing with isolated, reusable schemas.

## Non-goals

- Full OpenResponses 功能 parity in the first pass (images, 文件, hosted 工具).
- Replacing internal 代理 execution logic or 工具 orchestration.
- Changing the existing `/v1/chat/completions` 行为 during the first phase.

## Research 摘要

Sources: OpenResponses OpenAPI, OpenResponses specification site, and the Hugging Face blog post.

键 points extracted:

- `POST /v1/responses` accepts `CreateResponseBody` fields like `model`, `input` (字符串 or
  `ItemParam[]`), `instructions`, `tools`, `tool_choice`, `stream`, `max_output_tokens`, and
  `max_tool_calls`.
- `ItemParam` is a discriminated union of:
  - `message` items with roles `system`, `developer`, `user`, `assistant`
  - `function_call` and `function_call_output`
  - `reasoning`
  - `item_reference`
- Successful responses return a `ResponseResource` with `object: "response"`, `status`, and
  `output` items.
- 流式 uses semantic 事件 such as:
  - `response.created`, `response.in_progress`, `response.completed`, `response.failed`
  - `response.output_item.added`, `response.output_item.done`
  - `response.content_part.added`, `response.content_part.done`
  - `response.output_text.delta`, `response.output_text.done`
- The spec requires:
  - `Content-Type: text/event-stream`
  - `event:` must match the JSON `type` 字段
  - terminal event must be literal `[DONE]`
- Reasoning items may expose `content`, `encrypted_content`, and `summary`.
- HF examples include `OpenResponses-Version: latest` in 请求 (可选 header).

## Proposed Architecture

- Add `src/gateway/open-responses.schema.ts` containing Zod schemas only (no Gateway imports).
- Add `src/gateway/openresponses-http.ts` (or `open-responses-http.ts`) for `/v1/responses`.
- Keep `src/gateway/openai-http.ts` intact as a legacy compatibility adapter.
- Add config `gateway.http.endpoints.responses.enabled` (default `false`).
- Keep `gateway.http.endpoints.chatCompletions.enabled` independent; allow both 端点 to be
  toggled separately.
- Emit a startup 警告 when Chat Completions is 已启用 to signal legacy 状态.

## Deprecation 路径 for Chat Completions

- Maintain strict 模块 boundaries: no shared 模式 types between 响应 and chat completions.
- Make Chat Completions opt-in by 配置 so it can be 已禁用 without code changes.
- Update docs to label Chat Completions as legacy once `/v1/responses` is stable.
- 可选 future step: map Chat Completions 请求 to the 响应 处理器 for a simpler
  removal 路径.

## Phase 1 Support Subset

- Accept `input` as string or `ItemParam[]` with message roles and `function_call_output`.
- Extract system and developer messages into `extraSystemPrompt`.
- Use the most recent `user` or `function_call_output` as the current 消息 for 代理 runs.
- Reject unsupported content parts (image/file) with `invalid_request_error`.
- Return a single assistant message with `output_text` content.
- Return `usage` with zeroed values until 令牌 accounting is wired.

## 验证 Strategy (No SDK)

- Implement Zod schemas for the supported subset of:
  - `CreateResponseBody`
  - `ItemParam` + 消息 content part unions
  - `ResponseResource`
  - 流式 事件 shapes used by the Gateway
- Keep schemas in a single, isolated 模块 to avoid drift and allow future codegen.

## 流式 实现 (Phase 1)

- SSE lines with both `event:` and `data:`.
- 必需 序列 (minimum viable):
  - `response.created`
  - `response.output_item.added`
  - `response.content_part.added`
  - `response.output_text.delta` (repeat as needed)
  - `response.output_text.done`
  - `response.content_part.done`
  - `response.completed`
  - `[DONE]`

## 测试 and Verification Plan

- Add e2e coverage for `/v1/responses`:
  - 认证 必需
  - Non-流 响应 shape
  - Stream event ordering and `[DONE]`
  - Session routing with headers and `user`
- Keep `src/gateway/openai-http.e2e.test.ts` unchanged.
- Manual: curl to `/v1/responses` with `stream: true` and verify 事件 ordering and terminal
  `[DONE]`.

## Doc 更新 (Follow-up)

- Add a new docs page for `/v1/responses` 用法 and 示例.
- Update `/gateway/openai-http-api` with a legacy note and pointer to `/v1/responses`.
