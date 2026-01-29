---
summary: "Harden cron.add 输入 handling, align schemas, and improve cron UI/代理 tooling"
owner: "moltbot"
status: "complete"
last_updated: "2026-01-05"
---

# Cron Add Hardening & 模式 Alignment

## 上下文
Recent gateway logs show repeated `cron.add` failures with invalid parameters (missing `sessionTarget`, `wakeMode`, `payload`, and malformed `schedule`). This indicates that at least one client (likely the agent tool call path) is sending wrapped or partially specified job payloads. Separately, there is drift between cron provider enums in TypeScript, gateway schema, CLI flags, and UI form types, plus a UI mismatch for `cron.status` (expects `jobCount` while gateway returns `jobs`).

## Goals
- Stop `cron.add` INVALID_REQUEST spam by normalizing common wrapper payloads and inferring missing `kind` fields.
- Align cron 提供商 lists across Gateway 模式, cron types, CLI docs, and UI forms.
- Make 代理 cron 工具 模式 explicit so the LLM produces correct job payloads.
- Fix the Control UI cron 状态 job count display.
- Add 测试 to cover normalization and 工具 行为.

## Non-goals
- Change cron scheduling semantics or job execution 行为.
- Add new schedule kinds or cron expression parsing.
- Overhaul the UI/UX for cron beyond the necessary 字段 fixes.

## Findings (current gaps)
- `CronPayloadSchema` in gateway excludes `signal` + `imessage`, while TS types include them.
- Control UI CronStatus expects `jobCount`, but gateway returns `jobs`.
- Agent cron tool schema allows arbitrary `job` objects, enabling malformed inputs.
- Gateway strictly validates `cron.add` with no normalization, so wrapped payloads fail.

## What changed

- `cron.add` and `cron.update` now normalize common wrapper shapes and infer missing `kind` fields.
- 代理 cron 工具 模式 matches the Gateway 模式, which reduces invalid payloads.
- 提供商 enums are aligned across Gateway, CLI, UI, and macOS picker.
- Control UI uses the gateway’s `jobs` count 字段 for 状态.

## Current 行为

- **Normalization:** wrapped `data`/`job` payloads are unwrapped; `schedule.kind` and `payload.kind` are inferred when safe.
- **Defaults:** safe defaults are applied for `wakeMode` and `sessionTarget` when missing.
- **提供商:** Discord/Slack/Signal/iMessage are now consistently surfaced across CLI/UI.

参见 [Cron jobs](/automation/cron-jobs) for the normalized shape and 示例.

## Verification

- Watch gateway logs for reduced `cron.add` INVALID_REQUEST 错误.
- Confirm Control UI cron 状态 shows job count after refresh.

## 可选 Follow-ups

- Manual Control UI smoke: add a cron job per 提供商 + verify 状态 job count.

## Open Questions
- Should `cron.add` accept explicit `state` from 客户端 (currently disallowed by 模式)?
- Should we allow `webchat` as an explicit delivery 提供商 (currently filtered in delivery resolution)?
