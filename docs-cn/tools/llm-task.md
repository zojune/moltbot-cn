---
summary: "用于工作流的仅限 JSON 的 LLM 任务（可选插件工具）"
read_when:
  - 您想要工作流内的仅限 JSON 的 LLM 步骤
  - 您需要架构验证的 LLM 输出用于自动化
---

# LLM 任务

`llm-task` 是一个**可选插件工具**，运行仅限 JSON 的 LLM 任务并返回结构化输出（可选择通过 JSON Schema 验证）。

这非常适用于工作流引擎，如 Lobster：您可以添加单个 LLM 步骤，而无需为每个工作流编写自定义 Moltbot 代码。

## 启用插件

1) 启用插件：

```json
{
  "plugins": {
    "entries": {
      "llm-task": { "enabled": true }
    }
  }
}
```

2) 允许列表工具（它注册为 `optional: true`）：

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "tools": { "allow": ["llm-task"] }
      }
    ]
  }
}
```

## 配置（可选）

```json
{
  "plugins": {
    "entries": {
      "llm-task": {
        "enabled": true,
        "config": {
          "defaultProvider": "openai-codex",
          "defaultModel": "gpt-5.2",
          "defaultAuthProfileId": "main",
          "allowedModels": ["openai-codex/gpt-5.2"],
          "maxTokens": 800,
          "timeoutMs": 30000
        }
      }
    }
  }
}
```

`allowedModels` 是 `provider/model` 字符串的允许列表。如果设置，列表之外的任何请求都会被拒绝。

## 工具参数

- `prompt`（字符串，必需）
- `input`（任何，可选）
- `schema`（对象，可选 JSON Schema）
- `provider`（字符串，可选）
- `model`（字符串，可选）
- `authProfileId`（字符串，可选）
- `temperature`（数字，可选）
- `maxTokens`（数字，可选）
- `timeoutMs`（数字，可选）

## 输出

返回包含解析的 JSON 的 `details.json`（并在提供时验证 `schema`）。

## 示例：Lobster 工作流步骤

```lobster
clawd.invoke --tool llm-task --action json --args-json '{
  "prompt": "给定输入电子邮件，返回意图和草稿。",
  "input": {
    "subject": "Hello",
    "body": "Can you help?"
  },
  "schema": {
    "type": "object",
    "properties": {
      "intent": { "type": "string" },
      "draft": { "type": "string" }
    },
    "required": ["intent", "draft"],
    "additionalProperties": false
  }
}'
```

## 安全说明

- 工具是**仅限 JSON 的**，并指示模型仅输出 JSON（无代码围栏、无评论）。
- 此运行不向模型公开工具。
- 除非使用 `schema` 验证，否则将输出视为不受信任。
- 在任何产生副作用的步骤（发送、发布、exec）之前放置批准。
