---
summary: "RPC 协议 notes for onboarding wizard and 配置 模式"
read_when: "Changing onboarding wizard steps or 配置 模式 端点"
---

# Onboarding + 配置 协议

Purpose: shared onboarding + 配置 surfaces across CLI, macOS 应用, and Web UI.

## Components
- Wizard engine (shared 会话 + prompts + onboarding 状态).
- CLI onboarding uses the same wizard flow as the UI 客户端.
- Gateway RPC exposes wizard + 配置 模式 端点.
- macOS onboarding uses the wizard step 模型.
- Web UI renders 配置 forms from JSON 模式 + UI hints.

## Gateway RPC
- `wizard.start` params: `{ mode?: "local"|"remote", workspace?: string }`
- `wizard.next` params: `{ sessionId, answer?: { stepId, value? } }`
- `wizard.cancel` params: `{ sessionId }`
- `wizard.status` params: `{ sessionId }`
- `config.schema` params: `{}`

响应 (shape)
- Wizard: `{ sessionId, done, step?, status?, error? }`
- Config schema: `{ schema, uiHints, version, generatedAt }`

## UI Hints
- `uiHints` keyed by 路径; 可选 元数据 (label/help/group/order/advanced/sensitive/placeholder).
- Sensitive fields render as password inputs; no redaction layer.
- Unsupported 模式 节点 fall back to the raw JSON editor.

## Notes
- This doc is the single place to track 协议 refactors for onboarding/配置.
